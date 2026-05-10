# ストーリー × ユニット マッピング — 全自動PTA - おやのわ

**ドキュメント種別**: AI-DLC Units Generation - Story Map 成果物
**作成日**: 2026-05-09
**ストーリー総数**: 32本
**ユニット総数**: 15（追加要望6 で U-LOCAL を新設）

---

## 1. ストーリー → ユニット マッピング表

凡例:
- ◎ = 主所属（Primary Owner）— このユニットの Functional Design / Code Generation でストーリーの主要受け入れ基準を実装
- ◯ = 副所属（Contributor）— このユニットも一部実装に関与する
- 空欄 = 関与なし

### F-01 デジタル目安箱

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F01-001 | 目安箱への投稿 | U-BE-SUGGEST-BOX | U-FE-PARENT, U-SHARED | Must |
| US-F01-002 | 投稿の自動カテゴリ分類による絞り込み | U-BE-SUGGEST-BOX | U-FE-PARENT, U-AI-AGENTS, U-AI-CORE | Must |
| US-F01-003 | 投稿への支持表明 | U-BE-SUGGEST-BOX | U-FE-PARENT | Must |
| US-F01-004 | 投稿対応状況の閲覧 | U-BE-SUGGEST-BOX | U-FE-PARENT, U-FE-ADMIN | Should |
| US-F01-005 | 匿名投稿のプライバシー保護 | U-BE-SUGGEST-BOX | U-FE-PARENT | Must |

### F-02 AIアンケート + 結果分析AI

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F02-001 | AIによるアンケート配信判断 | U-AI-AGENTS | U-AI-ORCHESTRATION, U-AI-CORE, U-BE-SURVEY, U-BE-NOTIFICATION, U-FE-PARENT | Must |
| US-F02-002 | アンケート回答 | U-BE-SURVEY | U-FE-PARENT | Must |
| US-F02-003 | アンケート結果のリアルタイム集計表示 | U-BE-SURVEY | U-FE-PARENT | Must |
| US-F02-004 | アンケート結果からの次アクション提案 | U-AI-AGENTS | U-AI-ORCHESTRATION, U-AI-CORE, U-BE-SURVEY, U-FE-ADMIN | Must |
| US-F02-005 | AIによる質問内容の品質期待 | U-AI-CORE | U-AI-AGENTS | Should |

### F-03 プッシュ通知

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F03-001 | AIアンケート配信通知の受信 | U-BE-NOTIFICATION | U-FE-PARENT, U-INFRA (NotificationStack) | Must |
| US-F03-002 | 管理者からの即時通知配信 | U-BE-NOTIFICATION | U-BE-ADMIN, U-FE-ADMIN | Must |

### F-04 月次レポート

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F04-001 | 月次レポートの自動配信 | U-BE-REPORT | U-AI-AGENTS, U-AI-ORCHESTRATION, U-AI-CORE, U-FE-PARENT | Must |
| US-F04-002 | 学校管理者によるレポート確認 | U-BE-REPORT | U-FE-ADMIN | Should |

### F-05 認証

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F05-001 | 学校コードを使った初回登録 | U-BE-AUTH | U-FE-PARENT, U-SHARED (shared-auth) | Must |
| US-F05-002 | ログイン・トークン検証 | U-BE-AUTH | U-FE-PARENT, U-FE-ADMIN, U-SHARED (shared-auth) | Must |
| US-F05-003 | 退会とデータ匿名化 | U-BE-AUTH | U-FE-PARENT, U-BE-SUGGEST-BOX (匿名化処理連動) | Should |
| US-F05-004 | 学校管理者の MFA 強制ログイン | U-BE-AUTH | U-FE-ADMIN | Must |

### F-06 基本管理者画面

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F06-001 | 学校コードの発行と無効化 | U-BE-ADMIN | U-FE-ADMIN | Must |
| US-F06-002 | 保護者一覧の閲覧 | U-BE-ADMIN | U-FE-ADMIN | Should |
| US-F06-003 | 破壊的操作の警告と公開操作ログ | U-BE-ADMIN | U-FE-ADMIN, U-BE-SUGGEST-BOX, U-BE-AUTH | Must |
| US-F06-004 | アクセス傾向の俯瞰 | U-BE-ADMIN | U-FE-ADMIN | Could |

### F-07 多言語UI

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F07-001 | 英語UIによる外国籍保護者対応 | U-SHARED (shared-i18n) | U-FE-ADMIN, U-FE-PARENT | Should |

### F-08 イベント企画AI

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F08-001 | AI企画案の閲覧 | U-BE-EVENT | U-AI-AGENTS, U-AI-ORCHESTRATION, U-AI-CORE, U-FE-PARENT | Must |
| US-F08-002 | イベント企画案への賛否投票 | U-BE-EVENT | U-FE-PARENT | Must |
| US-F08-003 | 企画 AI の実用性検証 | U-BE-EVENT | U-AI-CORE, U-FE-ADMIN | Should |

### F-09 Coming Soon UI

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F09-001 | 将来機能への期待感 | U-FE-PARENT | U-FE-ADMIN, U-SHARED (shared-i18n) | Should |

> **質問 B-2 A 反映**: U-FE-PARENT 主所属、U-FE-ADMIN は副所属（管理者向け Coming Soon リストは別実装）

### F-10 参加者チェックイン

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F10-001 | イベント当日のチェックイン | U-BE-EVENT | U-FE-PARENT | Must |
| US-F10-002 | 当日のリアルタイム参加者数把握 | U-BE-EVENT | U-FE-ADMIN | Must |

### F-11 Android アプリ配信

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-F11-001 | Android アプリの利用開始 | U-FE-PARENT | U-INFRA (MobileBuildStack) | Must |

### 横断（エラー・セキュリティ）

> **質問 B-1 A 反映**: 責任の中心となるユニットに割り当て

| Story ID | ストーリー要約 | 主所属 | 副所属 | 優先度 |
|---|---|---|---|---|
| US-XX-001 | ログイン失敗時のロックアウト | U-BE-AUTH | U-INFRA (MonitoringStack: Slack 通知), U-FE-PARENT, U-FE-ADMIN | Must |
| US-XX-002 | AI 呼び出し失敗時の代替動作 | U-AI-CORE | U-BE-SUGGEST-BOX, U-AI-AGENTS, U-AI-ORCHESTRATION, U-INFRA (Slack), U-FE-PARENT | Must |

---

## 2. ユニット → 担当ストーリー一覧

各ユニットが「主所属」として担当するストーリー数:

### U-FE-ADMIN（管理者向け Web）
**主所属ストーリー: 0本**（管理者画面のすべてのストーリーは Backend 主所属で、U-FE-ADMIN は副所属の UI 実装担当）

→ 機能要件視点では F-06 系・F-08 管理機能・F-10 管理機能・F-04 確認・F-09 管理者向けが該当。
→ ストーリーレベルでは Backend 主所属 + UI 副所属の構造。
→ **per-unit Construction 期間中に Backend ユニットと並行して UI 実装をレビュー対象に含める**

### U-FE-PARENT（保護者向け Android アプリ）
**主所属ストーリー: 2本**
- US-F09-001 (Coming Soon UI)
- US-F11-001 (Android アプリ利用開始)

**副所属ストーリー: 多数**（F-01〜F-08 のほぼすべての保護者向け UI）

### U-BE-AUTH（認証バックエンド）
**主所属ストーリー: 5本**
- US-F05-001, US-F05-002, US-F05-003, US-F05-004
- US-XX-001（ログイン失敗ロックアウト、横断）

### U-BE-SUGGEST-BOX（目安箱バックエンド）
**主所属ストーリー: 5本**
- US-F01-001, US-F01-002, US-F01-003, US-F01-004, US-F01-005

### U-BE-SURVEY（アンケートバックエンド）
**主所属ストーリー: 2本**
- US-F02-002, US-F02-003

### U-BE-NOTIFICATION（通知バックエンド）
**主所属ストーリー: 2本**
- US-F03-001, US-F03-002

### U-BE-REPORT（月次レポートバックエンド）
**主所属ストーリー: 2本**
- US-F04-001, US-F04-002

### U-BE-ADMIN（管理者操作バックエンド）
**主所属ストーリー: 4本**
- US-F06-001, US-F06-002, US-F06-003, US-F06-004

### U-BE-EVENT（イベントバックエンド）
**主所属ストーリー: 5本**
- US-F08-001, US-F08-002, US-F08-003
- US-F10-001, US-F10-002

### U-AI-CORE（AI 共通基盤）
**主所属ストーリー: 2本**
- US-F02-005 (AI 質問品質期待)
- US-XX-002 (AI 失敗時代替動作、横断)

### U-AI-AGENTS（AI サブエージェント群）
**主所属ストーリー: 3本**
- US-F02-001 (AI 配信判断)
- US-F02-004 (結果分析と次アクション提案)

> 注: US-F08-001 (企画案閲覧)・US-F04-001 (月次レポート) は主所属を Backend にしたが、AI 生成ロジックは U-AI-AGENTS 内のサブエージェントが担当する

### U-AI-ORCHESTRATION（AI ワークフロー）
**主所属ストーリー: 0本**（基盤）

> Step Functions State Machine と EventBridge スケジュールが実装されることで、F-02-001/004/F-04-001/F-08-001 のチェーンが成立する

### U-SHARED（共通パッケージ）
**主所属ストーリー: 1本**
- US-F07-001 (英語 UI)

### U-INFRA（インフラ）
**主所属ストーリー: 0本**（基盤）

> 全ストーリーの実行基盤として AWS リソース定義を担う

### U-LOCAL（ローカル開発環境、追加要望6 で新設）
**主所属ストーリー: 0本**（基盤）

> 全ストーリーのローカル動作確認を可能にする。直接的なストーリー所有はないが、MVP リリース前の検証フェーズで全ストーリーの受け入れ基準確認に利用される。
> 内部的には全 Backend Service と U-AI-CORE の MockBedrockClient を再利用するため、ストーリー実装そのものは各ユニットが行う。

---

## 3. ユニットごとの per-unit Construction フェーズ作業ボリューム

| ユニット | 主所属S数 | 副所属S数 | Functional Design 規模 | Code Generation 規模 |
|---|---|---|---|---|
| U-FE-ADMIN | 0 | ~15 | 中（UI 実装） | 大（多画面）|
| U-FE-PARENT | 2 | ~25 | 大（多画面・モバイル特有）| 大 |
| U-BE-AUTH | 5 | – | 中（Cognito 連携・MFA）| 中 |
| U-BE-SUGGEST-BOX | 5 | – | 中（CRUD + Streams）| 中 |
| U-BE-SURVEY | 2 | – | 小（CRUD + 集計）| 中 |
| U-BE-NOTIFICATION | 2 | – | 小（SNS Publish）| 小 |
| U-BE-REPORT | 2 | – | 小（GET 系のみ）| 小 |
| U-BE-ADMIN | 4 | – | 中（管理機能 + 公開操作ログ）| 中 |
| U-BE-EVENT | 5 | – | **大**（F-08 + F-10 両方、最大規模）| **大** |
| U-AI-CORE | 2 | – | 中（プロンプト管理・Guardrails・ゴールデンテスト基盤）| 中 |
| U-AI-AGENTS | 3 | – | **特大**（6エージェント分を章立て）| **特大** |
| U-AI-ORCHESTRATION | 0 | – | 中（Step Functions State Machine 設計）| 中 |
| U-SHARED | 1 | – | 中（5パッケージの I/O）| 中 |
| U-INFRA | 0 | – | **特大**（10スタック設計）| **特大** |

→ **U-AI-AGENTS, U-INFRA, U-FE-PARENT, U-BE-EVENT が最大ボリューム** = Construction フェーズの主要時間消費要因

---

## 4. カバレッジ検証

### 全ストーリーのユニット所属確認 (32/32)

| 機能 | 主所属ユニット数 | 主所属未割当 |
|---|---|---|
| F-01 | 5（全 U-BE-SUGGEST-BOX）| ✅ 0 |
| F-02 | 5（U-AI-AGENTS×2, U-BE-SURVEY×2, U-AI-CORE×1）| ✅ 0 |
| F-03 | 2（全 U-BE-NOTIFICATION）| ✅ 0 |
| F-04 | 2（全 U-BE-REPORT）| ✅ 0 |
| F-05 | 4（全 U-BE-AUTH）| ✅ 0 |
| F-06 | 4（全 U-BE-ADMIN）| ✅ 0 |
| F-07 | 1（U-SHARED）| ✅ 0 |
| F-08 | 3（全 U-BE-EVENT）| ✅ 0 |
| F-09 | 1（U-FE-PARENT）| ✅ 0 |
| F-10 | 2（全 U-BE-EVENT）| ✅ 0 |
| F-11 | 1（U-FE-PARENT）| ✅ 0 |
| 横断 | 2（U-BE-AUTH, U-AI-CORE）| ✅ 0 |
| **合計** | **32** | **✅ 0**（全ストーリーが主所属を持つ）|

### 全機能要件のカバレッジ確認

| 要件ID | カバーされるユニット | カバー状態 |
|---|---|---|
| F-01.1〜6 | U-BE-SUGGEST-BOX + U-AI-AGENTS (sub-classify) + U-FE-PARENT | ✅ |
| F-02.1〜9 | U-BE-SURVEY + U-AI-AGENTS + U-AI-ORCHESTRATION + U-FE-PARENT + U-FE-ADMIN | ✅ |
| F-03.1〜3 | U-BE-NOTIFICATION + U-FE-PARENT | ✅ |
| F-04.1〜3 | U-BE-REPORT + U-AI-AGENTS + U-AI-ORCHESTRATION + U-FE-PARENT + U-FE-ADMIN | ✅ |
| F-05.1〜5 | U-BE-AUTH + U-FE-PARENT + U-FE-ADMIN + U-SHARED + U-INFRA (Cognito) | ✅ |
| F-06.1〜5 | U-BE-ADMIN + U-FE-ADMIN | ✅ |
| F-07.1〜4 | U-SHARED + U-FE-ADMIN + U-FE-PARENT | ✅ |
| F-08.1〜7 | U-BE-EVENT + U-AI-AGENTS + U-AI-ORCHESTRATION + U-FE-PARENT + U-FE-ADMIN | ✅ |
| F-09.1〜6 | U-FE-PARENT + U-FE-ADMIN + U-SHARED | ✅ |
| F-10.1〜7 | U-BE-EVENT + U-FE-PARENT + U-FE-ADMIN | ✅ |
| F-11.1〜6 | U-FE-PARENT + U-INFRA (MobileBuildStack) | ✅ |
| AI-ARCH-01〜04 | U-AI-CORE + U-AI-AGENTS + U-AI-ORCHESTRATION | ✅ |
| AI-PROMPT-01〜04 | U-AI-CORE | ✅ |
| SECURITY-01〜15 | U-INFRA + U-SHARED + U-BE-AUTH + U-AI-CORE + 全ユニット | ✅ |
| PBT-02/03/07/08/09 | U-SHARED + U-BE-SUGGEST-BOX + U-BE-EVENT + U-AI-AGENTS + U-AI-CORE | ✅ |

---

## 5. ペルソナ別ストーリー分布のユニット確認

| ペルソナ | 関与ストーリー数 | 主に関わるユニット |
|---|---|---|
| **P-01 一般保護者** | 約 18 | U-FE-PARENT + 多数の Backend |
| **P-02 意欲的保護者** | 約 12 | U-FE-PARENT + U-BE-SUGGEST-BOX, U-BE-SURVEY, U-BE-EVENT |
| **P-03 学校管理者** | 約 14 | U-FE-ADMIN + U-BE-ADMIN, U-BE-AUTH (MFA), U-BE-EVENT |
| **P-04 学校の先生** | 限定的（MVP）| Phase 2 で拡大予定 |
| **P-05 新入学保護者** | 約 8 | U-FE-PARENT + U-BE-AUTH（オンボーディング）|

---

## 6. 並列開発時のストーリー独立性

per-unit Construction フェーズで複数ユニットを並列開発する場合の、ストーリー観点での独立性確認:

### 並列開発しやすい組（同時着手しても干渉が少ない）
- ✅ U-BE-SUGGEST-BOX + U-BE-SURVEY + U-BE-NOTIFICATION + U-BE-REPORT + U-BE-ADMIN（全て独立した CRUD 系）
- ✅ U-FE-ADMIN と U-FE-PARENT（異なるアプリで干渉なし）
- ✅ U-AI-CORE と U-INFRA 基盤部分

### 並列開発しにくい組（順序依存あり）
- ⚠️ U-BE-AUTH を先に完成させないと他 Backend は E2E テストできない
- ⚠️ U-AI-AGENTS は Backend Service の DynamoDB スキーマ確定後（U-SHARED の型確定が前提）
- ⚠️ U-AI-ORCHESTRATION は U-AI-AGENTS の Lambda ARN 確定後
- ⚠️ U-FE-* は対応する Backend Service が動作可能になってから本格 E2E

---

## 7. 次ステージ: CONSTRUCTION フェーズ

本ストーリーマップに基づき、CONSTRUCTION フェーズでは以下を per-unit ループで実行:

```
For each Unit (推奨順序: unit-of-work-dependency.md §3 参照):
  1. Functional Design (CONDITIONAL, 大半 EXECUTE)
     ↓ 主所属ストーリーの受け入れ基準を実装ロジックに落とす
  2. NFR Requirements (EXECUTE)
     ↓ SECURITY/PBT 担当ルールを確認
  3. NFR Design (EXECUTE)
     ↓ NFR パターンを組み込み
  4. Infrastructure Design (EXECUTE)
     ↓ CDK 定義を生成
  5. Code Generation (ALWAYS, Plan + Generation)
     ↓ AI 主導でコード生成、人間レビュー
After all units complete:
  6. Build and Test (ALWAYS)
     ↓ 全 32ストーリーの受け入れ基準を E2E で検証
```
