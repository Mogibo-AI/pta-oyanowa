# テクニカル環境ドキュメント — 全自動PTA - おやのわ

> AI-DLC インプット用テクニカル環境ドキュメント（最大版）
>  マークは元ファイルに記載のなかった新規追加内容です

---

## Languages (言語)

### 必須

| 言語 | バージョン | 用途 |
|-----|-----------|------|
| TypeScript | 5.x 以上 | フロントエンド・バックエンド全般 |
| Node.js | 20.x LTS 以上 | バックエンドランタイム |

### 任意（状況次第で使用）

| 言語 | バージョン | 用途 | 条件 |
|-----|-----------|------|------|
| Python | 3.12以上 | Bedrock連携の一部・データ変換バッチ | TypeScriptで対応困難な場合のみ |

### 禁止

| 言語 | 理由 |
|-----|------|
| PHP | 採用スタックに不一致 |
| Ruby | 採用スタックに不一致 |

---

## Frameworks and Libraries (フレームワーク・ライブラリ)

### フロントエンド（必須）

| 種別 | ライブラリ/FW | バージョン | 用途 | 備考 |
|-----|-------------|-----------|------|------|
| Web（管理者画面・保護者向け） | React | 18.x以上 | UIコンポーネント | |
| Web フレームワーク | Next.js | 14.x以上（App Router） | SSR・ルーティング | |
| アプリ（iOS/Android） | React Native | 0.74以上 | 保護者向けアプリ | Expo使用を検討 |
| UIコンポーネントライブラリ | TBD (shadcn/ui or MUI) | TBD | 共通UIコンポーネント | 選定中 |
| 状態管理 | Zustand | 最新版 | グローバル状態管理 | Reduxは不要な複雑さのため不採用 |
| HTTPクライアント | Axios or fetch（標準） | — | API通信 | — |
| グラフ | Recharts | 最新版 | アンケート結果・会費グラフ | — |
| カレンダー | TBD (react-big-calendar or fullcalendar) | TBD | 機能7 カレンダー表示 | 選定中 |

### バックエンド（必須）

| ライブラリ/FW | バージョン | 用途 | 備考 |
|-------------|-----------|------|------|
| Express または Fastify | 最新安定版 | REST API サーバー | Hono は使用しない（後述） |
| Zod | 最新版 | 入力バリデーション・スキーマ定義 | — |
| AWS SDK v3 (@aws-sdk/*) | v3.x以上 | 全AWSサービスとの通信 | モジュラー設計で必要なサービスのみimport |
| @anthropic-ai/sdk | 最新版 | Bedrock経由でのClaude呼び出し | Bedrock対応版を使用 |
| jsonwebtoken | 最新版 | JWT検証（Cognitoトークン） | — |

### 禁止ライブラリ・フレームワーク

| 禁止 | 理由 | 代わりに使うもの |
|-----|------|----------------|
| **Hono** | バックエンドとして採用しない（ユーザー指示） | Express または Fastify |
| **Supabase** | AWSに統一するため。PostgreSQLはAurora、認証はCognito | Amazon Aurora / Amazon Cognito |
| **Firebase** | AWSに統一するため。プッシュ通知はSNS+FCM（FCMはAPN経由のみ） | Amazon SNS + APNs/FCM |
| **Prisma（RDB ORM）** | DynamoDBが主DBのため不要。Auroraが必要になった場合のみ検討 | DynamoDB DocumentClient / Aurora: knex or raw SQL |
| **Redux** | このアプリの規模に対して複雑すぎる | Zustand |
| **moment.js** | メンテナンス終了・バンドルサイズが大きい | date-fns または Day.js |

---

## Cloud Services — AWS Only (クラウドサービス — AWS 統一)

### 使用するサービス（許可リスト）

| サービス | 用途 | 備考 |
|---------|------|------|
| **Amazon Bedrock** | Claude呼び出し（AI機能全般） | claude-sonnet-4-6（高精度）、claude-haiku-4-5（コスト重視）を使い分け |
| **AWS Lambda** | バックエンドAPI・バッチ処理 | Node.js 20.x ランタイム |
| **Amazon API Gateway** | REST APIのエンドポイント管理 | Lambda統合 |
| **Amazon DynamoDB** | 主DB（学校ごとにテーブル分離） | オンデマンドキャパシティ推奨 |
| **Amazon Aurora** | サブDB（検索・集計が多い場合のみ） | DynamoDBで対応困難な複雑クエリが必要になった場合に導入 |
| **Amazon S3** | レポートPDF・画像・静的ファイル保存 | バケットポリシーで学校ごとにプレフィックス分離 |
| **Amazon Cognito** | 保護者・管理者の認証 | ユーザープール分離（保護者 / 管理者） |
| **Amazon SES** | メール通知（目安箱の通知・学校への通知） | 送信ドメイン認証必須 |
| **Amazon SNS** | プッシュ通知（FCM/APNs連携） | アンケート配信・タスク通知 |
| **Amazon CloudFront** | CDN（WebアプリとS3の配信） | WAF連携 |
| **AWS WAF** | Webアプリケーションファイアウォール | CloudFrontに付与 |
| **AWS CDK** | インフラコード定義（TypeScript） | すべてのインフラをコード管理 |
| **Amazon CloudWatch** | ログ・メトリクス・アラート | Lambda・API Gatewayのログ集約 |
| **AWS Secrets Manager** | APIキー・DB接続情報の管理 | ハードコード禁止 |
| **AWS IAM** | 権限管理 | 最小権限の原則 |
| **Amazon EventBridge** | 月次アンケート自動配信・定期バッチのスケジューリング | CronベースのLambdaトリガー |
| **Amazon DynamoDB Streams** | 投稿・回答のイベント駆動処理 | 目安箱投稿時のAI分類トリガー等 |
| **AWS X-Ray** | 分散トレーシング（Bedrock呼び出しのレイテンシ計測含む） | 本番環境で必須 |
| **Amazon SQS** | 非同期処理キュー（AI処理の遅延処理） | Bedrock呼び出しをキューイングして過負荷を防ぐ |

### 使用しないサービス（禁止リスト）

| サービス | 理由 | 代わりに使うもの |
|---------|------|----------------|
| Amazon EC2 | サーバーレス優先 | Lambda |
| Amazon ECS / EKS | このフェーズでは不要 | Lambda |
| Google Cloud / Azure | AWSに統一 | AWSの対応サービス |
| Vercel / Netlify（本番） | AWSに統一（開発環境での利用は可） | CloudFront + S3 / Amplify Hosting |

---

## Architecture and Patterns (アーキテクチャとパターン)

### API スタイル

- **REST API**（API Gateway + Lambda）
- 認証: Cognito JWT トークン（Authorizationヘッダー Bearer）
- レスポンス形式: JSON、エラーは `{ error: { code, message } }` で統一

### データパターン

- **主DB: DynamoDB**（学校ごとにテーブル分離 or パーティションキーに学校IDを含める）
- **サブDB: Aurora**（複雑な集計クエリが必要になった場合のみ導入）
- **シングルテーブルデザイン**を基本方針とし、アクセスパターン優先でGSIを設計する

### マルチテナント設計

- 学校ごとにデータを分離する（他校のデータにアクセスできない設計）
- DynamoDBのパーティションキーに `schoolId` を含める
- S3はプレフィックスで学校ごとに分離（`/{schoolId}/...`）
- Cognitoユーザープールは保護者/管理者で分離する

### イベント駆動

- 目安箱への投稿 → DynamoDB Streams → Lambda（AI分類・グルーピング処理）
- アンケート回答 → DynamoDB Streams → Lambda（集計リアルタイム更新）
- 月次アンケート自動配信 → EventBridge（Cron） → Lambda → SES/SNS

### プロジェクト構造


```
oyano-wa/
├── apps/
│   ├── web-admin/         # Next.js: 管理者Webアプリ
│   ├── web-parent/        # Next.js: 保護者WebアプリKei（Phase 2）
│   └── app-parent/        # React Native: 保護者モバイルアプリ
├── packages/
│   ├── api-client/        # 共通APIクライアント（型定義含む）
│   ├── ui/                # 共通UIコンポーネント
│   └── shared/            # 共通型・ユーティリティ
├── backend/
│   ├── functions/         # Lambda関数（機能別）
│   ├── layers/            # Lambda Layer（共通依存）
│   └── lib/               # 共通ビジネスロジック
├── infra/
│   └── cdk/               # AWS CDK (TypeScript)
└── docs/
    └── api/               # OpenAPI仕様
```


---

## Security (セキュリティ)

### 認証・認可

| 対象 | 認証方式 | 備考 |
|-----|---------|------|
| 保護者 | Amazon Cognito（メール認証）+ 学校コード | SNS連携なし |
| 学校管理者 | Amazon Cognito（メール認証）+ 二要素認証（TOTP） | 強制MFA |
| Lambda関数間の通信 | IAMロール（最小権限） | — |

### 暗号化

- 通信: HTTPS（TLS 1.2以上）必須
- DynamoDB: 保管時暗号化（AWS Managed Key）
- S3: サーバーサイド暗号化（SSE-S3）
- Secrets Manager: KMSによる暗号化

### 入力バリデーション

- すべてのAPIエンドポイントでZodによるスキーマ検証を実施
- 目安箱の投稿内容: 文字数上限・XSS対策（HTMLエスケープ）
- SQLインジェクション: DynamoDB使用のためリスク低いが、Auroraを使う場合はプリペアドステートメント必須

### シークレット管理

- **ハードコード禁止**（Bedrock APIキー・DB接続情報・外部サービスキーなど）
- すべてのシークレットは AWS Secrets Manager に格納
- Lambda関数は起動時にSecrets Managerから取得（キャッシュを活用し呼び出し回数を削減）

### 操作ログ（透明性担保）

- 会費変更・提案削除などの破壊的操作は「公開操作ログ」としてDynamoDBに記録し保護者が閲覧可能
- 全管理者操作はCloudTrailにも記録（監査用）
- AIによる管理者設定の自動変更は仕様上不可能とする（Lambda側で制御）

### コンプライアンス目標

- ISMS準拠・ISO 27001認証を目指す
- 個人情報保護法対応: 学校単位のデータ分離・退会時データ削除オプション
- 児童の個人情報・成績情報はシステムに保持しない（設計制約）

---

## Testing (テスト)

### テスト種別と方針

| 種別 | ツール | カバレッジ目標 | 対象 |
|-----|-------|--------------|------|
| 単体テスト | Jest + ts-jest | 80%以上（ビジネスロジック） | バックエンド共通ロジック・AI呼び出し部 |
| コンポーネントテスト | Jest + React Testing Library | 70%以上 | Webアプリのコンポーネント |
| E2Eテスト | Playwright | 主要フロー100% | 目安箱投稿・アンケート回答・管理者操作の主要パス |
| APIテスト | Jest + Supertest | 全エンドポイント | Lambda関数のHTTPレスポンス検証 |
| インフラテスト | AWS CDK Assertions | 全スタック | CDKスタックのスナップショットテスト |

### CI/CDゲート


- PR作成時: 単体テスト・Lintが必須（通らなければマージ不可）
- mainブランチへのマージ時: E2Eテストも必須
- デプロイ前: CDKのdiff確認・セキュリティスキャン（npm audit）


---

## Example Code (サンプルコード)

### Lambda 関数の基本構造


```typescript
// backend/functions/suggest-box/handler.ts
import { APIGatewayProxyHandlerV2 } from 'aws-lambda';
import { z } from 'zod';
import { createPost } from '../../lib/suggest-box/create-post';
import { getUserFromToken } from '../../lib/auth/get-user';

const PostSchema = z.object({
  content: z.string().min(1).max(500),
  isAnonymous: z.boolean().default(false),
});

export const handler: APIGatewayProxyHandlerV2 = async (event) => {
  try {
    const user = await getUserFromToken(event.headers.authorization ?? '');
    const body = PostSchema.parse(JSON.parse(event.body ?? '{}'));

    const post = await createPost({
      schoolId: user.schoolId,
      userId: user.userId,
      content: body.content,
      isAnonymous: body.isAnonymous,
    });

    return {
      statusCode: 201,
      body: JSON.stringify(post),
    };
  } catch (err) {
    if (err instanceof z.ZodError) {
      return { statusCode: 400, body: JSON.stringify({ error: { code: 'VALIDATION_ERROR', message: err.message } }) };
    }
    return { statusCode: 500, body: JSON.stringify({ error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } }) };
  }
};
```


### DynamoDB 操作例（DocumentClient v3）


```typescript
// backend/lib/suggest-box/create-post.ts
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';
import { randomUUID } from 'crypto';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

export async function createPost(params: {
  schoolId: string;
  userId: string;
  content: string;
  isAnonymous: boolean;
}) {
  const postId = randomUUID();
  const now = new Date().toISOString();

  const item = {
    PK: `SCHOOL#${params.schoolId}`,
    SK: `POST#${postId}`,
    GSI1PK: `SCHOOL#${params.schoolId}#POSTS`,
    GSI1SK: now,
    postId,
    schoolId: params.schoolId,
    userId: params.isAnonymous ? 'ANONYMOUS' : params.userId,
    content: params.content,
    isAnonymous: params.isAnonymous,
    status: 'PENDING',
    supportCount: 0,
    createdAt: now,
  };

  await ddb.send(new PutCommand({
    TableName: process.env.TABLE_NAME!,
    Item: item,
  }));

  return item;
}
```


### Amazon Bedrock（Claude）呼び出し例


```typescript
// backend/lib/ai/classify-post.ts
import { BedrockRuntimeClient, InvokeModelCommand } from '@aws-sdk/client-bedrock-runtime';

const bedrock = new BedrockRuntimeClient({ region: process.env.AWS_REGION });

export async function classifyPost(content: string): Promise<{
  category: 'SAFETY' | 'EVENT' | 'CONTACT' | 'REQUEST' | 'OTHER';
  summary: string;
}> {
  const prompt = `
以下のPTA目安箱への投稿内容を分類してください。

投稿内容: "${content}"

以下のカテゴリから最も適切なものを1つ選び、JSON形式で返してください:
- SAFETY（安全・防犯）
- EVENT（イベント・行事）
- CONTACT（連絡・情報共有）
- REQUEST（要望・改善提案）
- OTHER（その他）

返答形式: { "category": "カテゴリ名", "summary": "投稿の要約（20文字以内）" }
`;

  const response = await bedrock.send(new InvokeModelCommand({
    modelId: 'anthropic.claude-haiku-4-5-20251001', // コスト重視の分類タスク
    contentType: 'application/json',
    accept: 'application/json',
    body: JSON.stringify({
      anthropic_version: 'bedrock-2023-05-31',
      max_tokens: 256,
      messages: [{ role: 'user', content: prompt }],
    }),
  }));

  const result = JSON.parse(new TextDecoder().decode(response.body));
  return JSON.parse(result.content[0].text);
}
```


### React コンポーネント例（目安箱投稿フォーム）


```tsx
// apps/web-parent/components/SuggestBox/PostForm.tsx
'use client';
import { useState } from 'react';
import { usePostToSuggestBox } from '../../hooks/useSuggestBox';

export function PostForm() {
  const [content, setContent] = useState('');
  const [isAnonymous, setIsAnonymous] = useState(false);
  const { mutate, isPending } = usePostToSuggestBox();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!content.trim()) return;
    mutate({ content, isAnonymous }, {
      onSuccess: () => setContent(''),
    });
  };

  return (
    <form onSubmit={handleSubmit} className="p-4">
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="気になることや意見を書いてください"
        maxLength={500}
        className="w-full border rounded p-2 resize-none"
        rows={3}
      />
      <div className="flex items-center gap-4 mt-2">
        <label className="flex items-center gap-1 text-sm">
          <input
            type="checkbox"
            checked={isAnonymous}
            onChange={(e) => setIsAnonymous(e.target.checked)}
          />
          匿名で投稿する
        </label>
        <button
          type="submit"
          disabled={isPending || !content.trim()}
          className="ml-auto bg-teal-600 text-white px-4 py-2 rounded disabled:opacity-50"
        >
          {isPending ? '送信中...' : '投稿する'}
        </button>
      </div>
    </form>
  );
}
```


---


## CI/CD Design (CI/CD 設計)

### パイプライン（GitHub Actions）

```yaml
# .github/workflows/ci.yml の概要
on: [push, pull_request]
jobs:
  lint-and-test:
    - TypeScript コンパイルチェック (tsc --noEmit)
    - ESLint
    - Jest 単体テスト + カバレッジレポート

  e2e:
    - Playwright E2Eテスト（mainブランチへのPRのみ）

  deploy-dev:
    - on: push to develop ブランチ
    - cdk deploy --context env=dev

  deploy-prod:
    - on: push to main ブランチ（手動承認あり）
    - cdk deploy --context env=prod
```

### デプロイ環境

| 環境 | ブランチ | 目的 |
|-----|---------|------|
| dev | develop | 開発・動作確認 |
| staging | release/* | リリース前検証 |
| prod | main | 本番 |


---


## Monitoring Design (モニタリング設計)

### CloudWatch ダッシュボード（主要メトリクス）

| メトリクス | アラート閾値 | 通知先 |
|---------|-----------|-------|
| Lambda エラー率 | 1%以上 | 開発者Slack（TBD） |
| API Gateway 5xx 率 | 0.5%以上 | 開発者Slack（TBD） |
| Bedrock 呼び出しレイテンシ | 5秒以上（p99） | 開発者Slack（TBD） |
| DynamoDB ThrottledRequests | 1件以上 | 開発者Slack（TBD） |
| SES バウンス率 | 5%以上 | 開発者Slack（TBD） |

### ログ設計

- すべてのLambda関数は構造化ログ（JSON）を出力
- `schoolId` をすべてのログに含め、学校ごとのフィルタリングを可能にする
- 個人情報（メールアドレス・ユーザーID）はログに出力しない

### X-Ray トレーシング

- Lambda → DynamoDB → Bedrock の呼び出しチェーンをトレース
- AIの応答時間をメトリクスとして記録し、レイテンシの改善に活用


---


## Development Environment Setup (開発環境セットアップ — TBD)

TBD — 以下の内容を後から追記予定：
- ローカル開発環境の構築手順
- DynamoDB Local のセットアップ
- AWS SAM / LocalStack での Lambda ローカル実行
- `.env.local` の必要な環境変数一覧
- Bedrock のローカル代替（モックサーバー or テスト用Claudeエンドポイント）


---


## Accessibility and Localization (アクセシビリティ・多言語対応 — TBD)

TBD — 以下の内容を後から追記予定：
- WCAG 2.1 AA 準拠の目標
- 外国籍保護者への対応方針（日本語のみ or 多言語化）
- 音声読み上げ対応の範囲

