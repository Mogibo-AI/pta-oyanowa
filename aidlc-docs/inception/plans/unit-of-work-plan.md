# ユニット分解計画 — 全自動PTA - おやのわ

**ドキュメント種別**: AI-DLC Units Generation - Plan 成果物
**作成日**: 2026-05-09
**ロール**: Software Architect

このドキュメントでは、Application Design で識別した **14ユニット候補** を正式に確定し、ユニット間の依存関係とストーリーマッピングを定義するための質問を提示します。

回答後、3つの成果物を生成します:
- `unit-of-work.md` — ユニット定義
- `unit-of-work-dependency.md` — 依存関係マトリクス
- `unit-of-work-story-map.md` — ストーリー × ユニット マッピング

---

## ベースライン: Application Design からの14ユニット候補

| Unit ID | 範囲 | 関連ストーリー数 |
|---|---|---|
| U-FE-ADMIN | 管理者 Web | 6（F-06系列・関連）|
| U-FE-PARENT | 保護者 Android アプリ | 16（F-01〜F-05, F-07〜F-11）|
| U-BE-AUTH | 認証バックエンド + Authorizer | 4 |
| U-BE-SUGGEST-BOX | 目安箱バックエンド + Stream | 5 |
| U-BE-SURVEY | アンケートバックエンド | 5 |
| U-BE-NOTIFICATION | 通知バックエンド | 2 |
| U-BE-REPORT | レポートバックエンド | 2 |
| U-BE-ADMIN | 管理者操作バックエンド | 4 |
| U-BE-EVENT | イベントバックエンド + Stream | 5 |
| U-AI-CORE | AI 共通基盤（C-AI-LIB）| 0（基盤）|
| U-AI-AGENTS | AI サブエージェント群 6体 | 0（基盤）|
| U-AI-ORCHESTRATION | Step Functions + EventBridge | 0（基盤）|
| U-SHARED | 共通パッケージ 5つ | 1（F-07）|
| U-INFRA | CDK スタック群 | 0（基盤、F-11含む）|

---

## セクション A: ユニット境界の確認（コア6問）

### 質問 A-1: 14ユニット構成の確定
Application Design で提案した14ユニット構成をそのまま採用しますか？

A) **14ユニット構成をそのまま採用**（推奨、Bounded Context と整合済み）
B) AI 関連の3ユニット（U-AI-CORE / U-AI-AGENTS / U-AI-ORCHESTRATION）を1つに統合 → 12ユニット
C) U-SHARED を5パッケージ別に分割（types/ui/api-client/auth/i18n）→ 18ユニット
D) U-INFRA を CDK スタック単位（10スタック）に分割 → 23ユニット
X) Other（自由記述）

[Answer]: A

---

### 質問 A-2: AI ユニットの粒度
AI 関連ユニット（U-AI-CORE / U-AI-AGENTS / U-AI-ORCHESTRATION）の分け方は？

A) **3ユニットに分離**（推奨、ライフサイクルとレイヤーが異なるため）
   - U-AI-CORE: Bedrock ラッパー・プロンプト管理・ゴールデンテスト基盤（共通ライブラリ）
   - U-AI-AGENTS: 6つのサブエージェント Lambda（個別エージェント実装）
   - U-AI-ORCHESTRATION: Step Functions ワークフロー + EventBridge トリガー
B) 1ユニットに統合（U-AI として1つの大ユニット、内部モジュール化）
C) サブエージェントごとに細分化（オーケストレーション + 6サブエージェント = 7ユニット）
X) Other（自由記述）

[Answer]: X 折衷案: A を採用しつつ、AGENTS ユニットの内部に明確なモジュール分離を強制する。具体的には ①Functional Design で6エージェント分を章立てて個別に詳細記述する ②`backend/functions/ai/sub-*/` でディレクトリ分離する ③Code Generation Plan で各サブエージェントを段階的に着手可能な構成にする

---

### 質問 A-3: 共有パッケージの粒度
U-SHARED（C-PKG-TYPES / UI / API-CLIENT / AUTH / I18N の5パッケージ）の扱いは？

A) **U-SHARED 1ユニットに統合**（推奨、依存関係がシンプル、独立デプロイ不要）
B) 5パッケージごとに別ユニット化（types/ui/api-client/auth/i18n 各々独立）
C) 2グループに分割（types は基盤として独立、他4つは UI 系として統合）
X) Other（自由記述）

[Answer]: A

---

### 質問 A-4: インフラユニットの粒度
U-INFRA（CDK 10スタック）の扱いは？

A) **U-INFRA 1ユニットに統合**（推奨、CDK プロジェクトとして1つで管理しやすい、スタック間依存も同一プロジェクト内）
B) スタックごとに分割（NetworkStack/AuthStack/DataStack/ApiStack/AiStack/NotificationStack/MonitoringStack/SecretsStack/FrontendStack/MobileBuildStack）
C) 機能群でグループ化（Core基盤 / API系 / AI系 / フロント系 の4ユニット）
X) Other（自由記述）

[Answer]: A

---

### 質問 A-5: デプロイ単位の独立性
各ユニットの独立デプロイ性は？

A) **共通CDKデプロイ**（推奨、MVP 規模でデプロイをまとめる、Lambda は CDK で個別デプロイ）
B) ユニットごとに独立 CI/CD パイプライン（マイクロサービス的）
C) フロントエンド系は独立、バックエンド系は CDK で統合
X) Other（自由記述）

[Answer]: A

---

### 質問 A-6: チーム構成・所有権境界
ユニットを誰が担当する想定ですか？（並列開発の前提）

A) **小規模チーム（1〜3名）が全ユニットを順次担当**（推奨、MVP規模に合致）
B) 中規模チーム（4〜6名）が機能エリアごとに分担（FE/BE/AI/インフラ）
C) フルスタック型（各メンバーが特定ユニット群を縦に担当）
D) AI（Claude Agent SDKなど）が大半のコード生成を担当、人間がレビュー
X) Other（自由記述）

[Answer]: D

---

## セクション B: ストーリーマッピング戦略（2問）

### 質問 B-1: 横断的ストーリーの所属
US-XX-001（ログイン失敗ロックアウト）/ US-XX-002（AI 呼び出し失敗時の代替動作）など横断的ストーリーは？

A) **責任の中心となるユニットに割り当て**（推奨、US-XX-001 → U-BE-AUTH / US-XX-002 → U-AI-CORE & U-BE-SUGGEST-BOX 等）
B) 別途「横断ユニット」を1つ追加して集約
C) 全ユニットに共通要件として明示（特定のユニット所有なし）
X) Other（自由記述）

[Answer]: A

---

### 質問 B-2: F-09 Coming Soon UI ストーリー（US-F09-001）の所属
このストーリーは管理者・保護者・新入学保護者すべてに関わります。

A) **U-FE-PARENT に主所属、U-FE-ADMIN は副所属（管理者向け Coming Soon リストは別実装）**（推奨）
B) U-FE-ADMIN と U-FE-PARENT 両方に同等所属
C) U-SHARED に所属（UIコンポーネントとしての位置付け）
X) Other（自由記述）

[Answer]: A

---

## セクション C: 自由記入

### 質問 C-1: 上記で網羅していないユニット分解の希望・制約があれば
（特になければ「なし」）

[Answer]: なし

---

## セクション D: 計画実行ステップ（承認後に実行）

セクション A・B・C の質問にすべて回答後、Part 2 で以下のステップを実行します。

### Phase D-1: ユニット分解の確定
- [x] D-1.1 質問 A-1〜A-4 の回答に基づき、最終ユニット数を確定
- [x] D-1.2 各ユニットの境界・責任・関連コンポーネントを定義
- [x] D-1.3 デプロイ単位・所有権境界を明示
- [x] D-1.4 `aidlc-docs/inception/application-design/unit-of-work.md` を生成
- [x] D-1.5 グリーンフィールドのコード組織戦略を `unit-of-work.md` に含める（モノレポ構造）

### Phase D-2: 依存関係の整理
- [x] D-2.1 質問 A-5 を反映してユニット間依存マトリクスを作成
- [x] D-2.2 ユニットの開発順序・並列開発可能性を判定
- [x] D-2.3 Critical Path を特定（ブロッキングユニット）
- [x] D-2.4 `aidlc-docs/inception/application-design/unit-of-work-dependency.md` を生成

### Phase D-3: ストーリーマッピング
- [x] D-3.1 全32ストーリーを各ユニットに割り当て（質問B-1, B-2 反映）
- [x] D-3.2 各ユニットの担当ストーリー一覧を生成
- [x] D-3.3 主所属・副所属の区別を明示
- [x] D-3.4 `aidlc-docs/inception/application-design/unit-of-work-story-map.md` を生成

### Phase D-4: 検証
- [x] D-4.1 全32ストーリーがどこかのユニットに所属しているか確認
- [x] D-4.2 全機能要件 F-01〜F-11 がユニットでカバーされているか確認
- [x] D-4.3 SECURITY 拡張ルール 15個のユニット担当が明確か確認
- [x] D-4.4 PBT 適用ルール 5個のユニット担当が明確か確認
- [x] D-4.5 audit.md 更新
- [x] D-4.6 aidlc-state.md 更新

### Phase D-5: 完了報告
- [x] D-5.1 ユーザー向け完了メッセージ準備
- [x] D-5.2 承認待ち状態に遷移

---

## 回答方法

セクション A・B・C の質問（合計9問）にすべて `[Answer]:` で回答してください。
回答完了後、チャットで「**ユニット計画承認**」とお伝えください。
（修正依頼の場合は「**修正依頼: 〇〇**」のように）

---

## 補足: なぜ Units Generation が必要か

Application Design で定義した **コンポーネント** は技術的構造（Lambda・テーブル・ライブラリ）の単位ですが、**ユニット** は **開発作業単位 + per-unit Construction フェーズの実行単位** です。

具体的に Units Generation 完了後、CONSTRUCTION フェーズでは:
- 各ユニットに対して per-unit ループ実行（Functional Design → NFR Requirements → NFR Design → Infrastructure Design → Code Generation）
- ユニット単位で並列開発可能性が決まる
- ユニット境界がコードリポジトリのフォルダ構造と一致する
