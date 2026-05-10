# ユニット定義 — 全自動PTA - おやのわ

**ドキュメント種別**: AI-DLC Units Generation - Unit Definition 成果物
**作成日**: 2026-05-09
**確定ユニット数**: **15ユニット**（追加要望6 で U-LOCAL を新設）

---

## 1. ユニット一覧（確定）

| Unit ID | 種別 | 主要機能要件 | コンポーネント | デプロイ | 担当ストーリー数 |
|---|---|---|---|---|---|
| **U-FE-ADMIN** | Frontend | F-06, F-09, F-04, F-08(管理), F-10(管理) | C-FE-01 | CDK FrontendStack | 6（主）+ 副複数 |
| **U-FE-PARENT** | Frontend (Mobile) | F-01〜F-05, F-07, F-08(投票), F-09(主), F-10(チェックイン), F-11 | C-FE-02 | EAS Build → Google Play | 4（主）+ 副多数 |
| **U-BE-AUTH** | Backend | F-05, SECURITY-08, SECURITY-12 | C-BE-AUTH + C-AUTHZ | CDK ApiStack/AuthStack | 5（主）|
| **U-BE-SUGGEST-BOX** | Backend | F-01 | C-BE-SUGGEST-BOX + C-STREAM-POST | CDK ApiStack | 5（主）|
| **U-BE-SURVEY** | Backend | F-02 | C-BE-SURVEY | CDK ApiStack | 3（主）|
| **U-BE-NOTIFICATION** | Backend | F-03 | C-BE-NOTIFICATION | CDK ApiStack/NotificationStack | 2（主）|
| **U-BE-REPORT** | Backend | F-04 | C-BE-REPORT | CDK ApiStack | 2（主）|
| **U-BE-ADMIN** | Backend | F-06 | C-BE-ADMIN | CDK ApiStack | 4（主）|
| **U-BE-EVENT** | Backend | F-08, F-10 | C-BE-EVENT + C-STREAM-EVENT-CONFIRM | CDK ApiStack | 5（主）|
| **U-AI-CORE** | AI Library | AI-PROMPT, OQ-05 | C-AI-LIB | Lambda Layer (CDK AiStack) | 2（主）|
| **U-AI-AGENTS** | AI Service | AI-ARCH | C-AI-SUB-CLASSIFY/DECISION/GENERATE/ANALYZE/EVENT/REPORT + C-AI-MONTHLY | CDK AiStack（6 Lambda + 1 Lambda） | 1（主）|
| **U-AI-ORCHESTRATION** | AI Workflow | AI-ARCH-01〜02 | C-AI-ORCH（Step Functions）+ EventBridge | CDK AiStack/NotificationStack | 0（基盤）|
| **U-SHARED** | Shared Packages | F-07, SECURITY-05, PBT-02, PBT-07 | C-PKG-TYPES/UI/API-CLIENT/AUTH/I18N | npm workspace（デプロイなし） | 1（主）|
| **U-INFRA** | Infrastructure (AWS) | F-11, NFR-MON, SECURITY-01〜10, 14 | C-INFRA（CDK 10スタック） | CDK Bootstrap → Deploy | 0（基盤）|
| **U-LOCAL** | Local Development（追加要望6）| ローカル動作確認全般 | C-INFRA-LOCAL + C-LOCAL-SERVER | デプロイなし（開発者マシン）| 0（基盤）|

---

## 2. 各ユニットの詳細定義

### U-FE-ADMIN — 管理者向け Web アプリ

| 項目 | 値 |
|---|---|
| **目的** | 学校管理者（校長・教頭）が PTA 運営を行う Web 管理画面 |
| **デプロイ単位** | CloudFront + S3（CDK FrontendStack 経由） |
| **コード配置** | `apps/web-admin/` |
| **技術** | Next.js 16 (App Router) / React 19.2 / shadcn/ui / Zustand / TanStack Query |
| **主要画面** | ログイン (MFA) / ダッシュボード / 学校コード管理 / 保護者一覧 / 投稿一覧 / アンケート状況 / 企画案レビュー / イベント管理 / 即時通知配信 / 公開操作ログ / 月次レポート確認 |
| **モジュール内分離** | `app/login/`, `app/dashboard/`, `app/school-codes/`, `app/parents/`, `app/posts/`, `app/surveys/`, `app/events/`, `app/notifications/`, `app/operation-log/`, `app/reports/` |
| **AI生成適合度** | ◎（shadcn/ui の定型実装、TypeScript 厳格型）|

### U-FE-PARENT — 保護者向け Android アプリ

| 項目 | 値 |
|---|---|
| **目的** | 保護者がスマホから PTA 活動に参加する Android アプリ |
| **デプロイ単位** | EAS Build → .apk → Google Play |
| **コード配置** | `apps/app-parent/` |
| **技術** | React Native 0.85 + Expo + EAS Build / React Navigation / Zustand / TanStack Query |
| **ターゲット** | Android 6.0+ (API level 23+) MVP のみ。iOS は Phase 3 |
| **主要画面** | オンボーディング (QR読込・登録) / ログイン / ホーム (Coming Soon含む) / 目安箱 / アンケート / 月次レポート / イベント企画案 / チェックイン / プロフィール |
| **モジュール内分離** | `screens/`, `components/`, `hooks/`, `navigation/`, `i18n/`, `store/` |
| **AI生成適合度** | ◎（パターン化されたコンポーネント実装）|

### U-BE-AUTH — 認証バックエンド

| 項目 | 値 |
|---|---|
| **目的** | Cognito 連携 + Lambda Authorizer（マルチテナント認可） |
| **デプロイ単位** | CDK AuthStack (Cognito) + ApiStack (Lambda) |
| **コード配置** | `backend/functions/auth/` + `backend/functions/auth-authorizer/` |
| **主要 API** | `POST /auth/signup`, `POST /auth/verify-school-code`, `POST /auth/refresh-token`, `DELETE /auth/me`, `POST /auth/admin/sign-in` |
| **モジュール内分離** | `signup.ts`, `verify-school-code.ts`, `refresh-token.ts`, `delete-me.ts`, `admin-sign-in.ts`, `lib/cognito-client.ts` |
| **責任 SECURITY ルール** | SECURITY-08 (IDOR), SECURITY-12 (認証), SECURITY-14 (アラート: ロックアウト) |

### U-BE-SUGGEST-BOX — 目安箱バックエンド

| 項目 | 値 |
|---|---|
| **目的** | 目安箱の投稿 CRUD + AI分類トリガー |
| **デプロイ単位** | CDK ApiStack (Lambda) + DataStack (DynamoDB Streams) |
| **コード配置** | `backend/functions/suggest-box/` + `backend/functions/streams/post-classification-trigger/` |
| **主要 API** | `POST /posts`, `GET /posts`, `GET /posts/:id`, `POST /posts/:id/support`, `PATCH /posts/:id/status` |
| **モジュール内分離** | `create-post.ts`, `list-posts.ts`, `support-post.ts`, `update-status.ts` + Stream Trigger |
| **AI連携** | Streams → SQS → U-AI-AGENTS の Sub-Classify |

### U-BE-SURVEY — アンケートバックエンド

| 項目 | 値 |
|---|---|
| **目的** | アンケート CRUD + 回答受付 + リアルタイム集計 |
| **デプロイ単位** | CDK ApiStack |
| **コード配置** | `backend/functions/survey/` |
| **主要 API** | `GET /surveys`, `GET /surveys/:id`, `POST /surveys/:id/answers`, `GET /surveys/:id/results`, `GET /surveys/:id/analysis` |

### U-BE-NOTIFICATION — 通知バックエンド

| 項目 | 値 |
|---|---|
| **目的** | プッシュ通知配信（SNS → FCM） |
| **デプロイ単位** | CDK ApiStack + NotificationStack |
| **コード配置** | `backend/functions/notification/` |
| **主要 API** | `POST /notifications/broadcast`, デバイストークン登録 + 内部 invoke API |

### U-BE-REPORT — 月次レポートバックエンド

| 項目 | 値 |
|---|---|
| **目的** | 月次レポートの取得 API |
| **デプロイ単位** | CDK ApiStack |
| **コード配置** | `backend/functions/report/` |
| **主要 API** | `GET /reports`, `GET /reports/:id` |

### U-BE-ADMIN — 管理者操作バックエンド

| 項目 | 値 |
|---|---|
| **目的** | 学校コード発行・保護者管理・公開操作ログ |
| **デプロイ単位** | CDK ApiStack |
| **コード配置** | `backend/functions/admin/` |
| **主要 API** | `POST /admin/school-codes`, `GET /admin/parents`, `DELETE /admin/parents/:id`, `GET /admin/post-trends`, `GET /public-operation-log` |

### U-BE-EVENT — イベントバックエンド

| 項目 | 値 |
|---|---|
| **目的** | イベント企画案・賛否投票・実施確定・チェックイン |
| **デプロイ単位** | CDK ApiStack + DataStack (Streams) |
| **コード配置** | `backend/functions/event/` + `backend/functions/streams/event-confirm-notifier/` |
| **主要 API** | 企画案系 (`/events/proposals/*`) + イベント系 (`POST /events/:id/confirm`, `POST /events/:id/check-in`, `GET /events/:id/check-ins`) |
| **モジュール内分離** | `proposals/`, `events/`, `check-in/` + Stream Trigger |

### U-AI-CORE — AI 共通基盤

| 項目 | 値 |
|---|---|
| **目的** | Bedrock SDK ラッパー + プロンプト管理 + ゴールデンテスト基盤 + Guardrails 設定 + **ローカル開発用モック実装**（追加要望6） |
| **デプロイ単位** | Lambda Layer（CDK AiStack 経由）または共通ライブラリ |
| **コード配置** | `backend/lib/ai/` |
| **主要モジュール** | `bedrock-client.ts`（**interface 定義**）, `bedrock-client-real.ts`（本番実装）, **`bedrock-client-mock.ts`（ローカル/テスト用モック実装）**, `bedrock-client-factory.ts`（環境変数 RUNTIME で切替）, `prompts/`, `golden-tests/`（Mock 利用）, `guardrails-config.ts` |
| **責任 OQ** | OQ-05 (Guardrails), OQ-06 (ゴールデンテストコーパス) |
| **ローカル開発適合度** | **◎**（ゴールデンテストセットを Mock の応答源として再利用可能、Construction で MockBedrockClient と GoldenTestRunner を同時実装）|

### U-AI-AGENTS — AI サブエージェント群（折衷案: 内部モジュール分離）

| 項目 | 値 |
|---|---|
| **目的** | 6つのサブエージェント Lambda + Monthly Trigger Lambda |
| **デプロイ単位** | CDK AiStack（7 Lambda 関数） |
| **コード配置** | `backend/functions/ai/` |
| **モジュール内分離（折衷案による必須要件）** | `sub-classify/` (Haiku), `sub-decision/` (Sonnet), `sub-generate/` (Sonnet), `sub-analyze/` (Sonnet), `sub-event/` (Sonnet), `sub-report/` (Sonnet), `monthly-report-trigger/` |
| **Functional Design 必須** | 6エージェント分を章立てて個別詳細記述 |
| **Code Generation Plan** | 各サブエージェントを段階的に着手可能な構成（例: 1.分類 → 2.配信判断 → 3.生成 → 4.分析 → 5.企画 → 6.レポート の順）|
| **依存** | U-AI-CORE の bedrock-client / prompts を必須利用 |

### U-AI-ORCHESTRATION — AI ワークフロー基盤

| 項目 | 値 |
|---|---|
| **目的** | Step Functions State Machine + EventBridge スケジュール |
| **デプロイ単位** | CDK AiStack (Step Functions) + NotificationStack (EventBridge) |
| **コード配置** | `infra/cdk/stepfunctions/daily-scheduler.ts` + EventBridge ルール定義 |
| **主要構成** | DailyScheduler State Machine (4 ステップ) + 月初 EventBridge ルール |
| **依存** | U-AI-AGENTS の Lambda ARN（Step Functions ステップ参照）|

### U-SHARED — 共通パッケージ

| 項目 | 値 |
|---|---|
| **目的** | フロントエンド・バックエンド両方で使う共通ライブラリ |
| **デプロイ単位** | npm workspace（pnpm + Turborepo）= ビルド時に依存解決 |
| **コード配置** | `packages/shared-types/`, `packages/shared-ui/`, `packages/shared-api-client/`, `packages/shared-auth/`, `packages/shared-i18n/` |
| **モジュール内分離** | 5パッケージで明確に分離（ユニット内部だが各々独立 npm パッケージ）|
| **責任 PBT ルール** | PBT-02 (ラウンドトリップ), PBT-07 (ジェネレータ品質) |
| **責任 SECURITY ルール** | SECURITY-05 (入力バリデーション基盤) |

### U-INFRA — インフラストラクチャ（AWS デプロイ）

| 項目 | 値 |
|---|---|
| **目的** | AWS リソース全体を CDK で定義 + EAS Build IAM 連携 |
| **デプロイ単位** | CDK Bootstrap → CDK Deploy（10スタック）|
| **コード配置** | `infra/cdk/` |
| **モジュール内分離** | `stacks/network-stack.ts`, `stacks/auth-stack.ts`, `stacks/data-stack.ts`, `stacks/api-stack.ts`, `stacks/ai-stack.ts`, `stacks/notification-stack.ts`, `stacks/monitoring-stack.ts`, `stacks/secrets-stack.ts`, `stacks/frontend-stack.ts`, `stacks/mobile-build-stack.ts` |
| **責任 SECURITY ルール** | SECURITY-01 (暗号化), 02 (ネットワークログ), 04 (ヘッダー), 06 (最小権限), 07 (ネットワーク制限), 09 (ハードニング), 10 (サプライチェーン), 14 (アラート) |
| **責任 NFR** | NFR-MON (CloudWatch + X-Ray + Slack) |
| **F-11 担当** | MobileBuildStack で EAS Build 連携 IAM ロール定義 |

### U-LOCAL — ローカル開発環境（追加要望6 で新設）

| 項目 | 値 |
|---|---|
| **目的** | ローカルマシン上で全機能の動作確認を可能にする（AWSデプロイ不要）|
| **デプロイ単位** | なし（開発者マシンで直接実行）|
| **コード配置** | `infra/local/` + `backend/local-server/` |
| **モジュール内分離** | `infra/local/docker-compose.yml`（DynamoDB Local + LocalStack + cognito-local）/ `infra/local/init-tables.ts` / `infra/local/seed-data.ts` / `infra/local/local-orchestrator.ts`（Step Functions 代替）/ `infra/local/trigger-daily-scheduler.ts` / `infra/local/trigger-monthly-report.ts` / `infra/local/issue-fake-jwt.ts` / `backend/local-server/index.ts`（Express 統合サーバ）/ `backend/local-server/local-auth-middleware.ts` |
| **依存** | U-AI-CORE の MockBedrockClient、全 Backend Service の Express ルータ、U-SHARED の型定義 |
| **担当 npm scripts** | `local:up`, `local:down`, `local:init`, `local:server`, `local:web`, `local:mobile`, `local:trigger-daily`, `local:trigger-monthly` |
| **想定使用シーン** | MVPリリース前の動作確認、E2Eテストのローカル実行、AI機能のUIフィードバック確認、新規開発者のオンボーディング |

---

## 3. コード組織戦略（Greenfield モノレポ構造）

### モノレポ全体構造

```
oyano-wa/
├── apps/                              # フロントエンドアプリケーション
│   ├── web-admin/                     # U-FE-ADMIN
│   │   ├── app/                       # Next.js App Router
│   │   ├── components/
│   │   ├── lib/
│   │   ├── public/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── next.config.ts
│   │   └── tailwind.config.ts
│   └── app-parent/                    # U-FE-PARENT
│       ├── app/                       # Expo Router
│       ├── components/
│       ├── hooks/
│       ├── store/
│       ├── i18n/
│       ├── package.json
│       ├── app.json                   # Expo 設定
│       ├── eas.json                   # EAS Build 設定
│       └── tsconfig.json
│
├── backend/                           # バックエンド Lambda
│   ├── functions/
│   │   ├── auth/                      # U-BE-AUTH (主)
│   │   ├── auth-authorizer/           # U-BE-AUTH (認可)
│   │   ├── suggest-box/               # U-BE-SUGGEST-BOX
│   │   ├── survey/                    # U-BE-SURVEY
│   │   ├── notification/              # U-BE-NOTIFICATION
│   │   ├── report/                    # U-BE-REPORT
│   │   ├── admin/                     # U-BE-ADMIN
│   │   ├── event/                     # U-BE-EVENT
│   │   ├── ai/                        # U-AI-AGENTS
│   │   │   ├── sub-classify/          # ★モジュール分離（折衷案）
│   │   │   ├── sub-decision/
│   │   │   ├── sub-generate/
│   │   │   ├── sub-analyze/
│   │   │   ├── sub-event/
│   │   │   ├── sub-report/
│   │   │   └── monthly-report-trigger/
│   │   └── streams/
│   │       ├── post-classification-trigger/  # U-BE-SUGGEST-BOX (Stream)
│   │       └── event-confirm-notifier/        # U-BE-EVENT (Stream)
│   └── lib/
│       ├── ai/                        # U-AI-CORE
│       │   ├── bedrock-client.ts
│       │   ├── prompts/
│       │   ├── golden-tests/
│       │   └── guardrails-config.ts
│       ├── auth/                      # 認証共通ロジック
│       ├── db/                        # DynamoDB ラッパー
│       └── utils/                     # 共通ユーティリティ
│
├── packages/                          # U-SHARED (5サブパッケージ)
│   ├── shared-types/                  # Zod schemas, 型定義
│   ├── shared-ui/                     # shadcn ベース共通 UI
│   ├── shared-api-client/             # TanStack Query hooks
│   ├── shared-auth/                   # Cognito ラッパー
│   └── shared-i18n/                   # i18next 設定 + ja/en JSON
│
├── infra/                             # U-INFRA
│   └── cdk/
│       ├── stacks/                    # 10スタック
│       ├── stepfunctions/             # U-AI-ORCHESTRATION
│       │   └── daily-scheduler.ts
│       ├── bin/                       # CDK エントリ
│       ├── lib/
│       │   └── constructs/            # 共通 CDK Construct
│       └── cdk.json
│   └── local/                         # ★追加要望6: U-LOCAL
│       ├── docker-compose.yml         # DynamoDB Local + LocalStack + cognito-local
│       ├── init-tables.ts             # 初期テーブル作成
│       ├── seed-data.ts               # シードデータ投入
│       ├── local-orchestrator.ts      # Step Functions 代替
│       ├── trigger-daily-scheduler.ts # EventBridge 9時 代替
│       ├── trigger-monthly-report.ts  # EventBridge 月初 代替
│       └── issue-fake-jwt.ts          # cognito-local fake JWT 発行ヘルパ
│
├── backend/local-server/              # ★追加要望6: U-LOCAL
│   ├── index.ts                       # Express 統合サーバ (port 3001)
│   ├── local-auth-middleware.ts       # Lambda Authorizer の Express 版
│   └── stream-emulator.ts             # DynamoDB Streams 代替（手動 invoke）
│
├── docs/
│   └── api/                           # OpenAPI 仕様
│
├── .github/
│   └── workflows/                     # GitHub Actions (Lint・Unit テスト・CodeBuild トリガー)
│
├── package.json                       # ワークスペースルート
├── pnpm-workspace.yaml
├── turbo.json                         # Turborepo 設定
├── tsconfig.base.json                 # 共通 TS 設定
├── .gitignore
└── README.md
```

### コード組織の原則

1. **モノレポ管理**: pnpm workspaces + Turborepo（ビルドキャッシュ・依存最適化）
2. **TypeScript 厳格**: `strict: true`, `noImplicitAny: true`, `strictNullChecks: true`
3. **ユニット境界 = ディレクトリ境界**: 各ユニットは固有のディレクトリツリーを持つ
4. **共有型は packages/shared-types に集約**: Zod スキーマで型と検証を兼用
5. **AI生成適合**: AI による Code Generation を最大化するため:
   - 各機能エリア内のファイル命名規則を統一（`create-*.ts`, `list-*.ts`, `delete-*.ts`）
   - インポート構造を明示的に（barrel export の `index.ts` 必須）
   - テストファイル配置規則 (`*.test.ts`, `*.pbt.test.ts`) を統一

---

## 4. デプロイ単位とライフサイクル

### A-5 A: 共通 CDK デプロイ採用

| デプロイ対象 | デプロイ手段 | トリガー |
|---|---|---|
| AWS リソース全体（10スタック）| `cdk deploy --all` | mainブランチへのマージ → CodePipeline |
| 管理者 Web (U-FE-ADMIN) | Next.js build → S3 upload → CloudFront invalidation | 同上 |
| Android アプリ (U-FE-PARENT) | EAS Build → .apk → アーティファクトとして保存 | mainブランチへのマージ → 別ジョブ |
| プロンプト・ゴールデンテスト (U-AI-CORE) | コードと一緒に CDK デプロイ | 同上 |

### 環境
- **MVP**: dev のみ（質問9 C）
- **Phase 2**: staging / prod 追加（OQ-01 で確定）

---

## 5. 所有権境界とチーム構成（A-6 D 採用: AI主導開発・人間レビュー）

### 開発モデル

```
┌─────────────────────────────────────────────────────────┐
│ AI エージェント（Claude Agent SDK 等）                    │
│   └─ ユニット単位で per-unit Construction ループを実行    │
│      ├─ Functional Design 生成                          │
│      ├─ NFR Requirements 生成                            │
│      ├─ NFR Design 生成                                  │
│      ├─ Infrastructure Design 生成                       │
│      └─ Code Generation 実行                             │
└─────────────────────────────────────────────────────────┘
              ↓ 各ステージごとに承認依頼
┌─────────────────────────────────────────────────────────┐
│ 人間レビュアー（1〜3名）                                  │
│   ├─ Functional Design レビュー                          │
│   ├─ Code レビュー                                        │
│   └─ E2E 動作確認                                         │
└─────────────────────────────────────────────────────────┘
```

### 推奨実行順序（依存関係に基づく）

`unit-of-work-dependency.md` で詳述しますが、Critical Path 順に:

```
Phase 1 (基盤): U-SHARED → U-AI-CORE → U-INFRA (基盤部分) → U-LOCAL（早期着手で開発者体験を立ち上げ、追加要望6）
Phase 2 (認証): U-BE-AUTH (Authorizer 含む)
Phase 3 (バックエンドコア): U-BE-SUGGEST-BOX → U-BE-SURVEY → U-BE-NOTIFICATION → U-BE-REPORT → U-BE-ADMIN → U-BE-EVENT
Phase 4 (AI連携): U-AI-AGENTS → U-AI-ORCHESTRATION
Phase 5 (フロントエンド): U-FE-ADMIN, U-FE-PARENT （並列）
Phase 6 (インフラ完成): U-INFRA (フロントエンド配信・モバイルビルド)
```

### AI主導の妥当性

| ユニット | AI生成適合度 | 理由 |
|---|---|---|
| U-FE-ADMIN | ◎ | shadcn/ui 定型実装、TS 厳格型 |
| U-FE-PARENT | ◎ | 同様にパターン化 |
| U-BE-* | ◎ | Express on Lambda + Zod の定型 |
| U-AI-CORE | ○ | プロンプト品質は人間レビュー必須 |
| U-AI-AGENTS | ○ | 6エージェントのテンプレ実装は AI 得意、プロンプト調整は人間 |
| U-AI-ORCHESTRATION | ◎ | CDK + Step Functions 定義は AI 得意 |
| U-SHARED | ◎ | Zod スキーマ・i18n JSON は AI 大量生成 |
| U-INFRA | ○〜◎ | CDK は AI 生成可能だが、IAM・WAF 設定は人間レビュー必須 |

---

## 6. 受け入れ基準のユニット担当マッピング

requirements.md §13 の受け入れ基準と各ユニットの担当:

| 受け入れ基準 | 主担当ユニット |
|---|---|
| F-01〜F-11 dev 環境動作 | 全ユニット |
| SECURITY 全15ルール適合 | U-INFRA + U-SHARED + U-BE-AUTH + U-AI-CORE |
| PBT 適用ルール（02/03/07/08/09）| U-SHARED + U-BE-SUGGEST-BOX + U-BE-EVENT + U-AI-AGENTS |
| AIエージェント分離（オーケストレーター + 6サブエージェント）| U-AI-ORCHESTRATION + U-AI-AGENTS + U-AI-CORE |
| 毎日9時 EventBridge チェーン動作 | U-AI-ORCHESTRATION + U-AI-AGENTS |
| イベント企画 → 投票 → チェックイン フロー | U-BE-EVENT + U-FE-PARENT + U-FE-ADMIN |
| プロンプトのゴールデンテスト CI 実行 | U-AI-CORE |
| CloudWatch Dashboard 可視化 | U-INFRA |
| Slack アラート発火確認 | U-INFRA |
| 日英 UI 主要画面表示 | U-SHARED + U-FE-ADMIN + U-FE-PARENT |
| CI/CD パイプライン動作 | U-INFRA + GitHub Actions 設定 |
| 参加者チェックイン (F-10) | U-BE-EVENT + U-FE-PARENT + U-FE-ADMIN |
| Android .apk 自動ビルド (F-11) | U-INFRA (MobileBuildStack) + U-FE-PARENT |
| iOS Phase 3 活性化可能性 | U-FE-PARENT (Expo 採用で確保) |
