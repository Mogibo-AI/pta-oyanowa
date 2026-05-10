# コンポーネント定義 — 全自動PTA - おやのわ

**ドキュメント種別**: AI-DLC Application Design - Components 成果物
**作成日**: 2026-05-09
**設計方針**: Bounded Context ベース（A-1: A）+ 機能エリア単位 Lambda（A-2: B）

---

## 1. コンポーネント一覧概観

| カテゴリ | コンポーネント数 | 配置 |
|---|---|---|
| Frontend | 2 | apps/ |
| Backend Service Lambdas | 7 | backend/functions/ |
| AI Components | 8（オーケストレーター1 + サブエージェント6 + 共通AIライブラリ1） | backend/functions/ai/ + backend/lib/ai/ |
| Authorizer | 1 | backend/functions/auth-authorizer/ |
| Stream Processor | 2 | backend/functions/streams/ |
| Shared Packages | 5 | packages/ |
| Infrastructure | 1（複数 CDK スタックを内包）| infra/cdk/ |
| **合計** | **26 コンポーネント** | |

---

## 2. Frontend コンポーネント

### C-FE-01: ADMIN-WEB（管理者向け Web アプリ）
| 項目 | 値 |
|---|---|
| 配置 | `apps/web-admin/` |
| 技術 | Next.js 16 (App Router) + React 19.2 + shadcn/ui + Zustand + TanStack Query |
| 関連機能要件 | F-06（基本管理者画面）、F-09（Coming Soon UI 管理者向け）、F-08（企画案確認・実施確定）、F-10（参加者数把握）、F-04（レポート確認） |
| 関連ストーリー | US-F06-001〜004 / US-F02-004 / US-F03-002 / US-F04-002 / US-F08-003 / US-F10-002 |
| 主要画面 | ログイン（MFA含む）/ ダッシュボード / 学校コード管理 / 保護者一覧 / 投稿一覧 / アンケート状況 / 企画案レビュー / イベント管理 / 月次レポート確認 / 即時通知配信 / 公開操作ログ |
| 認証 | Cognito Admin User Pool + TOTP 強制 |
| デプロイ | CloudFront + S3 経由（または Vercel/Amplify は禁止のため CloudFront 配信）|

### C-FE-02: PARENT-APP（保護者向けモバイルアプリ）
| 項目 | 値 |
|---|---|
| 配置 | `apps/app-parent/` |
| 技術 | React Native 0.85 + Expo + EAS Build + React Navigation + Zustand + TanStack Query |
| ターゲット | **Android 6.0+ (API level 23+) 限定（MVP）**、iOS は Phase 3 |
| 関連機能要件 | F-01〜F-05, F-07, F-08（投票）, F-09, F-10（チェックイン）, F-11 |
| 関連ストーリー | US-F01-001〜005 / US-F02-001〜003 / US-F03-001 / US-F04-001 / US-F05-001〜003 / US-F07-001 / US-F08-001/002 / US-F09-001 / US-F10-001 / US-F11-001 |
| 主要画面 | オンボーディング（QR読込・登録）/ ログイン / ホーム（Coming Soon含む）/ 目安箱（投稿・一覧・支持）/ アンケート（一覧・回答・結果）/ 月次レポート / イベント企画案（閲覧・投票）/ チェックイン / プロフィール（退会含む）|
| 認証 | Cognito Parent User Pool + メール認証 + 学校コード |
| 配信 | EAS Build → .apk → Google Play |

---

## 3. Backend Service Lambda コンポーネント（機能エリア単位）

各 Service は Express on Lambda で複数 API ルートを処理する。

### C-BE-AUTH（認証サービス）
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/auth/` |
| 関連要件 | F-05.1〜5（学校コード招待・登録・ログイン・退会・MFA） |
| 関連ストーリー | US-F05-001〜004 |
| API 担当範囲 | `POST /auth/signup` / `POST /auth/verify-school-code` / `POST /auth/refresh-token` / `DELETE /auth/me`（退会）|
| 主要責務 | Cognito SignUp/SignIn 連携、学校コード検証、退会処理（物理削除 + 投稿匿名化）|

### C-BE-SUGGEST-BOX（目安箱サービス）
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/suggest-box/` |
| 関連要件 | F-01.1〜6（投稿・一覧・カテゴリ・支持・対応状況） |
| 関連ストーリー | US-F01-001〜005 |
| API 担当範囲 | `POST /posts` / `GET /posts` / `GET /posts/:id` / `POST /posts/:id/support` / `PATCH /posts/:id/status`（管理者のみ）|
| 主要責務 | 投稿 CRUD、支持カウント、ステータス管理、匿名化処理 |
| AI連携 | 投稿時 → DynamoDB Streams → SQS → Post Classification Sub-Agent |

### C-BE-SURVEY（アンケートサービス）
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/survey/` |
| 関連要件 | F-02.1〜9（配信判断・生成・回答・集計・結果分析・連鎖起動） |
| 関連ストーリー | US-F02-001〜005 |
| API 担当範囲 | `GET /surveys` / `GET /surveys/:id` / `POST /surveys/:id/answers` / `GET /surveys/:id/results` / `GET /surveys/:id/analysis`（管理者のみ）|
| 主要責務 | アンケート CRUD、回答受付、リアルタイム集計（DynamoDB UpdateItem）、分析結果取得 |
| AI連携 | 配信判断・生成・結果分析は Step Functions（Daily Scheduler）から起動 |

### C-BE-NOTIFICATION（通知サービス）
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/notification/` |
| 関連要件 | F-03.1〜3（プッシュ通知・即時配信） |
| 関連ストーリー | US-F03-001/002 / OQ-16 |
| API 担当範囲 | `POST /notifications/broadcast`（管理者の即時配信）/ `POST /notifications/event-confirmed`（イベント実施確定時の自動配信）|
| 主要責務 | SNS Publish 実行、デバイストークン管理、配信履歴記録 |
| 内部連携 | EventBridge / Step Functions から呼び出される |

### C-BE-REPORT（月次レポートサービス）
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/report/` |
| 関連要件 | F-04.1〜3（生成・閲覧・管理者確認） |
| 関連ストーリー | US-F04-001/002 |
| API 担当範囲 | `GET /reports` / `GET /reports/:id` |
| 主要責務 | DynamoDB から保存済みレポート取得 |
| AI連携 | レポート生成は EventBridge（月初）→ Monthly Report Sub-Agent |

### C-BE-ADMIN（管理者操作サービス）
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/admin/` |
| 関連要件 | F-06.1〜5（学校コード発行・保護者管理・破壊的操作 + 公開操作ログ） |
| 関連ストーリー | US-F06-001〜004 |
| API 担当範囲 | `POST /admin/school-codes` / `PATCH /admin/school-codes/:id` / `GET /admin/parents` / `DELETE /admin/parents/:id` / `GET /admin/post-trends` / `GET /public-operation-log` |
| 主要責務 | 管理者専用 CRUD、公開操作ログ生成（破壊的操作時）|

### C-BE-EVENT（イベントサービス: F-08 + F-10）
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/event/` |
| 関連要件 | F-08.1〜7（企画案・投票・実施候補マーク）、F-10.1〜7（実施確定・チェックイン）|
| 関連ストーリー | US-F08-001〜003 / US-F10-001/002 |
| API 担当範囲 | `GET /events/proposals` / `GET /events/proposals/:id` / `POST /events/proposals/:id/vote` / `POST /events/:id/confirm`（管理者）/ `POST /events/:id/check-in` / `GET /events/:id/check-ins`（管理者）|
| 主要責務 | 企画案 CRUD、投票管理、実施候補判定（賛成≥反対+興味なし & 投票率≥20%）、チェックイン処理（QRトークン or タップ）、参加者集計 |
| AI連携 | 企画案生成は Step Functions チェーン（Daily Scheduler の③）から起動 |

---

## 4. AI コンポーネント

### C-AI-ORCH: Daily Scheduler Orchestrator（Step Functions）
| 項目 | 値 |
|---|---|
| 配置 | `infra/cdk/stepfunctions/daily-scheduler.ts` |
| 関連要件 | AI-ARCH-01〜04 |
| 関連ストーリー | US-F02-001/004 / US-F08-001 |
| トリガー | EventBridge: 毎日9時 JST（cron(0 0 * * ? *) UTC）|
| 構成 | Step Functions State Machine: ① Survey Decision → ② Survey Generation（条件付き）→ ③ Survey Result Analysis → ④ Event Planning（条件付き）|
| 状態管理 | 各ステップの実行結果を Step Functions Execution History で追跡、X-Ray でトレース |

### C-AI-MONTHLY: Monthly Report Trigger
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/ai/monthly-report-trigger/` |
| 関連要件 | F-04, AI-ARCH-04 |
| トリガー | EventBridge: 月初 0時 JST（cron(0 15 1 * ? *) UTC = 月初 0時 JST）|
| 構成 | Lambda → Monthly Report Sub-Agent 直接起動 |

### C-AI-SUB-CLASSIFY: Post Classification Sub-Agent
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/ai/sub-classify/` |
| モデル | **Bedrock Claude Haiku 4.5**（高頻度低コスト処理）|
| 入力 | 投稿テキスト |
| 出力 | カテゴリ（SAFETY/EVENT/CONTACT/REQUEST/OTHER）+ 要約 |
| トリガー | DynamoDB Streams（投稿挿入）→ SQS → 本 Lambda |
| プロンプト | `backend/lib/ai/prompts/post-classification.ts` で管理（AI-PROMPT-01）|

### C-AI-SUB-DECISION: Survey Decision Sub-Agent
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/ai/sub-decision/` |
| モデル | **Bedrock Claude Sonnet 4.6** |
| 入力 | 当該学校の直近30日の投稿一覧 + 過去配信履歴 + 学校コンテキスト |
| 出力 | `{ shouldDeliver: bool, reason: string, mode: "ai" \| "threshold" }` |
| 動作モード | デフォルト = AI判断、管理者設定で「しきい値モード（投稿数+経過日数）」に切替可能（B-2 X 回答）|
| トリガー | Step Functions ステップ①|

### C-AI-SUB-GENERATE: Survey Generation Sub-Agent
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/ai/sub-generate/` |
| モデル | **Bedrock Claude Sonnet 4.6** |
| 入力 | Survey Decision の出力 + 学校カレンダー（MVPでは静的）+ 季節情報 + 過去意見傾向 |
| 出力 | アンケート設問 5〜7問（JSON）|
| トリガー | Step Functions ステップ②（条件: Decision.shouldDeliver = true）|

### C-AI-SUB-ANALYZE: Survey Result Analysis Sub-Agent
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/ai/sub-analyze/` |
| モデル | **Bedrock Claude Sonnet 4.6** |
| 入力 | 期日到来したアンケートの集計結果 + 該当期間の目安箱投稿 |
| 出力 | `{ nextAction: "EVENT_PLAN" \| "URGENT_TOPIC" \| "NO_ACTION", reason: string, urgencyScore: number }` |
| 動作モード | ルールベース（EVENT_PLAN = 要望集中・季節性 / URGENT_TOPIC = 緊急性高い意見一定数以上）+ AI判断による緊急性ブースト（1件でも明確な緊急性がある場合は URGENT_TOPIC として上書き、B-6 X 回答）|
| トリガー | Step Functions ステップ③|

### C-AI-SUB-EVENT: Event Planning Sub-Agent
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/ai/sub-event/` |
| モデル | **Bedrock Claude Sonnet 4.6** |
| 入力 | 直近の目安箱投稿 + アンケート分析結果 + 学校情報 |
| 出力 | 企画案 `{ title, description, expectedAttendees, suggestedTimeframe }` |
| トリガー | Step Functions ステップ④（条件: Analysis.nextAction = EVENT_PLAN）|

### C-AI-SUB-REPORT: Monthly Report Sub-Agent
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/ai/sub-report/` |
| モデル | **Bedrock Claude Sonnet 4.6** |
| 入力 | 前月の活動データ（投稿数・アンケート結果・実施イベント・予算データ）|
| 出力 | レポート Markdown + メタデータ |
| トリガー | C-AI-MONTHLY 経由 |

### C-AI-LIB: Bedrock Common Library
| 項目 | 値 |
|---|---|
| 配置 | `backend/lib/ai/` |
| 役割 | Bedrock SDK ラッパー、Guardrails 適用、リトライ処理、共通プロンプト管理ディレクトリ、ゴールデンテストインフラ |
| サブモジュール | `prompts/`（プロンプトファイル群）、`golden-tests/`（CIで実行）、`guardrails-config.ts` |

---

## 5. Authorizer コンポーネント

### C-AUTHZ: Lambda Authorizer (REQUEST 型)
| 項目 | 値 |
|---|---|
| 配置 | `backend/functions/auth-authorizer/` |
| 関連要件 | F-05.3 / SECURITY-08（IDOR防止）/ SECURITY-12（トークン検証）|
| 主要責務 | (1) Cognito JWT 署名・有効期限検証 / (2) `schoolId` クレーム抽出 / (3) リクエストパス内の schoolId と一致確認（IDOR防止）/ (4) 学校アクティブ確認（DynamoDB クエリ）/ (5) 結果を5分キャッシュ |
| 出力 | IAM Policy + コンテキスト（`schoolId`, `userId`, `userType`）|

---

## 6. Stream Processor コンポーネント

### C-STREAM-POST: Post Classification Trigger
| 配置 | `backend/functions/streams/post-classification-trigger/` |
| トリガー | DynamoDB Streams（投稿テーブル INSERT イベント）|
| 責務 | SQS にメッセージ送信（C-AI-SUB-CLASSIFY を間接起動、A-4 非同期方式）|

### C-STREAM-EVENT-CONFIRM: Event Confirmation Notifier
| 配置 | `backend/functions/streams/event-confirm-notifier/` |
| トリガー | DynamoDB Streams（イベントテーブル UPDATE: status=CONFIRMED）|
| 責務 | C-BE-NOTIFICATION 経由で全保護者にプッシュ通知（B-8 C: 全員に通知）|

---

## 7. Shared Packages

### C-PKG-TYPES: Shared Types
| 配置 | `packages/shared-types/` |
| 内容 | TypeScript 型定義 + Zod スキーマ（API リクエスト/レスポンス、DynamoDB エンティティ、AI入出力）|
| SECURITY | SECURITY-05（入力バリデーション基盤）|
| PBT | PBT-02（Zod パース/シリアライズのラウンドトリップテスト対象）|

### C-PKG-UI: Shared UI Library
| 配置 | `packages/shared-ui/` |
| 内容 | shadcn/ui ベースの共通 UI コンポーネント（Button, Card, Modal, Toast, Form など）|
| 注意 | RN と Web の両対応のため、コア UI はプラットフォームごとに分けるか、Tamagui のようなクロスプラットフォーム UI を検討 |

### C-PKG-API-CLIENT: API Client
| 配置 | `packages/shared-api-client/` |
| 内容 | superagent ベースの HTTP クライアント、TanStack Query 用 hooks、Cognito JWT 自動付与、リフレッシュトークン処理 |

### C-PKG-AUTH: Auth Helpers
| 配置 | `packages/shared-auth/` |
| 内容 | Cognito SDK ラッパー（SignIn / SignUp / SignOut / RefreshToken / MFA）、トークン永続化（Web=localStorage / RN=SecureStore）|

### C-PKG-I18N: i18n Configuration
| 配置 | `packages/shared-i18n/` |
| 内容 | i18next 設定、`locales/ja/*.json`、`locales/en/*.json`、言語切替ロジック、Coming Soon バッジの多言語化 |
| 関連 | F-07, F-09 |

---

## 8. Infrastructure Component

### C-INFRA: AWS CDK スタック群
| 配置 | `infra/cdk/` |
| 技術 | AWS CDK v2 (TypeScript) |
| 構成 | 以下のスタックに分離 |

| Stack | 内容 |
|---|---|
| `NetworkStack` | （MVP では VPC 不要、将来用プレースホルダ）|
| `AuthStack` | Cognito User Pool（保護者）+ Cognito User Pool（管理者）+ ID Pool |
| `DataStack` | DynamoDB シングルテーブル + GSI + Streams 設定 + S3 バケット（学校別プレフィックス）|
| `ApiStack` | API Gateway REST + Lambda Authorizer + 各サービス Lambda + WAF |
| `AiStack` | SQS Queue + DLQ + AI サブエージェント Lambda 群 + Step Functions State Machine + Bedrock Guardrails 定義 |
| `NotificationStack` | SNS（プッシュ通知用）+ SES（将来のメール通知用）+ EventBridge スケジュールルール |
| `MonitoringStack` | CloudWatch Dashboard + Alarms + X-Ray + Slack Webhook（OQ-02解決後）|
| `SecretsStack` | KMS Key + Secrets Manager（DB接続情報・APIキーは原則使わない、Bedrock は IAM Role認証）|
| `FrontendStack` | CloudFront + S3（admin web 配信）+ Route 53（将来）|
| `MobileBuildStack` | EAS Build 連携用 IAM ロール（OIDC） + S3 アーティファクト保存 |

---

## 9. SECURITY 拡張要件のコンポーネントマッピング

| SECURITY ルール | 主担当コンポーネント |
|---|---|
| SECURITY-01 暗号化 | C-INFRA（DataStack: KMS / S3 SSE）|
| SECURITY-02 ネットワーク中継ログ | C-INFRA（ApiStack: API Gatewayアクセスログ / FrontendStack: CloudFrontログ）|
| SECURITY-03 アプリ層ログ | C-AI-LIB + 全 Backend Lambda（structured JSON、schoolId必須、PII禁止）|
| SECURITY-04 HTTP セキュリティヘッダー | C-FE-01 ADMIN-WEB（Next.js middleware）+ C-INFRA（CloudFront 応答ヘッダーポリシー）|
| SECURITY-05 入力バリデーション | C-PKG-TYPES（Zod）+ 全 Backend Lambda |
| SECURITY-06 最小権限 | C-INFRA 全 Stack（IAM Role 個別定義）|
| SECURITY-07 ネットワーク制限 | C-INFRA（ApiStack: WAF / SQS/SNS の VPC エンドポイント不要MVP）|
| SECURITY-08 アプリ層認可 | C-AUTHZ + 全 Backend Lambda の handler ガード |
| SECURITY-09 設定ハードニング | C-INFRA（プロダクションエラー応答 / 公開バケット禁止 / 最新ランタイム）|
| SECURITY-10 サプライチェーン | CI/CD（GitHub Actions: pnpm-lock 必須・npm audit）+ C-INFRA（CodeBuild: 固定 Docker image）|
| SECURITY-11 セキュア設計 | C-PKG-AUTH に認証ロジック集約 + C-INFRA（API Gateway Throttling）|
| SECURITY-12 認証管理 | C-PKG-AUTH（Cognito MFA / パスワードポリシー）+ C-AUTHZ |
| SECURITY-13 整合性 | C-PKG-TYPES（安全な Zod パース）+ C-FE-01（CDN リソース SRI）|
| SECURITY-14 アラート | C-INFRA（MonitoringStack: アラート定義）|
| SECURITY-15 例外処理 | 全 Lambda の handler ラッパー（C-AI-LIB に共通エラーハンドラ）|

---

## 10. PBT 適用箇所マッピング

| PBT ルール | 適用コンポーネント |
|---|---|
| PBT-02 ラウンドトリップ | C-PKG-TYPES（Zod parse/stringify）+ C-BE-AUTH（Cognito クレームの round-trip）+ C-BE-EVENT（チェックイン QRトークンの encode/decode）|
| PBT-03 不変条件 | C-AI-SUB-CLASSIFY（出力カテゴリは必ず5種のいずれか）+ C-BE-SUGGEST-BOX（支持数の単調増加）+ C-BE-EVENT（投票集計の整合性）|
| PBT-07 ジェネレータ品質 | C-PKG-TYPES（ドメイン型ジェネレータ: Post, Survey, Event, User, SchoolCode）|
| PBT-08 シュリンク・再現性 | CI ジョブ全体の構成 |
| PBT-09 フレームワーク選定 | **fast-check** を C-PKG-TYPES と各 Backend Lambda の test/ で使用 |
