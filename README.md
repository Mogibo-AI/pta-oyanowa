# 全自動PTA - おやのわ

「おやのわ」は、PTA活動から人間が担う調整・連絡・記録・報告の負担をできるだけ取り除くための、学校単位のSaaSです。

保護者は会議や役員業務に参加する代わりに、スマホから意見投稿・アンケート回答・企画への投票などを行います。AIが投稿分類、アンケート生成、結果分析、イベント企画案作成、月次レポート生成を支援します。

## MVPスコープ

MVPでは以下を対象にします。

- デジタル目安箱
  - テキスト投稿、匿名/実名選択、AIカテゴリ分類、投稿一覧、支持、対応状況表示
- AIアンケート
  - 毎日9時の配信判断、アンケート自動生成、プッシュ通知、回答、集計グラフ、結果分析
- 月次レポート
  - 月初のAIレポート生成、保護者/管理者向け閲覧
- 認証
  - Cognitoメール認証、学校コードによる招待、保護者/管理者プール分離
- 管理者画面
  - 学校コード発行、保護者管理、投稿確認、即時通知、企画案確認、イベント管理
- 多言語UI
  - 主要UIの日本語/英語対応
- イベント企画AI
  - アンケート結果と目安箱投稿から企画案を生成し、保護者が賛成/反対/興味なしで投票
- 参加者チェックイン
  - 実施確定イベントで、QRまたはタップ式チェックインと参加状況確認
- Coming Soon UI
  - Phase 2/3予定機能への導線を準備中として表示
- Androidアプリ配信
  - React Native + Expoで保護者向けAndroidアプリを作成し、.apkビルドまで対応

MVPではiOSアプリのリリースは対象外です。Web版保護者画面はPhase 2、iOSリリースはPhase 3で対応予定です。

## 対象ユーザー

- 保護者
  - 目安箱投稿、アンケート回答、月次レポート閲覧、イベント企画への投票、チェックイン
- 学校管理者
  - 学校コード発行、保護者管理、投稿/アンケート/企画/イベントの確認、即時通知
- PTA担当の先生
  - MVPでは限定的な確認・連絡受信が中心
- 新入学保護者
  - QRコード、学校コード、メール認証による初回オンボーディング

## 技術方針

想定構成はTypeScript中心のモノレポです。

- 管理者Web: Next.js 16 App Router、React、shadcn/ui、TanStack Query、Zustand
- 保護者アプリ: React Native、Expo、EAS Build、React Navigation
- Backend: AWS Lambda、Express、API Gateway、Lambda Authorizer
- Data/Auth: DynamoDBシングルテーブル、DynamoDB Streams、S3、Cognito
- AI: Amazon Bedrock、Claude Sonnet/Haiku、Step Functions、EventBridge、SQS
- Notification: SNS、FCM
- Infrastructure: AWS CDK
- Monitoring: CloudWatch、X-Ray、Slack通知
- Test: Jest、React Testing Library、Supertest、Playwright、CDK Assertions、fast-check

## 想定モノレポ構成

```text
.
├── apps/
│   ├── web-admin/        # 管理者向けWeb
│   └── app-parent/       # 保護者向けAndroidアプリ
├── backend/
│   ├── functions/        # 機能別Lambda
│   ├── lib/              # AI/Auth/DBなどの共通処理
│   └── layers/           # Lambda Layer
├── packages/
│   ├── shared-types/
│   ├── shared-ui/
│   ├── shared-api-client/
│   ├── shared-auth/
│   └── shared-i18n/
├── infra/
│   └── cdk/              # AWS CDK
└── aidlc-docs/           # AI-DLC設計ドキュメント
```

## 開発ユニット

AI-DLC上では、以下の15ユニットに分割して設計しています。

- U-SHARED: 共通型、UI、APIクライアント、認証、i18n
- U-INFRA: AWS CDK、各種スタック、監視、モバイルビルド
- U-LOCAL: ローカル開発環境
- U-BE-AUTH: 認証、学校コード、Lambda Authorizer
- U-BE-SUGGEST-BOX: 目安箱
- U-BE-SURVEY: アンケート
- U-BE-NOTIFICATION: 通知
- U-BE-REPORT: 月次レポート
- U-BE-ADMIN: 管理者操作
- U-BE-EVENT: イベント企画、投票、チェックイン
- U-AI-CORE: Bedrockラッパー、プロンプト、Guardrails、ゴールデンテスト
- U-AI-AGENTS: 投稿分類、配信判断、質問生成、結果分析、企画生成、レポート生成
- U-AI-ORCHESTRATION: Step Functions、EventBridge
- U-FE-ADMIN: 管理者Web
- U-FE-PARENT: 保護者Androidアプリ

推奨開発順序は、共通/基盤、認証、バックエンド各機能、AI、フロントエンド、配信/ビルド基盤の順です。

## 参照ドキュメント

詳細は `aidlc-docs/` 配下を参照してください。

- 要件定義: `aidlc-docs/inception/requirements/requirements.md`
- ペルソナ: `aidlc-docs/inception/user-stories/personas.md`
- ユーザーストーリー: `aidlc-docs/inception/user-stories/stories.md`
- 実行計画: `aidlc-docs/inception/plans/execution-plan.md`
- アプリケーション設計: `aidlc-docs/inception/application-design/application-design.md`
- コンポーネント定義: `aidlc-docs/inception/application-design/components.md`
- メソッド定義: `aidlc-docs/inception/application-design/component-methods.md`
- サービス設計: `aidlc-docs/inception/application-design/services.md`
- ユニット定義: `aidlc-docs/inception/application-design/unit-of-work.md`
- ユニット依存関係: `aidlc-docs/inception/application-design/unit-of-work-dependency.md`
- ユーザーストーリーマッピング: `aidlc-docs/inception/application-design/unit-of-work-story-map.md`

## 現在の位置づけ

このリポジトリは、AI-DLCのINCEPTION成果物をもとにMVP実装へ進むための設計段階です。実装時は、READMEに記載した構成と `aidlc-docs/` の設計を基準に、Constructionフェーズのユニット単位で進めます。
