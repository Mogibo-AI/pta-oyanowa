# コンポーネントメソッド定義 — 全自動PTA - おやのわ

**ドキュメント種別**: AI-DLC Application Design - Component Methods 成果物
**作成日**: 2026-05-09
**注**: 詳細なビジネスロジックは Functional Design（per-unit, CONSTRUCTION phase）で定義。本ドキュメントは **メソッドシグネチャと I/O 型** のみ。

---

## 共通型定義（参照用）

```typescript
// 主要なドメイン型（packages/shared-types で実装）
type SchoolId = string; // ULID
type UserId = string;
type PostId = string;
type SurveyId = string;
type EventProposalId = string;

type UserType = "PARENT" | "ADMIN";
type PostCategory = "SAFETY" | "EVENT" | "CONTACT" | "REQUEST" | "OTHER";
type PostStatus = "PENDING" | "IN_PROGRESS" | "RESOLVED";
type EventStatus = "PROPOSED" | "VOTING" | "CANDIDATE" | "CONFIRMED" | "COMPLETED" | "CANCELLED";
type VoteOption = "AGREE" | "DISAGREE" | "NEUTRAL";
type CheckInMethod = "QR" | "TAP";
type NextAction = "EVENT_PLAN" | "URGENT_TOPIC" | "NO_ACTION";
type Locale = "ja" | "en";

interface AuthContext {
  schoolId: SchoolId;
  userId: UserId;
  userType: UserType;
}
```

---

## C-BE-AUTH: 認証サービス

```typescript
// POST /auth/signup
signup(input: { email, password, schoolCode, displayName }): Promise<{ userId, requiresEmailVerification: true }>

// POST /auth/verify-school-code（事前検証用）
verifySchoolCode(input: { schoolCode }): Promise<{ valid: boolean, schoolId?: SchoolId, schoolName?: string }>

// POST /auth/refresh-token
refreshToken(input: { refreshToken }): Promise<{ accessToken, idToken, expiresIn }>

// DELETE /auth/me
deleteMyAccount(ctx: AuthContext): Promise<{ deletedAt: string, postsAnonymized: number }>
// 内部処理: Cognito 物理削除 + DynamoDB 投稿の userId を "RETIRED" に置換（B-3 A）

// POST /auth/admin/sign-in（管理者専用、MFA フロー）
adminSignIn(input: { email, password, totpCode? }): Promise<{ accessToken, idToken, requiresMfa? }>
```

---

## C-BE-SUGGEST-BOX: 目安箱サービス

```typescript
// POST /posts
createPost(ctx: AuthContext, input: { content: string, isAnonymous: boolean })
  : Promise<{ postId: PostId, status: "PENDING" }>
// 入力検証: content max 500 文字 (Zod, SECURITY-05)

// GET /posts?category=&sort=&limit=&cursor=
listPosts(ctx: AuthContext, query: { category?: PostCategory, sort: "newest" | "supports", limit: number, cursor?: string })
  : Promise<{ items: Post[], nextCursor?: string }>
// 認可: schoolId フィルタ強制（IDOR防止, SECURITY-08）

// GET /posts/:id
getPost(ctx: AuthContext, postId: PostId): Promise<Post>

// POST /posts/:id/support
supportPost(ctx: AuthContext, postId: PostId): Promise<{ supportCount: number, alreadySupported: boolean }>
// 重複防止: DynamoDB ConditionExpression

// PATCH /posts/:id/status (管理者のみ)
updatePostStatus(ctx: AuthContext, postId: PostId, status: PostStatus): Promise<Post>
// 認可: ctx.userType === "ADMIN" 必須
// 副作用: 公開操作ログ記録（status=RESOLVED に遷移時）
```

---

## C-BE-SURVEY: アンケートサービス

```typescript
// GET /surveys
listSurveys(ctx: AuthContext, query: { status?: "ACTIVE" | "CLOSED", limit, cursor })
  : Promise<{ items: Survey[], nextCursor? }>

// GET /surveys/:id
getSurvey(ctx: AuthContext, surveyId: SurveyId): Promise<Survey>

// POST /surveys/:id/answers
submitAnswers(ctx: AuthContext, surveyId: SurveyId, input: { answers: Answer[] })
  : Promise<{ submittedAt: string }>
// 重複回答防止: DynamoDB ConditionExpression

// GET /surveys/:id/results
getSurveyResults(ctx: AuthContext, surveyId: SurveyId)
  : Promise<{ aggregations: AggregationByQuestion[], myAnswer?: Answer[] }>

// GET /surveys/:id/analysis（管理者のみ）
getSurveyAnalysis(ctx: AuthContext, surveyId: SurveyId)
  : Promise<{ nextAction: NextAction, reason: string, urgencyScore: number }>
```

---

## C-BE-NOTIFICATION: 通知サービス

```typescript
// POST /notifications/broadcast (管理者のみ)
broadcastNotification(ctx: AuthContext, input: { title: string, body: string })
  : Promise<{ notificationId: string, deliveryCount: number }>
// SECURITY-05 入力バリデーション（タイトル100文字、本文500文字）

// 内部 API（Step Functions / Stream Processor から呼び出し）
sendSurveyDeliveryNotification(schoolId: SchoolId, surveyId: SurveyId): Promise<{ deliveryCount: number }>
sendEventConfirmedNotification(schoolId: SchoolId, eventId: string): Promise<{ deliveryCount: number }>
// B-8 C: 全保護者に配信

// デバイストークン管理
registerDeviceToken(ctx: AuthContext, input: { deviceToken: string, platform: "ANDROID" }): Promise<void>
unregisterDeviceToken(ctx: AuthContext): Promise<void>
```

---

## C-BE-REPORT: 月次レポートサービス

```typescript
// GET /reports
listReports(ctx: AuthContext, query: { limit, cursor })
  : Promise<{ items: ReportSummary[], nextCursor? }>

// GET /reports/:id
getReport(ctx: AuthContext, reportId: string): Promise<Report>
// Report = { id, month, contentMarkdown, generatedAt, isPublished }

// （管理者専用）GET /admin/reports/:id/preview
getReportPreview(ctx: AuthContext, reportId: string): Promise<Report>
```

---

## C-BE-ADMIN: 管理者操作サービス

```typescript
// POST /admin/school-codes
createSchoolCode(ctx: AuthContext, input: { validUntil: string, maxUses: number })
  : Promise<{ schoolCode: string, qrCodeBase64: string, validUntil: string }>
// B-1 A: 8文字英数字 (大文字+数字、0/O/I/l 除外)、デフォルト90日

// PATCH /admin/school-codes/:id
invalidateSchoolCode(ctx: AuthContext, codeId: string): Promise<{ invalidatedAt: string }>

// GET /admin/parents
listParents(ctx: AuthContext, query: { limit, cursor })
  : Promise<{ items: ParentSummary[], nextCursor? }>
// 表示項目最小化: displayName, registeredAt, lastLoginAt のみ

// DELETE /admin/parents/:id（管理者による削除）
deleteParent(ctx: AuthContext, userId: UserId): Promise<{ deletedAt: string }>
// 副作用: 公開操作ログ記録（破壊的操作）

// GET /admin/post-trends?period=week|month
getPostTrends(ctx: AuthContext, query: { period: "week" | "month" })
  : Promise<{ byCategoryWeekly: TrendData[] }>

// GET /public-operation-log?limit=&cursor=
listPublicOperationLog(ctx: AuthContext, query)
  : Promise<{ items: OperationLogEntry[], nextCursor? }>
// 全保護者・管理者から閲覧可能
```

---

## C-BE-EVENT: イベントサービス

```typescript
// === F-08 イベント企画案 ===

// GET /events/proposals
listEventProposals(ctx: AuthContext, query: { status?: EventStatus, limit, cursor })
  : Promise<{ items: EventProposal[], nextCursor? }>

// GET /events/proposals/:id
getEventProposal(ctx: AuthContext, proposalId: EventProposalId): Promise<EventProposal>

// POST /events/proposals/:id/vote
voteEventProposal(ctx: AuthContext, proposalId: EventProposalId, input: { vote: VoteOption })
  : Promise<{ tally: { agree: number, disagree: number, neutral: number }, isCandidate: boolean }>
// B-5 A: 賛成 ≥ (反対 + 興味なし) かつ 投票率 ≥ 20% で isCandidate = true

// === F-08 + F-10 実施確定とチェックイン ===

// POST /events/:id/confirm （管理者のみ）
confirmEvent(ctx: AuthContext, eventId: string, input: {
  scheduledAt: string,
  checkInWindowStart: string,
  checkInWindowEnd: string,
  checkInMethod: CheckInMethod  // B-7 X: 管理者選択、デフォルト QR
})
  : Promise<{ event: Event, qrToken?: string }>
// 副作用: イベントステータス CANDIDATE → CONFIRMED、Stream Processor 経由で全保護者に通知（B-8 C）

// POST /events/:id/check-in
checkInEvent(ctx: AuthContext, eventId: string, input: { qrToken?: string })
  : Promise<{ checkedInAt: string }>
// 認可: 受付期間内、重複防止
// QR モードの場合: qrToken の有効性検証（時刻署名）

// GET /events/:id/check-ins （管理者のみ）
listEventCheckIns(ctx: AuthContext, eventId: string)
  : Promise<{ totalCount: number, items: CheckInEntry[] }>
```

---

## C-AUTHZ: Lambda Authorizer

```typescript
// Lambda Authorizer (REQUEST 型)
authorize(event: APIGatewayRequestAuthorizerEvent): Promise<APIGatewayAuthorizerResult>
// 内部処理:
// 1. Authorization ヘッダーから Bearer トークン抽出
// 2. Cognito JWKS から公開鍵取得（キャッシュ済み）
// 3. JWT 署名検証 + exp/aud/iss クレーム確認 (SECURITY-12)
// 4. クレームから schoolId, userId, userType 抽出
// 5. リクエストパスから schoolId 該当部分を抽出 → クレームと一致確認 (SECURITY-08 IDOR防止)
// 6. DynamoDB クエリで学校アクティブ確認（status=ACTIVE）
// 7. IAM Policy + コンテキスト返却（5分キャッシュ）

// 補助メソッド
extractSchoolIdFromPath(path: string): SchoolId | null
isSchoolActive(schoolId: SchoolId): Promise<boolean>
```

---

## C-AI-SUB-CLASSIFY: Post Classification Sub-Agent

```typescript
classifyPost(input: { schoolId: SchoolId, postId: PostId, content: string })
  : Promise<{ category: PostCategory, summary: string }>
// モデル: anthropic.claude-haiku-4-5-20251001
// 副作用: DynamoDB 該当投稿の category/summary 更新
// SECURITY-15: Bedrock 失敗時は category="OTHER"、Slack 通知、SQS DLQ
```

---

## C-AI-SUB-DECISION: Survey Decision Sub-Agent

```typescript
decideSurveyDelivery(input: {
  schoolId: SchoolId,
  recentPosts: Post[],
  lastDeliveredAt?: string,
  schoolMode: "ai" | "threshold",
  thresholds?: { minPostCount: number, minDaysSinceLastDelivery: number }
})
  : Promise<{ shouldDeliver: boolean, reason: string }>
// B-2 X: デフォルト AI判断、管理者がしきい値モード切替可能
// AI モードの場合: Sonnet にコンテキスト渡して判断
// しきい値モードの場合: ルールエンジンで判定
```

---

## C-AI-SUB-GENERATE: Survey Generation Sub-Agent

```typescript
generateSurvey(input: {
  schoolId: SchoolId,
  recentPosts: Post[],
  pastSurveyTrends: SurveyTrend[],
  seasonContext: { month: number, isHoliday: boolean }
})
  : Promise<{ questions: SurveyQuestion[], estimatedAnswerTime: number }>
// 5〜7問生成、各 question = { id, text, type: "single_choice" | "multi_choice" | "free_text", options? }
```

---

## C-AI-SUB-ANALYZE: Survey Result Analysis Sub-Agent

```typescript
analyzeSurveyResults(input: {
  schoolId: SchoolId,
  surveyId: SurveyId,
  aggregatedAnswers: SurveyAggregation,
  recentPosts: Post[]
})
  : Promise<{
    nextAction: NextAction,
    reason: string,
    urgencyScore: number,
    eventPlanContext?: { keywords: string[], timeframe: string }
  }>
// B-6 X: ルールベース判定 + AI による緊急性ブースト
// 1. ルールベース: 要望集中・季節性 → EVENT_PLAN / 緊急意見一定数 → URGENT_TOPIC
// 2. AI ブースト: ルール判定が NO_ACTION でも、1件でも明確に緊急性が高いものがあれば URGENT_TOPIC へ昇格
```

---

## C-AI-SUB-EVENT: Event Planning Sub-Agent

```typescript
planEvent(input: {
  schoolId: SchoolId,
  triggerSource: { surveyId?: SurveyId, postIds?: PostId[] },
  analysisContext: { keywords: string[], timeframe: string },
  schoolInfo: { name: string, season: string }
})
  : Promise<{
    title: string,
    description: string,
    expectedAttendees: number,
    suggestedTimeframe: string,  // 例: "2026年6月の土曜または日曜"
    sourcePosts: PostId[]  // 参考にした投稿の追跡用
  }>
```

---

## C-AI-SUB-REPORT: Monthly Report Sub-Agent

```typescript
generateMonthlyReport(input: {
  schoolId: SchoolId,
  month: string,  // "2026-04"
  activityData: {
    postCount: number,
    postsByCategory: Record<PostCategory, number>,
    surveysDelivered: number,
    surveyAvgResponseRate: number,
    eventsConfirmed: EventSummary[],
    checkIns: { eventId: string, attendeeCount: number }[]
  },
  budgetData: BudgetSnapshot  // MVP では暫定構造
})
  : Promise<{
    contentMarkdown: string,
    metadata: { generatedAt: string, modelUsed: string }
  }>
```

---

## C-AI-LIB: Bedrock Common Library

```typescript
// Bedrock 呼び出しラッパー
export class BedrockClient {
  constructor(config: { region: string, defaultModel: ModelId, guardrailsId?: string });

  async invokeModel<T>(input: {
    modelId: ModelId,
    prompt: string,
    maxTokens: number,
    temperature?: number,
    enforceGuardrails?: boolean  // B-10 A: デフォルト true
  }): Promise<T>;

  // SECURITY-15 例外処理 + リトライ
  private withRetry<T>(fn: () => Promise<T>, maxRetries: number): Promise<T>;
}

// プロンプト管理
export const PROMPTS = {
  postClassification: () => loadPrompt('post-classification'),
  surveyDecision: (mode: 'ai') => loadPrompt('survey-decision-ai'),
  surveyGeneration: () => loadPrompt('survey-generation'),
  surveyResultAnalysis: () => loadPrompt('survey-result-analysis'),
  eventPlanning: () => loadPrompt('event-planning'),
  monthlyReport: () => loadPrompt('monthly-report'),
};

// ゴールデンテスト基盤（CIで実行、AI-PROMPT-02）
export class GoldenTestRunner {
  async runAll(): Promise<TestResult[]>;
}

// Guardrails 設定（B-10 A）
export const GUARDRAILS_CONFIG = {
  deniedTopics: ['POLITICS', 'RELIGION', 'VIOLENCE', 'SEXUAL'],
  piiFilters: ['EMAIL', 'PHONE', 'ADDRESS'],
  promptInjectionDetection: 'STANDARD'
};
```

---

## C-STREAM-POST: Post Classification Trigger

```typescript
// DynamoDB Streams トリガー Lambda
handle(event: DynamoDBStreamEvent): Promise<{ batchItemFailures: any[] }>
// 処理:
// 1. INSERT イベントのみフィルタ（PK = SCHOOL#xxx, SK = POST#xxx）
// 2. SQS にメッセージ送信（A-4 非同期）
// 3. 失敗したレコードは batchItemFailures で部分リトライ
```

---

## C-STREAM-EVENT-CONFIRM: Event Confirmation Notifier

```typescript
handle(event: DynamoDBStreamEvent): Promise<{ batchItemFailures: any[] }>
// 処理:
// 1. UPDATE イベントで status: CANDIDATE → CONFIRMED に遷移したものをフィルタ
// 2. C-BE-NOTIFICATION.sendEventConfirmedNotification を呼び出し
// 3. 全保護者にプッシュ通知（B-8 C）
```

---

## C-PKG-API-CLIENT: API Client (Frontend Hooks)

```typescript
// TanStack Query 用 hooks 例
export function useSuggestBoxPosts(filters: { category?: PostCategory });
export function usePostMutation();
export function useSupportMutation();

export function useSurveys();
export function useSurveyById(surveyId: SurveyId);
export function useSubmitAnswers();

export function useEventProposals();
export function useVoteMutation();
export function useCheckInMutation();

export function useReports();

// Auth hooks（C-PKG-AUTH と連携）
export function useCurrentUser();
export function useSignIn();
export function useSignUp();
export function useSignOut();
export function useDeleteAccount();
```

---

## C-PKG-AUTH: Auth Helpers

```typescript
// Cognito クライアントラッパー
export class AuthClient {
  signUp(input: { email, password, schoolCode, displayName }): Promise<{ userId }>;
  signIn(input: { email, password }): Promise<{ tokens: TokenSet }>;
  signOut(): Promise<void>;
  refreshToken(refreshToken: string): Promise<TokenSet>;
  getCurrentUser(): Promise<User | null>;
  setupMfa(): Promise<{ qrCodeUrl: string, recoveryCode: string }>;  // 管理者用
  verifyMfa(code: string): Promise<TokenSet>;
}

// プラットフォーム別永続化
export interface TokenStorage {
  save(tokens: TokenSet): Promise<void>;
  load(): Promise<TokenSet | null>;
  clear(): Promise<void>;
}
// Web: LocalStorageTokenStorage, RN: SecureStoreTokenStorage
```

---

## 設計規約（全コンポーネント共通）

1. **エラーハンドリング**: 全 Lambda の handler は `withErrorHandler()` ラッパーを通す（C-AI-LIB に共通実装）。SECURITY-15 fail-closed。
2. **ログ**: `createLogger({ schoolId, requestId })` を必ず使用、PII 出力禁止（SECURITY-03）。
3. **入力バリデーション**: 全 API エンドポイントで Zod スキーマ検証（SECURITY-05）。スキーマは C-PKG-TYPES からインポート。
4. **認可**: 全 handler 内で `ctx.userType` 確認、`schoolId` 一致確認（SECURITY-08）。
5. **DynamoDB 操作**: 必ず `schoolId` を PK の一部として含める（IDOR防止）。
6. **国際化**: ユーザー向けエラーメッセージは i18n キーで返す（C-PKG-I18N）。
