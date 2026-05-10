# アプリケーション設計書（統合版） — 全自動PTA - おやのわ

**ドキュメント種別**: AI-DLC Application Design - 統合成果物
**作成日**: 2026-05-09
**プロジェクト**: 全自動PTA - おやのわ MVP（Greenfield SaaS）

このドキュメントは、以下の個別設計成果物を統合したエントリポイントです。詳細は各ドキュメントを参照してください。

- 📄 [components.md](./components.md) — 26コンポーネントの定義
- 📄 [component-methods.md](./component-methods.md) — 各コンポーネントのメソッドシグネチャ
- 📄 [services.md](./services.md) — オーケストレーション・サービス層設計
- 📄 [component-dependency.md](./component-dependency.md) — 依存関係マトリクス・データフロー

---

## 1. 設計判断サマリー（質問回答ベース）

### コア設計方針

| 質問 | 採用 | 影響範囲 |
|---|---|---|
| **A-1**: コンポーネント分割 | **A: Bounded Context ベース** | 機能エリア F-01〜F-11 ごとに独立 Lambda + AI Sub-Agent |
| **A-2**: Lambda 粒度 | **B: 機能エリア単位（Express on Lambda）** | 7つの Backend Service Lambda + 6つの AI Sub-Agent Lambda |
| **A-3**: AI オーケストレーター | **A: AWS Step Functions** | DailyScheduler State Machine、月次レポートはシンプル Lambda 連鎖 |
| **A-4**: AI 同期/非同期 | **A: 非同期（SQS / Streams 経由）** | ユーザーは待たない、Bedrock 過負荷防止 |
| **A-5**: DynamoDB | **A: シングルテーブルデザイン** | PK = `SCHOOL#{schoolId}` で全機能のマルチテナント分離 |
| **A-6**: API 認可 | **A: REQUEST 型 Lambda Authorizer** | カスタム検証で IDOR 防止 + 学校アクティブ確認 + 5分キャッシュ |
| **A-7**: Frontend ルーティング | **A: Next.js App Router + RN Navigation + TanStack Query** | 各プラットフォームのベストプラクティス |
| **A-8**: i18n | **A: JSON + i18next** | Web/RN 共通パターン、`packages/shared-i18n` |
| **A-9**: Coming Soon 管理 | **A: TypeScript const ハードコード** | `packages/shared` 配下の設定ファイル |

### オープン課題の確定

| OQ | 確定内容 |
|---|---|
| **OQ-05** Bedrock Guardrails | 拒否トピック（政治・宗教・暴力・性的）+ PIIフィルタ（メール・電話・住所）+ 標準PI検出 |
| **OQ-07** 学校コード仕様 | 8文字英数字（0/O/I/l 除外）、有効期限デフォルト90日 |
| **OQ-09** 配信判断しきい値 | デフォルト = AI判断（Sonnet）、管理者設定でしきい値モード（投稿数+経過日数）に切替可能 |
| **OQ-10** 退会時削除 | 物理削除 + 投稿の userId のみ "RETIRED" に置換 |
| **OQ-11** Coming Soon 5項目 | 📅カレンダー / 💰会費・予算 / ✅タスク管理 / 📝議事録 / 📊過去情報ダッシュボード |
| **OQ-12** 賛成多数判定 | 賛成 ≥ (反対 + 興味なし) かつ 全保護者の20%以上が投票済み |
| **OQ-13** 次アクション判定基準 | ルールベース（要望集中/緊急性）+ AIによる緊急性ブースト（1件でも明確な緊急性で URGENT_TOPIC 昇格）|
| **OQ-15** チェックイン方式 | QR/タップ両方サポート、管理者がイベント単位で選択、デフォルト QR |
| **OQ-16** 実施確定後通知 | 全保護者にプッシュ通知（賛成・反対・興味なし問わず）|
| **OQ-17** Expo 採用 | Expo + EAS Build を全面採用 |

---

## 2. 全体アーキテクチャ概観

### 物理アーキテクチャ図

```
┌─────────────────────────────────────────────────────────────────┐
│                         ユーザー層                                 │
├──────────────────────────────┬──────────────────────────────────┤
│  保護者 (Android端末)          │  学校管理者 (PC ブラウザ)          │
│  C-FE-02 PARENT-APP           │  C-FE-01 ADMIN-WEB               │
│  React Native + Expo          │  Next.js 16 + shadcn/ui          │
│  (.apk via Google Play)       │  (CloudFront + S3 配信)           │
└──────────────┬───────────────┴──────────────┬───────────────────┘
               │                              │
               │ HTTPS + Cognito JWT          │
               ▼                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  AWS WAF + CloudFront (CDN/WAF)                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│            API Gateway REST + Lambda Authorizer                  │
│  C-AUTHZ: JWT検証 / schoolId抽出 / IDOR防止 / 学校アクティブ確認  │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Allow + Context
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                Backend Service Lambda 層 (Express)               │
├─────┬─────────┬────────┬──────────┬──────────┬──────┬─────────┤
│AUTH │SUGGEST  │SURVEY  │NOTIFICA  │REPORT    │ADMIN │EVENT    │
│F-05 │F-01     │F-02    │F-03      │F-04      │F-06  │F-08+F-10│
└──┬──┴────┬────┴────┬───┴───┬──────┴───┬──────┴──┬───┴────┬────┘
   │       │         │       │          │         │        │
   └───────┴─────────┴───────┴──────────┴─────────┴────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                                  │
│  DynamoDB SingleTable + Streams + S3 + Cognito User Pools        │
└──────────────┬──────────────────────────────────────────────────┘
               │
               │ Streams (非同期)
               ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Stream Processors                            │
│  C-STREAM-POST (→ SQS) / C-STREAM-EVENT-CONFIRM (→ Notification) │
└──────────────┬──────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────┐
│                       AI Layer                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Step Functions: DailyScheduler                              │ │
│  │  ① Decision → ② Generation → ③ Analysis → ④ EventPlan    │ │
│  │       (Sonnet)    (Sonnet)     (Sonnet)      (Sonnet)      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  Sub-Classify (Haiku, SQS 駆動)                                  │
│  Monthly Report (Sonnet, EventBridge 駆動)                        │
│                                                                   │
│  ┌─ Amazon Bedrock + Guardrails ────────────────────────────────┐│
│  │  Claude Sonnet 4.6 (高精度)                                  ││
│  │  Claude Haiku 4.5 (高頻度低コスト)                           ││
│  └──────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘

トリガー基盤:
  EventBridge (毎日9時 / 月初0時) → Step Functions / Lambda
  DynamoDB Streams → Stream Processors → SQS / 通知
  
通知:
  SNS → FCM → Android デバイス（プッシュ通知）

監視:
  CloudWatch Dashboard / Logs / X-Ray / Slack Webhook
```

### モノレポ構造

```
oyano-wa/
├── apps/
│   ├── web-admin/                  # C-FE-01 (Next.js 16)
│   └── app-parent/                 # C-FE-02 (React Native + Expo)
├── backend/
│   ├── functions/
│   │   ├── auth/                   # C-BE-AUTH
│   │   ├── suggest-box/            # C-BE-SUGGEST-BOX
│   │   ├── survey/                 # C-BE-SURVEY
│   │   ├── notification/           # C-BE-NOTIFICATION
│   │   ├── report/                 # C-BE-REPORT
│   │   ├── admin/                  # C-BE-ADMIN
│   │   ├── event/                  # C-BE-EVENT
│   │   ├── auth-authorizer/        # C-AUTHZ
│   │   ├── ai/
│   │   │   ├── sub-classify/       # C-AI-SUB-CLASSIFY (Haiku)
│   │   │   ├── sub-decision/       # C-AI-SUB-DECISION (Sonnet)
│   │   │   ├── sub-generate/       # C-AI-SUB-GENERATE (Sonnet)
│   │   │   ├── sub-analyze/        # C-AI-SUB-ANALYZE (Sonnet)
│   │   │   ├── sub-event/          # C-AI-SUB-EVENT (Sonnet)
│   │   │   ├── sub-report/         # C-AI-SUB-REPORT (Sonnet)
│   │   │   └── monthly-report-trigger/  # C-AI-MONTHLY
│   │   └── streams/
│   │       ├── post-classification-trigger/  # C-STREAM-POST
│   │       └── event-confirm-notifier/       # C-STREAM-EVENT-CONFIRM
│   ├── lib/
│   │   ├── ai/                     # C-AI-LIB (Bedrock ラッパー、プロンプト管理)
│   │   ├── auth/                   # 認証共通ロジック
│   │   ├── db/                     # DynamoDB ラッパー
│   │   └── utils/                  # 共通ユーティリティ
│   └── layers/                     # Lambda Layer (共通依存)
├── packages/
│   ├── shared-types/               # C-PKG-TYPES (Zod schemas, 型定義)
│   ├── shared-ui/                  # C-PKG-UI (shadcn ベースの共通 UI)
│   ├── shared-api-client/          # C-PKG-API-CLIENT (TanStack Query hooks)
│   ├── shared-auth/                # C-PKG-AUTH (Cognito ラッパー)
│   └── shared-i18n/                # C-PKG-I18N (i18next 設定)
├── infra/
│   └── cdk/                        # C-INFRA (CDK Stacks)
│       ├── stacks/
│       │   ├── network-stack.ts
│       │   ├── auth-stack.ts
│       │   ├── data-stack.ts
│       │   ├── api-stack.ts
│       │   ├── ai-stack.ts
│       │   ├── notification-stack.ts
│       │   ├── monitoring-stack.ts
│       │   ├── secrets-stack.ts
│       │   ├── frontend-stack.ts
│       │   └── mobile-build-stack.ts
│       └── stepfunctions/
│           └── daily-scheduler.ts
└── docs/
    └── api/                        # OpenAPI 仕様
```

---

## 3. ユニット候補（Units Generation 段階で確定予定）

| Unit ID | 範囲 | 関連コンポーネント | 推定ストーリー数 |
|---|---|---|---|
| **U-FE-ADMIN** | 管理者向け Web | C-FE-01 | 6（F-06系列）|
| **U-FE-PARENT** | 保護者向け Android アプリ | C-FE-02 | 16（F-01〜F-05, F-07, F-08, F-09, F-10, F-11）|
| **U-BE-AUTH** | 認証バックエンド | C-BE-AUTH + C-AUTHZ | 4（F-05系列）|
| **U-BE-SUGGEST-BOX** | 目安箱バックエンド | C-BE-SUGGEST-BOX + C-STREAM-POST | 5（F-01系列）|
| **U-BE-SURVEY** | アンケートバックエンド | C-BE-SURVEY | 5（F-02系列）|
| **U-BE-NOTIFICATION** | 通知バックエンド | C-BE-NOTIFICATION | 2（F-03系列）|
| **U-BE-REPORT** | レポートバックエンド | C-BE-REPORT | 2（F-04系列）|
| **U-BE-ADMIN** | 管理者操作バックエンド | C-BE-ADMIN | 4（F-06系列）|
| **U-BE-EVENT** | イベントバックエンド | C-BE-EVENT + C-STREAM-EVENT-CONFIRM | 5（F-08, F-10系列）|
| **U-AI-CORE** | AI共通基盤 | C-AI-LIB + Bedrock Guardrails | 0（共通）|
| **U-AI-AGENTS** | AI サブエージェント群 | C-AI-SUB-* 6体 | 0（共通基盤）|
| **U-AI-ORCHESTRATION** | Step Functions + EventBridge | C-AI-ORCH + C-AI-MONTHLY | 0（基盤）|
| **U-SHARED** | 共通パッケージ | C-PKG-* 5つ | 1（F-07）|
| **U-INFRA** | CDK スタック群 | C-INFRA | 0（基盤、F-11含む）|

→ **想定14ユニット** （Workflow Planning と一致）

---

## 4. 受け入れ基準カバレッジ確認

requirements.md §13 の受け入れ基準と本設計の対応:

| 受け入れ基準 | 設計上の保証 |
|---|---|
| 機能要件 F-01〜F-11 すべてが dev 環境で動作 | C-BE-* + C-FE-* + C-AI-* + C-INFRA で完全カバー |
| 全 SECURITY 15 ルールの適合 | components.md §9 SECURITY マッピングで全ルール紐付け済み |
| PBT 適用ルール（02/03/07/08/09）満足 | components.md §10 PBT マッピングで紐付け |
| AIエージェント分離（オーケストレーター + 6サブエージェント）| C-AI-ORCH + C-AI-SUB-* 6体 + C-AI-MONTHLY |
| 毎日9時 EventBridge → 配信判断 → アンケート配信 → 結果分析 → イベント企画 のチェーン | services.md §2 DailyScheduler State Machine |
| イベント企画案 → 賛否投票 → 実施候補 → 実施確定 → チェックイン → 参加状況確認 | services.md §6 EventCheckin Service + §5 EventConfirmation Service |
| プロンプトのゴールデンテストが CI で実行 | C-AI-LIB の `golden-tests/` + AI-PROMPT-02 |
| CloudWatch Dashboard で主要メトリクス可視化 | C-INFRA: MonitoringStack |
| Slack アラート発火確認 | C-INFRA: MonitoringStack + Slack Webhook |
| 日本語・英語UI主要画面表示 | C-PKG-I18N + 各 Frontend コンポーネント |
| CI/CD パイプライン動作 | GitHub Actions + CodeBuild + CodePipeline |
| 参加者チェックイン (F-10) 動作 | C-BE-EVENT + 管理者画面リアルタイム表示 |
| Android .apk 自動ビルド (F-11) | C-INFRA: MobileBuildStack + EAS Build |
| iOS ビルド未実施だが Phase 3 で活性化可能 | C-FE-02 + Expo 採用で iOS バイナリ生成可能な状態維持 |

---

## 5. 設計の主要トレードオフと選択理由

### Trade-off 1: Step Functions vs カスタム Lambda Orchestrator
**選択**: Step Functions（A-3 A）

**理由**:
- DailyScheduler は条件分岐（shouldDeliver / nextAction）と並列実行（Map）を含む
- Step Functions の Execution History でトレーサビリティ確保、デバッグが容易
- AWS X-Ray と統合される（NFR-MON-03）
- リトライ・エラーハンドリングが宣言的に書ける
- コスト: Step Functions Standard で1日1回の起動なら無視できる程度

**代替案を退けた理由**:
- カスタム Lambda: フローが複雑になると保守困難、エラーハンドリングが冗長
- Strands Agents: AWS純正だが、要件のフローはシンプルなチェーンで Step Functions で十分
- LangGraph: TypeScript 縛り（言語制約）に反する

### Trade-off 2: 機能エリア単位 Lambda vs エンドポイント単位 Lambda
**選択**: 機能エリア単位（A-2 B）

**理由**:
- Lambda 数を抑えられ、Cold Start 影響を最小化
- Express on Lambda で REST ルーティングが自然
- 同じビジネスロジック（DynamoDB アクセスパターン）を共有しやすい
- デプロイ単位が分かりやすい

**代替案を退けた理由**:
- エンドポイント単位: 細粒度すぎてコールドスタート増加、Lambda 数管理が煩雑
- モノリシック: スケーリング・障害分離・独立デプロイの利点を失う

### Trade-off 3: 非同期 AI 処理（SQS 経由） vs 同期処理
**選択**: 非同期（A-4 A）

**理由**:
- ユーザーレスポンスを待たせない（投稿即返却、後で分類結果反映）
- Bedrock の同時呼び出しレート制限を SQS で吸収
- 障害時に DLQ 経由でリトライ可能（SECURITY-15 fail-degraded）

**トレードオフ**:
- カテゴリラベルが投稿直後は表示されない（数秒〜数十秒の遅延）
- → UX 上、「分類中」プレースホルダで対応、定期リフレッシュで解決

### Trade-off 4: シングルテーブル vs マルチテーブル
**選択**: シングルテーブル（A-5 A）

**理由**:
- マルチテナント分離（schoolId プレフィックス）と整合
- 1校あたりのデータ量が DynamoDB の単一パーティション上限内に収まる想定（1000世帯規模）
- TransactWriteItems で複数エンティティ間の整合性を保てる
- コスト効率（スキャン回数削減）

**トレードオフ**:
- アクセスパターンの事前設計が必須（GSI 設計の硬直性）
- → component-dependency.md §6 で網羅的に整理

### Trade-off 5: REQUEST 型 Lambda Authorizer vs Cognito Authorizer
**選択**: REQUEST 型 Lambda Authorizer（A-6 A）

**理由**:
- マルチテナント要件が強い（schoolId 一致確認、IDOR 防止）
- 学校アクティブ確認などカスタムロジックを 1 箇所に集約
- 5分キャッシュでコスト・レイテンシの影響を最小化
- 将来のレート制限・追加検証拡張が容易

---

## 6. 残されたオープン課題（後続段階で解決）

| OQ ID | 課題 | 解決ステージ |
|---|---|---|
| **OQ-01** | 本番環境 staging/prod 構築タイミング | Phase 2 着手時 |
| **OQ-02** | Slack チャンネル名・Webhook URL 確保 | Code Generation 前 |
| **OQ-03** | 解約期間・データエクスポート | プライバシーポリシー策定（並行） |
| **OQ-04** | サポート体制 | Phase 1 リリース前 |
| **OQ-06** | プロンプトのゴールデンテストコーパス初期作成 | Code Generation |
| **OQ-08** | i18n 英語訳のレビュー体制 | Code Generation |
| **OQ-14** | F-08 検証期間・成功指標 | MVP リリース後 |
| **OQ-18** | Google Play 開発者アカウント取得・運用責任者 | MVP リリース前 |
| **OQ-19** | アプリ内決済 IAP 将来必要性 | Phase 2 着手時 |

→ **Application Design 段階で 9 件確定（OQ-05/07/09/10/11/12/13/15/16/17）**、残り 9 件は別の段階。

---

## 7. 次ステージ: Units Generation

本設計で確定した「14ユニット候補」を **Units Generation** ステージで正式に確定します。

Units Generation で確定する内容:
- 各ユニットの正式な ID と境界
- ユニット間の依存関係マップ
- ユニット単位のストーリーマップ（USx-Fxx-001 → Unit ID）
- per-unit Construction フェーズの実行順序
- 並列開発可能性の判定
