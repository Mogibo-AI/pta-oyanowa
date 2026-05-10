# コンポーネント依存関係 — 全自動PTA - おやのわ

**ドキュメント種別**: AI-DLC Application Design - Component Dependency 成果物
**作成日**: 2026-05-09

---

## 1. 依存関係マトリクス

凡例:
- 🔵 = 同期呼び出し（API/関数呼び出し）
- 🟢 = 非同期呼び出し（SQS/SNS/Streams 経由）
- 🟡 = ライブラリ依存（import）
- 🔴 = データアクセス（DynamoDB/S3 など）
- ⚪ = 直接依存なし

### Frontend → Backend
| from \ to | C-AUTHZ | C-BE-AUTH | C-BE-SUGGEST | C-BE-SURVEY | C-BE-NOTIF | C-BE-REPORT | C-BE-ADMIN | C-BE-EVENT |
|---|---|---|---|---|---|---|---|---|
| C-FE-01 ADMIN-WEB | 🔵 | 🔵 | 🔵 | 🔵 | 🔵 | 🔵 | 🔵 | 🔵 |
| C-FE-02 PARENT-APP | 🔵 | 🔵 | 🔵 | 🔵 | 🔵 | 🔵 | ⚪ | 🔵 |

> 注: C-FE-02 から C-BE-ADMIN への直接呼び出しはなし（管理者専用エンドポイント）

### Frontend → Shared Packages
| from \ to | C-PKG-TYPES | C-PKG-UI | C-PKG-API-CLIENT | C-PKG-AUTH | C-PKG-I18N |
|---|---|---|---|---|---|
| C-FE-01 ADMIN-WEB | 🟡 | 🟡 | 🟡 | 🟡 | 🟡 |
| C-FE-02 PARENT-APP | 🟡 | 🟡 | 🟡 | 🟡 | 🟡 |

### Backend Services → Data Layer
| from \ to | DynamoDB | S3 | DynamoDB Streams |
|---|---|---|---|
| C-BE-AUTH | 🔴 | ⚪ | ⚪ |
| C-BE-SUGGEST-BOX | 🔴 | ⚪ | （投稿で発火）|
| C-BE-SURVEY | 🔴 | ⚪ | ⚪ |
| C-BE-NOTIFICATION | 🔴 | ⚪ | ⚪ |
| C-BE-REPORT | 🔴 | 🔴（PDF/将来） | ⚪ |
| C-BE-ADMIN | 🔴 | ⚪ | ⚪ |
| C-BE-EVENT | 🔴 | ⚪ | （実施確定で発火）|

### Backend Services → AI Layer
| from \ to | SQS | C-AI-ORCH | C-AI-SUB-* | C-AI-LIB |
|---|---|---|---|---|
| C-BE-SUGGEST-BOX | （Streams 経由）| ⚪ | ⚪ | ⚪ |
| C-BE-SURVEY | ⚪ | ⚪ | ⚪ | ⚪ |
| その他 Backend | ⚪ | ⚪ | ⚪ | ⚪ |

> Backend Service から AI Sub-Agent への**直接呼び出しは行わない**。
> AI 起動はすべて非同期（SQS / Step Functions / EventBridge / Streams）経由で疎結合を保つ。

### AI Layer 内部依存
| from \ to | C-AI-LIB | Bedrock | DynamoDB |
|---|---|---|---|
| C-AI-ORCH（Step Functions）| ⚪ | ⚪ | 🔴（実行履歴）|
| C-AI-SUB-CLASSIFY | 🟡 | 🔵 | 🔴 |
| C-AI-SUB-DECISION | 🟡 | 🔵 | 🔴 |
| C-AI-SUB-GENERATE | 🟡 | 🔵 | 🔴 |
| C-AI-SUB-ANALYZE | 🟡 | 🔵 | 🔴 |
| C-AI-SUB-EVENT | 🟡 | 🔵 | 🔴 |
| C-AI-SUB-REPORT | 🟡 | 🔵 | 🔴 |
| C-AI-MONTHLY | ⚪ | ⚪ | 🔴 |

### Stream Processors → Backend / AI
| from \ to | SQS | C-BE-NOTIFICATION |
|---|---|---|
| C-STREAM-POST | 🟢（Push）| ⚪ |
| C-STREAM-EVENT-CONFIRM | ⚪ | 🔵 |

### Backend Services → C-AUTHZ / Shared Packages
| from \ to | C-AUTHZ | C-PKG-TYPES | C-AI-LIB |
|---|---|---|---|
| 全 Backend Service Lambda | （API Gateway 経由で前段）| 🟡 | 🟡（必要時のみ）|

> C-AUTHZ は API Gateway の Lambda Authorizer として動作し、ビジネスロジック Lambda の前段で実行される。

---

## 2. 通信パターン

### パターン A: 同期 HTTPS（Frontend → API）
```
[Frontend (ADMIN-WEB / PARENT-APP)]
  ↓ HTTPS + Cognito JWT (Authorization: Bearer)
[CloudFront + WAF]  ← SECURITY-04, SECURITY-07
  ↓
[API Gateway REST]
  ↓ Authorizer 起動
[Lambda Authorizer (C-AUTHZ)]
  ├─ JWT検証 / schoolId抽出 / IDOR防止 / 学校アクティブ確認
  └─ Allow + Context (schoolId, userId, userType)
  ↓ Allow
[Backend Service Lambda (Express)]
  ├─ Zod バリデーション (SECURITY-05)
  ├─ ビジネスロジック実行
  └─ DynamoDB 操作 (schoolId フィルタ強制)
  ↓
[JSON レスポンス]
```

### パターン B: イベント駆動（DynamoDB Streams → SQS → Lambda）
```
[Backend Service]
  ↓ DynamoDB INSERT
[DynamoDB Streams]
  ↓ INSERT イベント
[Stream Processor Lambda (C-STREAM-POST)]
  ↓ SQS SendMessage
[SQS Queue]
  ↓ 非同期ポーリング（バッチサイズ 1〜10）
[AI Sub-Agent Lambda (C-AI-SUB-CLASSIFY)]
  ↓ Bedrock 呼び出し
[Amazon Bedrock]
  ↓ 結果
[DynamoDB UPDATE]
```

### パターン C: スケジュール駆動 + Step Functions（毎日9時）
```
[EventBridge: rate(1 day) at 9:00 JST]
  ↓
[Step Functions: DailyScheduler State Machine]
  ↓ 各学校に対して並列実行
[State 1: Survey Decision Lambda]
  ↓ 結果に応じて分岐
[State 2: Survey Generation Lambda] (条件付き)
  ↓
[State 2-Notify: Notification Lambda]
  ↓ SNS Publish → FCM
[State 3: Scan Expiring Surveys Lambda]
  ↓
[State 4 Map: Survey Result Analysis Lambda]
  ↓ 結果に応じて分岐
[State 4-EventPlan: Event Planning Lambda] (条件付き)
  ↓
[完了]
```

### パターン D: スケジュール駆動シンプル（月初レポート）
```
[EventBridge: cron(0 15 1 * ? *) UTC = 月初 0:00 JST]
  ↓
[Lambda: Monthly Report Trigger]
  ↓ 各学校反復
[Lambda: Monthly Report Sub-Agent]
  ↓ Bedrock Sonnet 呼び出し
[DynamoDB: レポート保存]
```

### パターン E: ストリーム駆動 → 通知（イベント実施確定）
```
[管理者: confirmEvent API 呼び出し]
  ↓
[C-BE-EVENT: confirmEvent]
  ↓ DynamoDB UPDATE (status: CANDIDATE → CONFIRMED)
[DynamoDB Streams]
  ↓ UPDATE イベント
[C-STREAM-EVENT-CONFIRM]
  ↓ 同期 invoke
[C-BE-NOTIFICATION.sendEventConfirmedNotification]
  ↓ SNS Publish
[全保護者プッシュ通知 (B-8 C)]
```

---

## 3. データフロー図（主要シナリオ）

### シナリオ 1: 投稿から AI 分類まで

```
保護者 → [POST /posts]
  → APIGateway
  → Authorizer
  → C-BE-SUGGEST-BOX.createPost
  → DynamoDB.PutItem (status=PENDING, category=null)
  → 201 返却（保護者は待たない）
  
DynamoDB Streams (非同期)
  → C-STREAM-POST
  → SQS.SendMessage
  
SQS Queue (ポーリング)
  → C-AI-SUB-CLASSIFY
  → Bedrock.InvokeModel (Haiku, 適用 Guardrails)
  → DynamoDB.UpdateItem (category=SAFETY, summary=...)
  
保護者 → [GET /posts] (リフレッシュ後)
  → 分類済みデータ取得
```

### シナリオ 2: アンケート配信〜結果分析〜イベント企画

```
EventBridge (毎日 9:00 JST)
  → Step Functions DailyScheduler 起動

Step Functions State Machine
  └─ Map (各学校)
      ├─ State 1: Survey Decision
      │    Lambda → Bedrock Sonnet
      │    → { shouldDeliver: true }
      │
      ├─ State 2: Survey Generation
      │    Lambda → Bedrock Sonnet → DynamoDB.PutItem (Survey)
      │    
      ├─ State 2-Notify: Notification
      │    C-BE-NOTIFICATION → SNS Publish → FCM → 保護者
      │
      ├─ State 3: Scan Expiring Surveys
      │    DynamoDB.Query (期日到来 + 未分析)
      │
      └─ State 4 Map (各期日到来サーベイ)
          ├─ Survey Result Analysis
          │    Lambda → Bedrock Sonnet
          │    → { nextAction: EVENT_PLAN }
          │
          └─ Event Planning
               Lambda → Bedrock Sonnet
               → DynamoDB.PutItem (EventProposal, status=VOTING)

保護者 (任意のタイミング)
  → アプリでアンケート回答
  → C-BE-SURVEY.submitAnswers → DynamoDB.UpdateItem (集計加算)
  
保護者 (任意のタイミング)
  → アプリで企画案閲覧
  → C-BE-EVENT.listEventProposals → DynamoDB.Query
  → 賛否投票
  → C-BE-EVENT.voteEventProposal → DynamoDB.UpdateItem
  → 投票率20%以上 & 賛成多数 → status: CANDIDATE
  
管理者
  → 管理画面で実施確定
  → C-BE-EVENT.confirmEvent → DynamoDB.UpdateItem (status=CONFIRMED)

DynamoDB Streams (非同期)
  → C-STREAM-EVENT-CONFIRM
  → C-BE-NOTIFICATION.sendEventConfirmedNotification
  → SNS → FCM → 全保護者通知

イベント当日
  → 保護者がアプリで「チェックイン」
  → C-BE-EVENT.checkinEvent (QR or Tap, B-7 X)
  → DynamoDB.PutItem (Checkin)

管理者 (リアルタイム)
  → 管理画面で C-BE-EVENT.listEventCheckins
  → 5秒間隔ポーリングでリアルタイム表示
```

---

## 4. 結合度・凝集度の分析

### 高凝集（同じ機能エリア内のコンポーネント）
| 機能エリア | 高凝集なコンポーネント群 |
|---|---|
| 認証 | C-BE-AUTH + C-AUTHZ + C-PKG-AUTH |
| 目安箱 | C-BE-SUGGEST-BOX + C-STREAM-POST + C-AI-SUB-CLASSIFY |
| アンケート | C-BE-SURVEY + C-AI-SUB-DECISION + C-AI-SUB-GENERATE + C-AI-SUB-ANALYZE |
| イベント | C-BE-EVENT + C-AI-SUB-EVENT + C-STREAM-EVENT-CONFIRM |
| レポート | C-BE-REPORT + C-AI-MONTHLY + C-AI-SUB-REPORT |
| AI共通 | C-AI-LIB（全 AI Sub-Agent から共通利用）|

### 疎結合（異なる機能エリア間）
- Backend Service Lambda 同士は **直接呼び出さない**（DynamoDB / Streams / SQS 経由で連携）
- AI Sub-Agent は Backend Service を呼び出さない（DynamoDB 直接更新で結果反映）
- フロントエンドは API Gateway を経由して各 Backend Service にアクセス

### 共通依存（横断ライブラリ）
| ライブラリ | 利用元 |
|---|---|
| C-PKG-TYPES | フロントエンド + Backend Service + AI Sub-Agent + Stream Processor（**ほぼ全コンポーネント**）|
| C-AI-LIB | 全 AI Sub-Agent + Monthly Report Trigger |
| C-PKG-API-CLIENT | フロントエンドのみ（C-FE-01 / C-FE-02）|
| C-PKG-AUTH | フロントエンドのみ |
| C-PKG-I18N | フロントエンドのみ |
| C-PKG-UI | フロントエンドのみ |

---

## 5. 依存サイクル防止

### 確認すべきルール
- [x] Backend Service Lambda 同士の直接呼び出しなし → イベント駆動で疎結合
- [x] AI Sub-Agent から Backend Service 呼び出しなし → DynamoDB 直接更新
- [x] Shared Package (C-PKG-*) → Backend / Frontend へ依存しない（一方向）
- [x] C-PKG-TYPES → 他のどのコンポーネントにも依存しない（最も基底）
- [x] C-AI-LIB → C-PKG-TYPES のみに依存

### 依存階層（下位ほど依存される側、上位ほど依存する側）

```
Level 5: Frontend (ADMIN-WEB / PARENT-APP)
            ↓ depends on
Level 4: API Gateway + Stream Processors + Step Functions
            ↓ depends on
Level 3: Backend Service Lambdas + AI Sub-Agents + Authorizer
            ↓ depends on
Level 2: Shared Packages (UI, API-CLIENT, AUTH, I18N) + AI Library
            ↓ depends on
Level 1: TYPES (Zod schemas, Domain types)
            ↓ depends on
Level 0: AWS Services (Cognito, DynamoDB, S3, SNS, EventBridge, Bedrock)
```

---

## 6. 共有データソースの利用パターン

### DynamoDB シングルテーブルへのアクセスパターン

各機能エリアは **同じテーブル** を異なる PK/SK パターンで利用:

| 機能 | PK パターン | SK パターン | 主な GSI |
|---|---|---|---|
| 学校情報 | `SCHOOL#{schoolId}` | `META` | – |
| 学校コード | `SCHOOL#{schoolId}` | `CODE#{codeId}` | GSI1: `CODE#{code}` |
| ユーザー | `SCHOOL#{schoolId}` | `USER#{userId}` | GSI1: `USER#{userId}` |
| 投稿 | `SCHOOL#{schoolId}` | `POST#{ulid}` | GSI1: `SCHOOL#{schoolId}#POSTS` (sort by createdAt) / GSI2: by category |
| 支持 | `SCHOOL#{schoolId}` | `POST#{postId}#SUPPORT#{userId}` | – |
| アンケート | `SCHOOL#{schoolId}` | `SURVEY#{surveyId}` | GSI3: by status |
| 回答 | `SCHOOL#{schoolId}` | `SURVEY#{surveyId}#ANSWER#{userId}` | – |
| 集計 | `SCHOOL#{schoolId}` | `SURVEY#{surveyId}#AGG` | – |
| 月次レポート | `SCHOOL#{schoolId}` | `REPORT#{yyyy-mm}` | – |
| イベント企画案 | `SCHOOL#{schoolId}` | `EVENT#{eventId}` | GSI3: by status |
| 投票 | `SCHOOL#{schoolId}` | `EVENT#{eventId}#VOTE#{userId}` | – |
| チェックイン | `SCHOOL#{schoolId}` | `EVENT#{eventId}#CHECKIN#{userId}` | – |
| 操作ログ | `SCHOOL#{schoolId}` | `OPLOG#{ulid}` | GSI4: timestamp ソート |
| デバイストークン | `SCHOOL#{schoolId}` | `DEVICE#{userId}` | – |

### マルチテナント分離の保証
- 全クエリは PK に `SCHOOL#{schoolId}` を含める（IDOR 防止）
- C-AUTHZ で抽出した schoolId と PK の schoolId 一致を Express middleware で強制
- E2E テストで他校データへのアクセス試行を必ず実施（403 確認）

---

## 7. 障害伝播と緩和策

| シナリオ | 影響範囲 | 緩和策 |
|---|---|---|
| Bedrock 障害 | AI Sub-Agent 全滅 → 投稿は分類されず、Daily Scheduler スキップ | SQS DLQ、SECURITY-15 fail-degraded（投稿は OTHER 扱い）、Slack 通知 |
| DynamoDB スロットリング | 全 Backend / AI Service が遅延・失敗 | オンデマンド設定で自動スケール、再試行は SDK レベル |
| Lambda コールドスタート | 認可・各 API レスポンスで遅延 | C-AUTHZ は5分キャッシュ、頻繁なエンドポイントは provisioned concurrency 検討（コスト要相談）|
| Step Functions 失敗 | Daily Scheduler 中断 → 当日の AI 配信無し | 各ステップに Retry、Catch で Slack 通知、翌日の Schedule で復旧 |
| SNS / FCM 障害 | プッシュ通知届かず | アプリ内通知で補完、配信履歴 DynamoDB 記録で重複防止しつつリトライ |
| Cognito 障害 | 新規ログイン不可、既存セッション継続 | フロントは既存トークンで継続、エラーメッセージで案内 |
| EventBridge 失敗 | Daily Scheduler 起動失敗 | 翌日の起動で実質的にカバー、CloudWatch アラーム設定 |

---

## 8. CDK スタック間の依存

```
NetworkStack (空、将来用)
   ↑
SecretsStack
   ↑
DataStack ← AuthStack ← MonitoringStack
   ↑           ↑              ↑
   └─── ApiStack
              ↑
   └─── AiStack (Step Functions, SQS, Bedrock Guardrails)
              ↑
   └─── NotificationStack (EventBridge rules, SNS)
              ↑
   └─── FrontendStack (CloudFront, S3 admin web)
              ↑
   └─── MobileBuildStack (EAS Build IAM, OIDC)
```

CDK のスタック間で SSM Parameter Store / CloudFormation Outputs を経由して値を共有する。
