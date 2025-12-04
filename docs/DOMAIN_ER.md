# DOMAIN_ER.md

<!--
このファイルでは、ドメインエンティティとその関係を mermaid の erDiagram 形式で記述する。

目的:
- 「このシステムの世界には何が存在し、どう関係しているか」をユーザーと合意する。
- クラス図やテーブル定義など、後続の詳細設計のベースとする。

ルール:
- 永続/非永続に関わらず「ドメインとして意味のあるエンティティ」を列挙する。
- 各エンティティの属性コメントに storage_scope を付与する (任意だが推奨):

  storage_scope 候補:
    - Ephemeral        : 1操作 / 1フレーム内だけで完結
    - Session          : セッション(ログイン中, ゲーム1プレイ中 等)の間維持
    - DeviceLocal      : デバイスローカルに永続化
    - UserPersistent   : ユーザアカウントに紐づき複数デバイス間で共有
    - GlobalPersistent : システム全体で共有される永続データ

- 関係(1:1, 1:N, N:M等)には簡潔な説明ラベルを書くとよい。
-->

```mermaid
erDiagram
  WORD_BOOK ||--o{ WORD : "単語帳は複数の単語を持つ (1:N)"
  WORD_BOOK ||--o{ QUIZ_SESSION : "単語帳は複数のクイズセッションを持つ (1:N)"
  QUIZ_SESSION ||--o{ QUIZ_QUESTION_RESULT : "クイズセッションは複数の設問結果を持つ (1:N)"
  WORD ||--o{ QUIZ_QUESTION_RESULT : "単語は複数のクイズ結果に紐づく (1:N)"
  WORD ||--o| WEAK_WORD_STATUS : "単語は0または1の苦手状態を持つ (0:1)"

  WORD_BOOK {
    string id PK "storage_scope: UserPersistent / UUID"
    string name "storage_scope: UserPersistent / 必須 / システム内で一意"
    datetime createdAt "storage_scope: UserPersistent"
    datetime updatedAt "storage_scope: UserPersistent"
  }

  WORD {
    string id PK "storage_scope: UserPersistent / UUID"
    string wordBookId FK "storage_scope: UserPersistent / ref: WORD_BOOK.id"
    string term "storage_scope: UserPersistent / 必須 / 単語（同一wordBook内で一意）"
    string meaning "storage_scope: UserPersistent / 必須"
    string note "storage_scope: UserPersistent / 任意"
  }

  QUIZ_SESSION {
    string id PK "storage_scope: UserPersistent / UUID"
    string wordBookId FK "storage_scope: UserPersistent / ref: WORD_BOOK.id"
    string mode "storage_scope: UserPersistent / NORMAL|REVIEW"
    int totalQuestionCount "storage_scope: UserPersistent / 1以上"
    int answeredQuestionCount "storage_scope: UserPersistent / 0以上"
    datetime startedAt "storage_scope: UserPersistent"
    datetime finishedAt "storage_scope: UserPersistent / 任意"
  }

  QUIZ_QUESTION_RESULT {
    string id PK "storage_scope: UserPersistent / UUID"
    string quizSessionId FK "storage_scope: UserPersistent / ref: QUIZ_SESSION.id"
    string wordId FK "storage_scope: UserPersistent / ref: WORD.id"
    int orderIndex "storage_scope: UserPersistent / 0以上 / 出題順を表す"
    boolean isCorrect "storage_scope: UserPersistent"
  }

  WEAK_WORD_STATUS {
    string wordId PK "storage_scope: UserPersistent / ref: WORD.id"
    boolean isWeak "storage_scope: UserPersistent / 不正解でtrue / 2回連続正解でfalse"
    int consecutiveCorrectCount "storage_scope: UserPersistent / 0以上 / 2で苦手解除"
  }