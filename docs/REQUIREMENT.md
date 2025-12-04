# Project Requirements

<!--
このファイルはプロジェクトの要件定義のメインエントリです。
Goal / Domain / Interaction / Architecture の4本柱をまとめて記述します。

他の設計ファイルとの関係:
- ドメイン構造: DOMAIN_ER.md
- インタラクション状態遷移: INTERACTION_FLOW.md
- コンポーネント構成: ARCHITECTURE_DIAGRAM.md
- ユースケース詳細: USE_CASES.md

運用ルール:
- 「TBD」や空欄のままにせず、すべての項目に具体的な内容を埋めてから次フェーズに進むこと。
- 説明文は必要に応じて削除してよいが、見出し・項目名は原則残すこと。
-->

---

## 1. Overview (Goal / Scope)

### 1.1 Product Summary

<!-- プロジェクトの概要を短く定義する。 -->

- Goal: 個人学習者が自分専用の単語帳を作成し、スマートフォンのブラウザからクイズ形式で反復学習できる Web アプリを提供する。  
- Target Users: 語学学習中の個人ユーザー（主に1人利用を想定）で、通勤・通学時間などのスキマ時間にスマホで単語を暗記したい人。  
- One-line Description: スマホから使える、自分で登録した単語だけを出題する個人向け単語帳クイズ Web アプリ。  

### 1.2 Scope

<!--
スコープ内とスコープ外を明確に切り分ける。
「やらないこと」を書くことでスコープの暴走を防ぐ。
-->

- In Scope:  
  - 単語帳の作成・一覧表示・編集・削除（単語帳名はシステム内で一意）  
  - 単語（単語・意味・メモ）の登録・編集・削除（同一単語帳内で単語は一意）  
  - 通常出題／復習出題によるクイズ機能（問題数指定・出題順設定）  
  - 苦手単語管理（不正解時に苦手登録し、2回連続正解で苦手解除）  
  - スマートフォン対応の Web UI と、バックエンド API＋DB によるデータ永続化  
- Out of Scope:  
  - 複数ユーザーアカウント管理やログイン機能  
  - 単語帳や成績の他ユーザーとの共有機能  
  - 完全オフライン動作（ネットワーク接続なしでの利用）  
  - 単語のインポート／エクスポート（CSV 等）  
  - 高度な間隔反復アルゴリズム（本バージョンでは簡易な苦手判定ロジックのみを提供）  

### 1.3 Success Criteria & Constraints

<!--
成功判定の基準と、守るべき制約条件を記述する。
例: 指標(DAU, 継続率, 応答時間)や締切、対応プラットフォームなど。
-->

- Success Criteria:  
  - 1人のユーザーが数百語規模の単語を登録し、スマートフォンのブラウザからストレスなくクイズ学習できる。  
  - 単語帳作成〜クイズ結果確認までが 3〜4 画面程度で完結し、操作に迷わない UI になっている。  
  - 苦手単語のみを復習出題でき、学習の進捗に応じて苦手単語が減っていくことをユーザーが把握できる。  
- Constraints:  
  - スマートフォンブラウザからの利用を第一優先とし、レスポンシブ Web とする。  
  - オンライン前提（バックエンド API＋DB への接続が必要）とし、完全オフライン動作は対象外とする。  
  - 想定する単語数は 1 ユーザーあたり数百語程度とし、数万語レベルのスケールは現時点では考慮しない。  
  - 個人利用を前提とし、ユーザー認証・認可は v1 では行わない（将来拡張を阻害しない設計とする）。  

---

## 2. Domain Model (Domain)

### 2.1 Domain Entities

<!--
ドメインエンティティとその関係は DOMAIN_ER.md に mermaid の erDiagram 形式で記述する。

DOMAIN_ER.md では、各エンティティに storage_scope をコメントで付与する運用を想定する:
- Ephemeral          : 1操作 / 1フレーム内だけで完結する一時データ
- Session            : セッション(ログイン中, ゲーム1プレイ中 等)の間維持されるデータ
- DeviceLocal        : デバイス上に永続化され、主にそのデバイスだけで利用されるデータ
- UserPersistent     : ユーザアカウントに紐づき、複数デバイス間で共有されるデータ
- GlobalPersistent   : システム全体で共有される永続データ
-->

- Domain ER Diagram: see `DOMAIN_ER.md`  

### 2.2 Domain Notes (Optional)

<!--
ER 図だけでは表現しづらいルールや補足がある場合に記述する。
何もなければ空でもよい。
-->

- Notes:  
  - 単語帳名はシステム内で一意とし、ユーザーが同じ名前の単語帳を複数作成できない。  
  - 各単語帳内では、「単語(term)」は一意とし、既存の単語と同じ term を持つ単語は登録・更新できない。  
  - 苦手単語は QUIZ_QUESTION_RESULT の履歴から算出しつつ、WEAK_WORD_STATUS に集約して保持する。  
  - WEAK_WORD_STATUS.isWeak は「直近で不正解が 1 回以上ある」または「consecutiveCorrectCount が 2 未満」のとき true とし、正解が 2 回連続すると false にする。  

---

## 3. Interaction / UI / Operations

### 3.1 Interaction State Flow

<!--
システム全体のインタラクション状態遷移は INTERACTION_FLOW.md に mermaid flowchart で記述する。

ルール:
- ノードIDは状態(State)IDとして扱う（例: STATE_TITLE, STATE_IN_GAME 等）。
- 矢印には可能な限り「イベント名」をラベルとして付ける。
  例: |start_button_clicked|, |timer_expired|, |message_received| など。
-->

- Interaction State Diagram: see `INTERACTION_FLOW.md`  

### 3.2 Public Operations / APIs

<!--
クライアント(人間・他システム・外部アプリ等)から見える操作/インターフェースを列挙する。
Web の場合は HTTP パス、CLI の場合はコマンド、ゲーム/組み込みの場合は公開APIやメッセージ名など。

列挙した OP_ID は USE_CASES.md の Operations から参照される前提。
-->

| OP_ID | Interface / Path / Command | Summary |
|-------|----------------------------|---------|
| OP_OPEN_WORD_BOOK_LIST | GET /api/word-books | Fetch list of all word books. |
| OP_CREATE_WORD_BOOK | POST /api/word-books | Create a new word book. |
| OP_UPDATE_WORD_BOOK | PUT /api/word-books/{wordBookId} | Update the name of a word book. |
| OP_DELETE_WORD_BOOK | DELETE /api/word-books/{wordBookId} | Delete a word book and all related data. |
| OP_CREATE_WORD | POST /api/word-books/{wordBookId}/words | Add a new word to a word book. |
| OP_UPDATE_WORD | PUT /api/word-books/{wordBookId}/words/{wordId} | Update term, meaning and note of a word. |
| OP_DELETE_WORD | DELETE /api/word-books/{wordBookId}/words/{wordId} | Delete a word from a word book. |
| OP_START_NORMAL_QUIZ | POST /api/quizzes/start | Start a normal quiz session for a word book. |
| OP_START_REVIEW_QUIZ | POST /api/quizzes/start | Start a review quiz session using weak words. |
| OP_UPDATE_QUIZ_PROGRESS | POST /api/quizzes/{quizSessionId}/answer | Submit answer result for a single quiz question. |
| OP_FINISH_QUIZ | POST /api/quizzes/{quizSessionId}/finish | Finish a quiz session and retrieve result summary. |

### 3.3 Use Cases

<!--
ユースケースの詳細仕様は USE_CASES.md に記述する。

ここでは「ユースケースの一覧」や「重要なユースケースIDだけ」を簡潔に列挙しておくとよい。
-->

- Key Use Case IDs: [UC_CREATE_WORD_BOOK_AND_REGISTER_WORD, UC_RUN_QUIZ_FROM_WORD_BOOK, UC_MANAGE_WORD_BOOK_AND_WORDS]  

---

## 4. Architecture

### 4.1 Components & Data Flow

<!--
システムを構成する主要なコンポーネント(箱)とデータフローは ARCHITECTURE_DIAGRAM.md に mermaid flowchart で記述する。

ルール:
- ノードはコンポーネント(例: Frontend, API Server, GameServer, DeviceController 等)。
- 外部サービス・外部デバイスはノード名に「(外部)」を付ける。
- 矢印には可能な限り「代表的な操作/プロトコル名」をラベルとして付ける。
  例: |HTTP /api/login|, |gRPC Match|, |I2C Read| など。
-->

- Architecture Diagram: see `ARCHITECTURE_DIAGRAM.md`  

### 4.2 Storage Scope Policy

<!--
DOMAIN_ER.md で付与した storage_scope を、具体的なストレージ/媒体にマッピングする方針を記述する。
例: UserPersistent -> クラウドDB, Session -> サーバメモリ+キャッシュ 等。
-->

- Ephemeral: クライアント側のコンポーネント状態や、画面内でのみ完結する一時的な入力値・クイズ進行中の UI 状態。  
- Session: 必要に応じてサーバ側セッションやトークンに一時キャッシュを保持しうるが、v1 では原則として UserPersistent に冪等に保存できる設計とする。  
- DeviceLocal: v1 ではクリティカルなデータは保持しない（将来、表示設定などのローカルキャッシュに利用する可能性あり）。  
- UserPersistent: WORD_BOOK / WORD / QUIZ_SESSION / QUIZ_QUESTION_RESULT / WEAK_WORD_STATUS をバックエンド DB に永続化し、将来のマルチデバイス利用にも対応可能とする。  
- GlobalPersistent: システム共通のマスタデータや設定値のみ（v1 では最小限に留める）。  

### 4.3 Non-Functional Requirements (Architecture-related)

<!--
アーキテクチャ設計に強く影響する非機能要件のみを記述する。
詳細なSLAや細かい数値要件がある場合は別ドキュメントに分けてもよい。
-->

- Performance / Throughput: 数百語規模のデータに対して、スマートフォンブラウザからの操作で体感 1 秒以内のレスポンスを目標とする。高頻度アクセスや大量同時接続は想定しない。  
- Security / Authentication / Authorization: v1 ではユーザー認証を行わないが、API レイヤでは入力バリデーションと SQL インジェクション／XSS 対策を行い、HTTPS 経由でのみアクセス可能とする。  
- Availability / Reliability / Backup: 個人用ツールとして常時稼働 SLA は厳格に定めないが、データ損失を避けるために DB のバックアップ手段（手動または自動）を用意する。  
- Observability (Logging / Metrics / Tracing / Alerting): クイズ開始・終了、エラー発生時のログを残せる設計とし、致命的なエラーの調査ができることを目指す。  
- Other NFRs: モバイルファーストのレスポンシブ UI とし、片手操作でも利用しやすい画面遷移・ボタン配置を意識する。  