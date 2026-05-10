# 要件定義書 — 全自動PTA - おやのわ (oyano-wa)

**ドキュメント種別**: AI-DLC Requirements Analysis 成果物（Comprehensive Depth）
**プロジェクト種別**: Greenfield SaaS 新規開発
**作成日**: 2026-05-09
**インプット元**:
- `writing-inputs/vision-document.md`
- `writing-inputs/technical-environment.md`
- `aidlc-docs/inception/requirements/requirement-verification-questions.md`
- `aidlc-docs/inception/requirements/requirement-clarification-questions.md`

---

## 1. インテント分析サマリー

| 項目 | 値 |
|---|---|
| **ユーザー要求** | 「全自動PTA - おやのわ」のMVPを最速でリリース可能な状態に構築する。AIがPTA活動を代行し、保護者は「気が向いたとき意見投稿するだけ」で参加できる仕組みを実現する |
| **リクエスト種別** | New Project（Greenfield 新規SaaS開発） |
| **スコープ** | Cross-system（Web管理者画面・RNモバイルアプリ・Lambdaバックエンド・CDKインフラ・Bedrock AI連携） |
| **複雑性** | Complex（マルチテナントSaaS・AIエージェント多数・段階的リリース）|
| **要件深度** | **Comprehensive**（複雑×高リスク領域=会費・個人情報・学校データ） |
| **本イテレーションのターゲット** | **MVPスコープのみ**（質問1: A） |

---

## 2. プロジェクトコンテキスト

### 2.1 プロジェクトの本質ゴール
PTAを廃止するのではなく、「**PTAから人間がやるべき仕事を限界まで除去する**」。
AIが企画・調整・連絡・記録・報告を担い、人間の判断は「賛否投票」と「タスクの受諾/辞退」のみとする。

### 2.2 ビジネスモデル
- 学校単位の SaaS 提供
- 保護者は無料、学校または自治体が月額費用を負担
- 1学期間（4ヶ月）の無料トライアル → スタンダード/プレミアム/自治体一括

### 2.3 成功指標
| 指標 | 目標値 |
|---|---|
| アンケート回答率 | 平均70%以上 |
| 目安箱投稿数（月次） | 1校あたり平均20件以上 |
| イベント実施数（年次） | 1校あたり平均3件以上 |
| 保護者満足度 (NPS) | +40以上 |
| タスク受諾率 | 85%以上 |

---

## 3. ステークホルダー

| 種別 | 主な特徴 | MVPでの主要操作 |
|---|---|---|
| 保護者（一般） | 共働き・時間がない・役員はやりたくない | アプリで目安箱投稿・アンケート回答・月次レポート閲覧 |
| 保護者（意欲的） | 提案したい・意見を言いたい | 目安箱で提案投稿・他人の投稿への支持表明 |
| 学校管理者（校長・教頭） | PTAとの調整が手間 | 管理画面で全体把握・学校コード発行・即時通知配信 |
| 学校の先生（担当） | 保護者意見の窓口役 | 管理画面でメール通知の受信・確認 |
| **新入学保護者** | アプリ初期化導線 | QRコード→インストール→学校コード入力→初回ログイン |

外国籍保護者（非日本語話者）は MVP の英語UI対応の主対象。

---

## 4. MVP スコープ確定

### 4.1 MVPに含む機能（IN）

| 機能 | 内容 | 出典 |
|---|---|---|
| **F-01: デジタル目安箱（基本）** | テキスト投稿・一覧表示・カテゴリ分類（AI）・支持数表示 | ビジョン MVP / 質問1 A |
| **F-02: AIアンケート自動生成** | 質問生成（AI）・全員配信・回答グラフ表示 | ビジョン MVP / 質問1 A |
| **F-03: プッシュ通知（AIアンケート配信のみ）** | EventBridge 9時発火 + AI判断 + SNS配信 | 質問13 X / 明確化2-2 A |
| **F-04: 月次レポート自動生成** | 活動内容・予算の自動公開（AI生成、アプリ内表示） | ビジョン MVP / 質問1 A |
| **F-05: 保護者アカウント認証** | Cognito メール認証 + 学校コード（招待方式） | ビジョン MVP / 質問12 A |
| **F-06: 基本管理者画面** | 学校コード発行・保護者管理・即時通知配信ボタン | ビジョン MVP / 明確化2-3 A |
| **F-07: 多言語UI（日本語+英語・最低限）** | 外国籍保護者が操作できる範囲のUI（メニュー・主要操作） | 質問16 C / 明確化3 X |
| **F-08: イベント企画AI（検証用）** | 目安箱・アンケート結果から AI が企画案を生成・全保護者への賛否投票機能（タスク割り当て・実施フローは Phase 2） | 追加要望2 |
| **F-09: 将来機能ナビゲーション（Coming Soon）** | Topページに将来機能の遷移ボタンを「Coming Soon」バッジ付きで配置し、システムの拡張性をユーザーにアピール | 追加要望1 |
| **F-10: 参加者チェックイン（イベント当日）** | 「実施確定」イベントの当日に保護者が QR コードまたはタップでチェックイン、管理者画面で参加者数を確認 | 追加要望4 |
| **F-11: Android アプリ配信（.apk ビルド）** | React Native で実装した保護者向けアプリを Android 用 .apk としてビルドし、Google Play への配信準備までを MVP に含む（iOS は Phase 3） | 追加要望5 |

### 4.2 MVPから除外する機能（OUT、Phase 2 以降）

| 機能 | 除外理由 | 対象フェーズ |
|---|---|---|
| ~~イベント企画AI（機能4）~~ | **MVP に追加（F-08）** — 企画案生成と賛否投票のみ。実施フローは Phase 2 | ~~Phase 2~~ → MVP |
| イベント実施フロー（賛否可決後の自動進行） | 企画AIの結果が出てから検討、検証目的のため最初は手動進行 | Phase 2 |
| タスク自動割り当て（機能3） | 企画AIの実用性検証後に判断 | Phase 2 |
| 前日AIマネジメント | タスク機能が前提 | Phase 2-3 |
| カレンダー表示（機能7） | 学校カレンダー連携APIが必要 | Phase 2 |
| 会費徴収機能（機能8） | 決済インフラが別途必要 | Phase 2 |
| 学校管理者画面（高度設定） | 基本管理は含む | Phase 2 |
| 過去情報ダッシュボード（機能6） | データ蓄積が必要 + イベント当日機能（F-10系列拡張）と連携 | Phase 3 |
| 自治体一括管理 / 複数校管理 | 営業・契約フロー要件 | Phase 3 |
| **イベント当日: リアルタイム進行管理（タイムライン表示）** | イベント進行状況をアプリでリアルタイム表示、各タスクの進捗を可視化 | **Phase 3**（追加要望4） |
| **イベント当日: 写真共有機能** | 参加者が当日撮影した写真をアプリ内で共有、思い出の集約 | **Phase 3**（追加要望4） |
| **イベント当日: 問題発生時のヘルプAI** | 当日トラブル発生時にAIチャットがすぐサポート、よくある質問への自動回答 | **Phase 3**（追加要望4） |
| **イベント終了時: リアルタイム参加者アンケート** | イベント終了直後に参加者へ満足度アンケートを配信、その場で回収・集計 | **Phase 3**（追加要望4） |
| 画像投稿 | テキストで十分 | Phase 2 |
| Web保護者向け（ブラウザ版） | iPhone 保護者・PC 利用希望者向けの暫定アクセス手段にもなる / アクセシビリティ向上 | **Phase 2**（追加要望5）|
| **iOS アプリリリース（App Store）** | RN で実装済みのため iOS でも動作するが、リリースは TestFlight + App Store 申請を含めて Phase 3 で対応 | **Phase 3**（追加要望5）|
| 既読管理・既読率表示 | 優先度低 | Phase 3 |
| 議事録・会議サポート（機能10） | 音声インフラ別途必要 | Phase 2 |
| アンケート意見主張トグル（機能11） | 基本機能安定後 | Phase 2 |
| **目安箱の類似投稿グルーピング** | カテゴリ分類だけでもMVP体験は成立 | Phase 2 |
| **アンケート回答リアルタイム集計AI** | DynamoDB集計のみで対応可、AI不要 | Phase 2 |
| **手動アンケート生成（管理者）** | 自動生成のみで MVP 成立 | Phase 2 |
| **月次レポート配信のプッシュ通知** | アプリ内表示のみ。明確化2-2 A により MVPプッシュ通知はAIアンケート配信のみ | Phase 2 |

---

## 5. 機能要件（Functional Requirements）

### F-01: デジタル目安箱

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-01.1 | 保護者はテキスト投稿（最大500文字）を送信できる | Zod バリデーション + Lambda で受付・ステータス保存 |
| F-01.2 | 保護者は匿名/実名を投稿時に選択できる | 匿名時は userId を保存しない。一覧でも匿名表示 |
| F-01.3 | 投稿は AI（Haiku）により以下に自動分類される | カテゴリ: SAFETY / EVENT / CONTACT / REQUEST / OTHER |
| F-01.4 | 投稿一覧はカテゴリ・新着順・支持数順でフィルタ・ソート可能 | API クエリパラメタで制御 |
| F-01.5 | 他保護者は投稿に「支持」を1回まで表明できる | 同一ユーザーの重複支持を DynamoDB で防止 |
| F-01.6 | 対応状況（PENDING/IN_PROGRESS/RESOLVED）が公開管理される | 管理者が状態遷移、保護者は閲覧 |

**AI起動トリガー**: F-01.1 投稿時 → DynamoDB Streams → Lambda → Bedrock Haiku 呼び出し（投稿分類エージェント）

### F-02: AIアンケート自動生成

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-02.1 | 配信判断AIが配信トリガー時に「配信可否」を判断する | 投稿状況 + 前回配信からの経過時間 + イベント情報を総合判断（明確化2-1 D） |
| F-02.2 | 配信可と判断された場合、AI（Sonnet）が5〜7問の質問を自動生成 | 過去意見傾向・季節性・学校行事を考慮 |
| F-02.3 | 配信不要と判断された場合、その日は配信されない | 「配信予約スキップ」を CloudWatch ログに記録 |
| F-02.4 | 全保護者にプッシュ通知でアンケートが配信される | Amazon SNS → FCM/APNs |
| F-02.5 | 保護者は1〜2タップでアンケートに回答できる | フォーム UI（shadcn/ui） |
| F-02.6 | 回答結果はリアルタイム集計（DynamoDB の UpdateItem による加算）で集計され、グラフ表示される | Recharts によるバーチャート/円グラフ |

**AI起動トリガー**:
- F-02.1: 毎日9時 EventBridge → Lambda → 配信判断エージェント（Sonnet）
- F-02.2: 配信判断の結果、生成 → アンケート生成エージェント（Sonnet）

### F-03: プッシュ通知

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-03.1 | MVP では「AIアンケート配信通知」のみがプッシュ通知対象 | 明確化2-2 A |
| F-03.2 | EventBridge による定期発火（毎日9時 JST）+ 管理者の即時配信ボタンの2系統 | 明確化2-3 A |
| F-03.3 | 保護者は通知の受信を有効化/無効化できる（OS の通知設定で完結） | アプリ内設定は MVP では不要 |

### F-04: 月次レポート自動生成

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-04.1 | 月初（毎月1日）に AI（Sonnet）が前月の活動内容・予算消化を要約してレポート生成 | EventBridge トリガー |
| F-04.2 | 生成されたレポートはアプリ内（保護者・管理者の両方）で閲覧可能 | DynamoDB 保存 + S3 PDF 化（任意）|
| F-04.3 | MVP ではレポート配信のプッシュ通知は行わない | 明確化2-2 A の通り |

**AI起動トリガー**: 月初 EventBridge → Lambda → 月次レポート生成エージェント（Sonnet）

### F-05: 保護者アカウント認証

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-05.1 | 学校管理者が Cognito ユーザープールに学校コード（招待コード）を発行 | 管理画面から学校コード生成・有効期限設定 |
| F-05.2 | 保護者はメール+パスワード+学校コードで初回登録 | Cognito SignUp + Custom Attribute (`schoolId`) |
| F-05.3 | ログイン後、トークンに `schoolId` クレームを含め API 認可で利用 | Lambda Authorizer で検証 |
| F-05.4 | 保護者は退会できる。退会時にアカウント削除、過去投稿は「退会済みユーザー」として匿名化保持 | 質問18 A |
| F-05.5 | 保護者プールと管理者プールは Cognito で分離 | テクニカル環境ドキュメント準拠 |

### F-06: 基本管理者画面

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-06.1 | 管理者は Cognito（メール認証 + TOTP MFA 必須）でログイン | テクニカル環境ドキュメント準拠 |
| F-06.2 | 学校コード発行・無効化・有効期限管理ができる | shadcn/ui ベースの管理画面 |
| F-06.3 | 保護者一覧の閲覧（個人情報は最小限のみ表示） | 必要列のみ：表示名・登録日・最終ログイン |
| F-06.4 | 即時通知配信ボタンで管理者からのお知らせを SNS 配信できる | 明確化2-3 A |
| F-06.5 | 破壊的操作（保護者削除・投稿削除）には「保護者が見れる操作ログに残ります」警告ダイアログを表示 | ビジョン記載 |

### F-07: 多言語UI（日英）

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-07.1 | 管理者画面・保護者モバイルアプリのメニュー・ボタン・主要操作のUIテキストは日本語と英語の両方を提供 | i18n 基盤（next-i18next / react-i18next）導入 |
| F-07.2 | AI生成テキスト（投稿要約・月次レポート）はMVPでは日本語のみ | 明確化3 X |
| F-07.3 | 主対象は外国籍保護者（学校在籍の非日本語話者） | 明確化3 補足 A |
| F-07.4 | 言語切り替えは OS 言語/ブラウザ言語の自動検出 + 手動切替設定 | localStorage / AsyncStorage 永続化 |

### F-08: イベント企画AI（検証用）

**MVP の目的**: 「イベント企画を AI に任せる」というアプローチが実用的かをユーザーフィードバックで検証する。
**MVP範囲外**: タスク自動割り当て・実施フローへの自動移行・前日AIマネジメント・カレンダー連携（学校行事カレンダー統合は Phase 2）。

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-08.1 | アンケート結果分析（F-02.6 の集計データ）と直近の目安箱投稿（F-01）を入力に、AI（Sonnet）がイベント企画案（タイトル・概要・想定参加者数・実施候補時期）を生成する | 実施候補日は「来月の土曜/日曜」など粗い時期帯のみ MVP で扱う（具体日付の確定は Phase 2 で学校カレンダー連携後） |
| F-08.2 | 生成された企画案は「企画案一覧」画面に表示され、全保護者がアプリで閲覧できる | shadcn/ui ベースのカード一覧 |
| F-08.3 | 各企画案に対して全保護者が「賛成 / 反対 / 興味なし」の3択で投票できる | DynamoDB で集計、1ユーザー1票 |
| F-08.4 | 投票結果は企画案カード上にリアルタイム表示される（賛成数 / 反対数 / 興味なし数） | Recharts または進捗バー |
| F-08.5 | 賛成多数（例: 賛成 ≥ 反対 + 興味なし）と判定された企画案は「実施候補」マークが付く | Lambda での判定ロジック、しきい値は設定可能（OQ-12 参照） |
| F-08.6 | **MVP範囲外**: 実施候補マークが付いた後の自動タスク割り当て・スケジュール確定・参加者通知などは行わず、管理者の手動判断に委ねる | Phase 2 で自動化検討 |
| F-08.7 | 企画AIの起動トリガーは「アンケート結果分析AI（後述 F-02.7）」からの連鎖起動を基本とする | AI-ARCH 経由 |

### F-11: Android アプリ配信（.apk ビルド）

**MVP の目的**: 保護者向けアプリを Android で利用可能な状態にする。iPhone 保護者は Phase 2 の Web 版または Phase 3 の iOS リリース（App Store）を待つ運用とする。

**MVP 範囲外**: iOS 版の TestFlight 配信・App Store 申請（Phase 3）。Google Play 本番リリース手続き（営業・契約フロー、Phase 1 後半〜2 で対応）。

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-11.1 | React Native 保護者アプリは **Android 6.0（API level 23）以上** をターゲットとする | minSdkVersion の明示 |
| F-11.2 | dev 環境向けの **デバッグ用 .apk** が CI（CodeBuild）で自動ビルドされる | アーティファクトとして S3 に保存 |
| F-11.3 | リリース用 .apk のビルド手順（署名・難読化・ProGuard 等）を `apps/app-parent/README.md` に文書化する | 文書化必須 |
| F-11.4 | iOS ビルドは MVP では行わない（コード対応はするが、ビルド・配信は Phase 3）| iOS 用ビルド設定は基本構成のみ含めて将来の Phase 3 で活性化可能とする |
| F-11.5 | Google Play への配信に必要なメタデータ（アプリ名・説明・スクリーンショット・プライバシーポリシーURL）を準備する | dev 環境動作確認後の MVP 完了タイミングで揃える |
| F-11.6 | iPhone 保護者向けには「Web 版（Phase 2）または iOS 版（Phase 3）の準備中」というアナウンスを公式サイト等に掲示する想定 | アプリ外コミュニケーション、要件としては明示のみ |

**Phase 2 連動**: Web 版（Next.js）リリースで iPhone・PC ユーザーがアクセス可能になる
**Phase 3 連動**: TestFlight + App Store 申請で iOS ユーザー向け正式リリース

---

### F-10: 参加者チェックイン（イベント当日）

**MVP の目的**: F-08 で実施候補と確定したイベントの当日参加者を把握し、参加実績データを蓄積する（F-08 検証の補完データ + Phase 3 過去情報ダッシュボードの基盤データ）。

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-10.1 | 学校管理者は「実施候補」マークの付いた企画案を「実施確定」状態に手動遷移できる | 管理者画面に「実施確定」ボタン、確定後はチェックイン受付が可能になる |
| F-10.2 | 実施確定済みイベントには「チェックイン受付期間」を設定する（イベント当日の前後数時間） | 開始時刻と終了時刻を管理者が指定 |
| F-10.3 | 保護者は受付期間中、アプリでチェックインできる | 方式は QRコード読取 or タップ式の2択（OQ-15 で確定） |
| F-10.4 | 1ユーザー1イベントで1回のみチェックイン可能（重複防止） | DynamoDB での重複登録防止 |
| F-10.5 | 管理者画面で当該イベントのチェックイン状況をリアルタイム参照できる | 参加者数・参加者リスト（表示名のみ）|
| F-10.6 | チェックインデータは Phase 3 の過去情報ダッシュボード基盤データとして蓄積される | DynamoDB の永続保存 |
| F-10.7 | チェックイン時刻・実行者は公開操作ログには載らない（参加履歴は個人情報扱い）| プライバシー保護 |

**Phase 3 拡張予定**: 当日のリアルタイム進行管理・写真共有・ヘルプAI・終了時リアルタイムアンケート（§4.2 参照）

---

### F-09: 将来機能ナビゲーション（Coming Soon UI）

**目的**: Topページに将来機能の遷移ボタンを「Coming Soon」バッジ付きで表示し、システムの将来性・拡張性をユーザー（学校管理者・保護者）にアピールする。

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-09.1 | 保護者モバイルアプリの Topページに、以下5つの「Coming Soon」ナビゲーション項目を表示する | 既存の MVP メニュー（目安箱・アンケート・月次レポート・イベント企画案）と並列に表示 |
| F-09.2 | Coming Soon 対象（MVP 時点）: 📅 カレンダー / 💰 会費・予算 / ✅ タスク管理 / 📝 議事録 / 📊 過去情報ダッシュボード | ビジョンドキュメントの機能エリア6, 7, 8, 10 + Phase 2/3 機能 |
| F-09.3 | Coming Soon ボタンはタップしても遷移せず、「近日公開予定」のトーストまたはモーダルを表示する | shadcn/ui の Toast / Dialog コンポーネント |
| F-09.4 | Coming Soon バッジは視覚的に「現在利用可能な機能」と区別される（例: 半透明 + バッジ表記） | アクセシビリティ: スクリーンリーダー読み上げで「準備中」と認識可能 |
| F-09.5 | 管理者画面でも将来機能のナビゲーション項目を同様に Coming Soon として表示する（例: 自治体ダッシュボード・操作ログ詳細） | 管理者向けは別の Coming Soon リスト |
| F-09.6 | 多言語対応: Coming Soon バッジは日英両方で表示（「Coming Soon」/ 「近日公開」） | F-07 i18n に準拠 |

**追加機能要件 F-02.7（アンケート結果分析AI）**:

| ID | 要件 | 受け入れ基準 |
|---|---|---|
| F-02.7 | アンケートが期日を迎えた時点で、AI（Sonnet）が回答結果を分析し、次のアクション提案（イベント企画AIへの入力 or 目安箱への新トピック提示 or 「アクション不要」）を判定する | 毎日9時 EventBridge で期日到来アンケートをスキャン |
| F-02.8 | 「次のアクション = イベント企画」と判定された場合、F-08 イベント企画AI を連鎖起動する | AI-ARCH のオーケストレーター経由 |
| F-02.9 | 分析結果（次のアクション判定理由・要約）は管理者画面で閲覧可能 | 透明性確保のためログ表示 |

---

## 6. 非機能要件（Non-Functional Requirements）

### 6.1 パフォーマンス
| ID | 要件 |
|---|---|
| NFR-PERF-01 | API レスポンス: p95 < 500ms（DBクエリ系）/ p95 < 5s（AI呼び出し系） |
| NFR-PERF-02 | プッシュ通知の配信開始: トリガーから5分以内 |
| NFR-PERF-03 | アプリ起動: コールドスタート < 3秒（モバイル） |

### 6.2 スケーラビリティ
| ID | 要件 |
|---|---|
| NFR-SCAL-01 | 1校あたり最大1000世帯規模を想定。DynamoDB オンデマンドキャパシティで自動スケール |
| NFR-SCAL-02 | Lambda の同時実行数は AWSアカウントの reserved concurrency で適切に制御 |
| NFR-SCAL-03 | Bedrock 呼び出しは SQS キューイングで過負荷を防ぐ（テクニカル環境記載） |

### 6.3 信頼性・可用性
| ID | 要件 |
|---|---|
| NFR-REL-01 | サーバーレス構成による高可用性（Lambda / API Gateway / DynamoDB はマルチAZ）|
| NFR-REL-02 | 月次レポート生成失敗時は EventBridge 再試行 + Slack アラート |
| NFR-REL-03 | プッシュ通知失敗時はリトライ（最大3回） |

### 6.4 セキュリティ（**Security Baseline 拡張 = 強制適用**）

質問3 A により Security Baseline 拡張ルール（SECURITY-01 〜 SECURITY-15）を全面適用。

| SECURITY ルール | 本プロジェクトでの適用 |
|---|---|
| SECURITY-01: 暗号化 | DynamoDB / S3 / Secrets Manager すべて KMS で暗号化、TLS 1.2+ 強制 |
| SECURITY-02: ネットワーク中継ログ | API Gateway アクセスログ + CloudFront ログを CloudWatch に集約 |
| SECURITY-03: アプリログ | 構造化JSONログ + `schoolId` 必ず含める + PII を出力しない |
| SECURITY-04: HTTPセキュリティヘッダー | CSP / HSTS / X-Content-Type-Options / X-Frame-Options / Referrer-Policy |
| SECURITY-05: 入力バリデーション | 全APIエンドポイントで Zod スキーマ検証、500文字制限など |
| SECURITY-06: 最小権限 | IAM Role はリソースID指定、ワイルドカード禁止 |
| SECURITY-07: ネットワーク制限 | API Gateway は WAF 経由、Lambda は VPC 不要構成（最小限のみ VPC 内） |
| SECURITY-08: アプリ層認可 | 全エンドポイント Cognito JWT + `schoolId` 検証で IDOR 防止 |
| SECURITY-09: 設定ハードニング | デフォルト認証情報禁止、本番エラーレスポンスはスタックトレース無し |
| SECURITY-10: 依存関係管理 | pnpm-lock.yaml 必須、CI で `npm audit` 必須、SBOM 生成 |
| SECURITY-11: セキュア設計 | 認証/認可ロジックを `backend/lib/auth/` に集約、レート制限を API Gateway で設定 |
| SECURITY-12: 認証管理 | Cognito 強制 MFA（管理者）、保護者は最低8文字+breached password チェック |
| SECURITY-13: 整合性検証 | デシリアライゼーション安全（Zod）、CDN リソースに SRI |
| SECURITY-14: アラート/監視 | 認証失敗・認可違反で Slack 通知、CloudWatch Logs 90日以上保持 |
| SECURITY-15: 例外処理 | 全 Lambda にグローバルエラーハンドラ + fail-closed 設計 |

### 6.5 テスト品質（**Property-Based Testing 拡張 = Partial 適用**）

質問4 B により PBT-02, PBT-03, PBT-07, PBT-08, PBT-09 のみ enforcement、他はアドバイザリ。

| PBTルール | 本プロジェクトでの適用 |
|---|---|
| PBT-02: ラウンドトリップ | Zod スキーマの parse/stringify、DynamoDB Item の serialize/deserialize に PBT 適用 |
| PBT-03: 不変条件 | カテゴリ分類関数（必ず5カテゴリのいずれか）、支持数の単調増加など |
| PBT-07: ジェネレータ品質 | ドメイン型（投稿、ユーザー、学校コード）の専用 fast-check generator を作成 |
| PBT-08: シュリンク・再現性 | fast-check のシュリンク有効化、CI で seed をログ出力 |
| PBT-09: フレームワーク選定 | **fast-check**（TypeScript/Jest 統合） |

### 6.6 アクセシビリティ
| ID | 要件 |
|---|---|
| NFR-A11Y-01 | WCAG 2.1 AA 準拠を**目標**とする（必須ゲートにはしない、質問15 A）|
| NFR-A11Y-02 | shadcn/ui（Radix UI ベース）でデフォルトのアクセシビリティを担保 |
| NFR-A11Y-03 | キーボード操作・スクリーンリーダー対応の主要画面動作確認 |

### 6.7 監視
| ID | 要件 |
|---|---|
| NFR-MON-01 | CloudWatch Dashboard で主要メトリクス可視化（テクニカル環境記載通り） |
| NFR-MON-02 | アラート通知先は **Slack（社内開発者チャンネル）**（質問17 A） |
| NFR-MON-03 | X-Ray で Lambda → DynamoDB → Bedrock のトレース取得 |

### 6.8 コンプライアンス
| ID | 要件 |
|---|---|
| NFR-COMP-01 | ISMS 準拠を目指す（テクニカル環境記載） |
| NFR-COMP-02 | 個人情報保護法対応：学校単位データ分離、退会時データ匿名化（質問18 A） |
| NFR-COMP-03 | 児童の個人情報・成績情報をシステムに保持しない（設計制約） |

---

## 7. AIエージェント要件（質問20 / 明確化4）

### 7.1 エージェント分離方針（マルチエージェント・マネージャーパターン）

明確化4-1 B により、**1つのオーケストレーターエージェント**が状況に応じて**サブエージェント**を呼び出す構成を採用。

```
[Orchestrator Agent]
    ├─ [Post Classification Sub-Agent]      ─ 目安箱投稿の分類（Haiku）
    ├─ [Survey Decision Sub-Agent]          ─ 配信可否判断（Sonnet）
    ├─ [Survey Generation Sub-Agent]        ─ アンケート質問生成（Sonnet）
    ├─ [Survey Result Analysis Sub-Agent]   ─ アンケート結果分析・次アクション判定（Sonnet）★追加
    ├─ [Event Planning Sub-Agent]           ─ イベント企画案生成（Sonnet）★追加（MVP）
    └─ [Monthly Report Sub-Agent]           ─ 月次レポート生成（Sonnet）
```

| 要件ID | 内容 |
|---|---|
| AI-ARCH-01 | 各サブエージェントは単一責任。プロンプトは独立管理 |
| AI-ARCH-02 | Orchestrator は入力コンテキストに応じて適切な Sub-Agent を選択し呼び出す |
| AI-ARCH-03 | Sub-Agent 間の直接通信は禁止（Orchestrator 経由） |
| AI-ARCH-04 | Bedrock モデル使い分け（質問14 A）: Sonnet 4.6 = 配信判断/質問生成/レポート生成 / Haiku 4.5 = 投稿分類 |

### 7.2 プロンプト品質保証（明確化4-2 X A,B,D）

| 要件ID | 内容 |
|---|---|
| AI-PROMPT-01 | プロンプトは **コードリポジトリ内の専用ディレクトリ** で管理（例: `backend/lib/ai/prompts/`）。バージョン履歴を Git で追跡 |
| AI-PROMPT-02 | 各プロンプトに **ゴールデンテストセット**（期待される入出力ペア）を整備し、CI で検証 |
| AI-PROMPT-03 | **Bedrock Guardrails** を導入し、プロンプトインジェクション・不適切出力を防止 |
| AI-PROMPT-04 | プロンプト変更時は PR レビューでプロンプト品質を必ず確認 |

### 7.3 AI起動トリガー（MVP最小限、明確化4-3 B + 追加要望3 反映）

MVPで実装するトリガーは以下の **4つに限定**。それ以外は Phase 2 以降。

| # | トリガー | 起動エージェント・フロー | モデル |
|---|---|---|---|
| 1 | 目安箱投稿時（DynamoDB Streams） | Post Classification Sub-Agent | Haiku 4.5 |
| 2 | **毎日9時 EventBridge**（拡張・追加要望3） | **下記の3ステップを順次実行**:<br/>**①** Survey Decision Sub-Agent → 配信可なら Survey Generation Sub-Agent でアンケート配信<br/>**②** 期日到来したアンケートをスキャン → Survey Result Analysis Sub-Agent で結果分析・次アクション判定（F-02.7）<br/>**③** 「次アクション = イベント企画」と判定された場合、Event Planning Sub-Agent を連鎖起動（F-02.8 / F-08） | Sonnet 4.6 |
| 3 | 月初 EventBridge（毎月1日 0時 JST） | Monthly Report Sub-Agent | Sonnet 4.6 |
| 4 | 管理者の即時配信ボタン | （AI起動なし、SNS直接配信） | N/A |

**毎日9時トリガーの詳細フロー（拡張版）**:

```
[毎日9時 EventBridge]
  │
  ▼
[Orchestrator]
  │
  ├──► ① 配信判断・アンケート配信処理
  │     Survey Decision → (配信可なら) Survey Generation → SNS配信
  │
  ├──► ② 期日到来アンケート処理
  │     期日アンケート抽出 → Survey Result Analysis（次アクション判定）
  │       │
  │       ├─ 「イベント企画」と判定 → ③へ
  │       ├─ 「目安箱新トピック」と判定 → 管理者通知のみ
  │       └─ 「アクション不要」と判定 → 終了
  │
  └──► ③ イベント企画案生成（条件付き）
        Event Planning（目安箱+アンケート結果から企画案生成）
         → DynamoDB保存 → 保護者の Topページに表示
```

**Phase 2 以降に拡張する候補**:
- アンケート回答時のリアルタイム集計AI
- 目安箱投稿の類似投稿グルーピングAI
- 管理者の手動アンケート生成
- イベント企画後のタスク自動割り当て・実施フロー自動進行
- 学校カレンダー連携による具体日付の自動算出

---

## 8. 技術スタック確定

### 8.1 言語・ランタイム
| 種別 | 技術 |
|---|---|
| 言語 | TypeScript 6.0.x |
| バックエンドランタイム | Node.js 22.x LTS（AWS Lambda） |

### 8.2 フロントエンド
| 種別 | 技術 |
|---|---|
| Web フレームワーク | Next.js 16.x（App Router） |
| Web UIライブラリ | React 19.2.x |
| モバイル | React Native 0.85.x **+ Expo (EAS Build)**（.apk 生成簡素化のため Expo 採用を推奨） |
| モバイル対象プラットフォーム | **Android のみ（MVP）**、Phase 2 で Web、Phase 3 で iOS |
| **UIコンポーネント** | **shadcn/ui**（質問5 A） |
| **状態管理（クライアント）** | **Zustand 5.0.x** |
| **状態管理（サーバー）** | **TanStack Query** |
| HTTPクライアント | superagent 10.3.x |
| グラフ | Recharts 3.8.x |
| カレンダー | react-big-calendar（Phase 2 採用予定） |
| 多言語 | next-i18next / react-i18next |

### 8.3 バックエンド
| 種別 | 技術 |
|---|---|
| API フレームワーク | Express 5.2.x（Lambda 上） |
| 入力バリデーション | Zod 4.4.x |
| AWS SDK | @aws-sdk/* (3.x モジュラー) |
| Bedrock 連携 | @anthropic-ai/sdk 0.95.x（Bedrock 対応版検討） |
| JWT検証 | jsonwebtoken 9.0.x |

### 8.4 AWS サービス
テクニカル環境ドキュメントの「許可リスト」全サービス採用。**AWS リージョン: ap-northeast-1（東京）**（質問8 A）。

### 8.5 インフラ・モノレポ
| 種別 | 技術 |
|---|---|
| インフラコード | AWS CDK（TypeScript） |
| モノレポ管理 | **pnpm workspaces + Turborepo**（質問11 A） |

### 8.6 テスト
| 種別 | 技術 |
|---|---|
| 単体テスト | Jest + ts-jest |
| **PBT** | **fast-check**（PBT-09） |
| コンポーネントテスト | Jest + React Testing Library |
| E2Eテスト | Playwright |
| APIテスト | Jest + Supertest |
| インフラテスト | AWS CDK Assertions |

### 8.7 CI/CD（明確化1 A + 補足 b）

| ステージ | 実行場所 |
|---|---|
| トリガー（PR / push） | **GitHub Actions** |
| Lint・単体テスト | **GitHub Actions** |
| ビルド・E2Eテスト・cdk synth | **AWS CodeBuild** |
| デプロイ | **AWS CodePipeline + CodeDeploy（CDK Deploy）** |

GitHub Actions が AWS CodePipeline を Webhook 起動する構成。

---

## 9. 環境構成

| 項目 | 値 |
|---|---|
| プライマリリージョン | ap-northeast-1（東京） |
| **MVP環境** | **dev のみ**（質問9 C）。本番化のタイミングは Phase 2 着手時に再検討 |
| ローカル開発 | DynamoDB Local + LocalStack（一部） + Bedrock はモック/スタブ（質問19 A） |

### 9.1 ローカル開発環境の詳細設計（追加要望6）

**目的**: MVPリリース前にローカルで全機能の動作確認を可能にする。AWSデプロイ不要・コスト発生なし。

#### サービスごとのローカル戦略

| サービス | 戦略 | 詳細 |
|---|---|---|
| **Bedrock** | **モック実装必須**（コード分離）| `MockBedrockClient` 実装、決定的レスポンス or プリセット応答 |
| DynamoDB | DynamoDB Local（Docker）| 公式ローカル版、コード変更不要（endpoint 指定のみ）|
| S3 | LocalStack S3 | LocalStack 標準対応 |
| Cognito | cognito-local（OSS）+ fake JWT 発行ツール | LocalStack の Cognito は機能限定的 |
| SNS / SQS | LocalStack | LocalStack 標準対応 |
| EventBridge | npm script + 手動 invoke | MVP は手動トリガーで十分 |
| Step Functions | 手動 Lambda チェーン（local-orchestrator スクリプト）| Lambda 直接呼び出しの方がデバッグ容易 |
| API Gateway + Lambda | **Express を直接起動**（Lambda Adapter 不使用）| Express on Lambda の利点を活かしてポート3001 で素の Express として起動 |
| Web (Admin) | Next.js dev server（`pnpm dev`）| 標準 |
| Mobile (Parent) | Expo dev (`pnpm start` + Expo Go アプリ) | 標準、API endpoint をローカルに向ける |

#### 起動フロー（開発者の体験）

```bash
pnpm local:up           # docker-compose 起動（DynamoDB Local + LocalStack + cognito-local）
                        # + テーブル初期化 + シードデータ投入
                        # + Express ローカルサーバ起動（ポート3001）

pnpm local:web          # 別ターミナル: 管理者画面 (ポート3000)
pnpm local:mobile       # 別ターミナル: Expo Mobile (ポート19000等)

# 必要に応じて手動トリガー
pnpm local:trigger-daily    # 毎日9時の AI チェーンを手動発火
pnpm local:trigger-monthly  # 月初レポート生成を手動発火

pnpm local:down         # 全停止
```

#### 環境切替戦略

環境変数 `RUNTIME=local | dev | prod` で実装を自動切替:
- `local` → MockBedrockClient + DynamoDB Local + cognito-local + LocalStack エンドポイント
- `dev` / `prod` → 本物のAWSサービス利用

詳細な実装パターン・モノレポ構造への反映は Application Design 成果物（components.md / unit-of-work.md / services.md）を参照。

---

## 10. 制約事項

### 10.1 技術的制約
- 禁止技術: PHP、Ruby、Java（Kotlin 例外可）、Go、Hono、Fastify、Axios、Supabase、Firebase、Prisma、Redux、moment.js
- 禁止AWSサービス: EC2、ECS/EKS、Vercel/Netlify（本番）、Google Cloud / Azure
- Bedrock は IAM ロール認証必須（API キー禁止）
- **dev環境のみのMVP** = 本番デプロイ前にstaging/prod環境追加が必要
- **MVP プラットフォーム制約**: 保護者アプリは **Android のみリリース**（.apk 配布、Google Play 配信準備まで）。iPhone は Phase 3 の iOS リリース、PC・iPhone の暫定アクセス手段は Phase 2 の Web 版を待つ運用。

### 10.2 ビジネス制約
- 個人情報保護法準拠（学校単位データ分離）
- 児童個人情報をシステムに保持しない
- 会費の AI 自動変更は不可能（Lambda で制御、ビジョン記載）
- 操作ログの保護者公開（透明性担保、ビジョン記載）

### 10.3 コンプライアンス制約
- ISMS / ISO 27001 認証を目標
- HTTPS（TLS 1.2+）必須
- すべての永続化は暗号化必須

---

## 11. 前提・想定（Assumptions）

| ID | 内容 |
|---|---|
| ASSUM-01 | MVP の保護者数は 1校あたり最大1000世帯を想定 |
| ASSUM-02 | dev 環境のみでスタートし、本番化時に追加設計を行う前提 |
| ASSUM-03 | 「dev のみ × Security 強制」は将来の本番化を見越した先取り適用と解釈 |
| ASSUM-04 | プッシュ通知は MVP では「AIアンケート配信」のみ（月次レポート通知は Phase 2） |
| ASSUM-05 | Slack ワークスペースは MVP 開発開始時にプロジェクト用チャンネルを準備済み前提 |

---

## 12. オープン課題（Open Questions / TBD）

| ID | 課題 | 解決タイミング |
|---|---|---|
| OQ-01 | 本番環境（staging/prod）の構築タイミング・アカウント分離戦略 | Phase 2 着手時 |
| OQ-02 | Slack 通知用チャンネル名・Webhook URL の確保 | dev 環境構築時 |
| OQ-03 | 保護者の解約期間・データエクスポート期間の詳細 | プライバシーポリシー策定時 |
| OQ-04 | サポート体制（チャット/電話/メール）| Phase 1 リリース前 |
| OQ-05 | Bedrock Guardrails の具体的設定（拒否トピック・PII除外ルール） | Application Design 段階 |
| OQ-06 | プロンプトのゴールデンテストセットの初期コーパス作成方法 | Code Generation 段階 |
| OQ-07 | 学校コードの長さ・文字種・有効期限デフォルト | Application Design 段階 |
| OQ-08 | i18n の英語訳のレビュー体制（翻訳品質保証） | Code Generation 段階 |
| OQ-09 | アンケート配信判断の「投稿数しきい値」「経過時間しきい値」のチューニング指針 | Application Design 段階 |
| OQ-10 | 退会時の「アカウント削除」の物理削除/論理削除の選択 | Application Design 段階 |
| OQ-11 | 将来機能ナビゲーション（F-09）の Coming Soon 対象機能の最終リスト・優先順位 | Application Design 段階 |
| OQ-12 | イベント企画AI（F-08）の「賛成多数」判定しきい値（賛成数 / 反対数 / 興味なし数のロジック） | Application Design 段階 |
| OQ-13 | アンケート結果分析AI（F-02.7）の「次アクション判定」基準（どんな結果ならイベント企画AIを起動するか） | Application Design 段階 |
| OQ-14 | F-08 イベント企画AI の検証期間・成功指標（ユーザーフィードバックの収集方法） | MVPリリース後 |
| OQ-15 | F-10 参加者チェックイン方式: QRコード読取 / タップ式 / 両方併用 のいずれか | Application Design 段階 |
| OQ-16 | F-10 「実施確定」遷移後の保護者通知（プッシュ通知 / アプリ内のみ） | Application Design 段階 |
| OQ-17 | F-11 Expo 採用 vs bare React Native の最終決定（EAS Build と Lambda Bedrock SDK の互換性確認） | Application Design 段階 |
| OQ-18 | F-11 Google Play 開発者アカウント取得・本番配信手順の運用責任者 | MVP リリース前 |
| OQ-19 | F-11 アプリ内決済（IAP）が将来必要になるか（会費徴収 Phase 2 機能との関係） | Phase 2 着手時 |

---

## 13. 受け入れ基準（Acceptance Criteria - MVP全体）

MVPリリース可能と判断する基準:

- [ ] 機能要件 F-01 〜 F-11 すべてが dev 環境で動作する
- [ ] 全 SECURITY ルール（15個）の検証項目が満たされている、またはN/A理由が文書化されている
- [ ] PBT 適用対象ルール（02/03/07/08/09）の検証項目が満たされている
- [ ] AIエージェント分離（オーケストレーター + 6サブエージェント）が実装されている
- [ ] 毎日9時EventBridgeトリガーで「配信判断 → アンケート配信 → 結果分析 → イベント企画（条件付き）」のチェーンが動作する
- [ ] イベント企画案（F-08）が目安箱・アンケート結果を入力に生成され、賛否投票が機能する
- [ ] 将来機能ナビゲーション（F-09）に Coming Soon バッジ付きで5項目が表示される
- [ ] 参加者チェックイン（F-10）が動作し、管理者画面で参加者数が確認できる
- [ ] **保護者向け Android アプリの .apk が CI で自動ビルドされ、実機（または Android Emulator）で起動・動作確認できる**（F-11）
- [ ] iOS ビルドは行わないが、コード対応は Phase 3 で活性化可能な状態である
- [ ] プロンプトのゴールデンテストが CI で実行される
- [ ] CloudWatch Dashboard で主要メトリクスが可視化されている
- [ ] Slack アラートが少なくとも1件のテストイベントで発火することを確認
- [ ] 日本語・英語UIの主要画面が表示できる（Coming Soon バッジ含む）
- [ ] CI/CD パイプライン（GitHub Actions → CodeBuild → CodePipeline）が動作する

---

## 14. トレーサビリティ

質問回答 → 要件 ID マッピング（抜粋）:

| 回答 | 要件への反映 |
|---|---|
| 質問1 A: MVPスコープのみ | §4 MVP IN/OUT 全体 |
| 質問2 A: 管理者Web+保護者モバイル | §3 ステークホルダー、§8.2 |
| 質問3 A: Security 強制 | §6.4 全ルール、§13 受け入れ基準 |
| 質問4 B: PBT Partial | §6.5、§13 受け入れ基準 |
| 質問5 A: shadcn/ui | §8.2 |
| 質問6 X: Zustand+TanStack Query | §8.2 |
| 質問8 A: ap-northeast-1 | §9 |
| 質問9 C: dev のみ | §9、ASSUM-02 |
| 質問10 X / 明確化1 A+b: GHA + AWS統合 | §8.7 |
| 質問11 A: pnpm + Turborepo | §8.5 |
| 質問12 A: Cognito招待コード | F-05 |
| 質問13 X / 明確化2: AI判断 + 9時 + 即時配信 | F-02, F-03, AI-ARCH-04 |
| 質問14 A: Sonnet/Haiku 使い分け | AI-ARCH-04 |
| 質問15 A: WCAG AA 目標 | §6.6 |
| 質問16 C / 明確化3 X: 日英最低限UI | F-07 |
| 質問17 A: Slack通知 | NFR-MON-02 |
| 質問18 A: 退会時匿名化 | F-05.4 |
| 質問19 A: ローカル開発環境 | §9 |
| 質問20 / 明確化4: AIエージェント方針 | §7 全体 |
| **追加要望1**: Phase2/3機能チラ見せUI | F-09、§13 受け入れ基準、OQ-11 |
| **追加要望2**: イベント企画AI MVP追加（タスク割り当て除く） | F-08、§4.1/§4.2、§7.1 Event Planning Sub-Agent追加、OQ-12, OQ-14 |
| **追加要望3**: アンケート結果分析AIトリガー | F-02.7〜9、§7.1 Survey Result Analysis Sub-Agent追加、§7.3 トリガー2拡張、OQ-13 |
| **追加要望4 (MVP)**: 参加者チェックイン | F-10、§4.1 MVP IN追加、§13 受け入れ基準、OQ-15, OQ-16 |
| **追加要望4 (Phase 3)**: 当日リアルタイム機能群 | §4.2 MVP OUT に Phase 3 として明文化（タイムライン表示・写真共有・ヘルプAI・終了時アンケート）|
| **追加要望5 (MVP)**: Android のみリリース・.apk 配布 | F-11、§4.1 MVP IN追加、§8.2 Expo+EAS Build 採用、§10.1 制約追加、§13 受け入れ基準、OQ-17, 18, 19 |
| **追加要望5 (Phase 2)**: Web 版で iPhone/PC ユーザーをサポート | §4.2 Phase 2（既存「Web保護者向け」を補強） |
| **追加要望5 (Phase 3)**: iOS App Store リリース | §4.2 Phase 3（新規追加） |
| **追加要望6**: ローカル開発環境の設計詳細化 | §9.1 ローカル開発設計、Application Design 各種ドキュメントに反映 |

---

## 15. 次ステップ

本要件定義書が承認された後、以下に進む:
1. **User Stories**（条件付き、本プロジェクトでは複雑性とユーザー多様性から実施推奨）
2. **Workflow Planning**（必須）
3. **Application Design**（条件付き、本プロジェクトでは複雑性から実施推奨）
4. **Units Generation**（条件付き、本プロジェクトでは Lambda分離・モジュール多数から実施推奨）

