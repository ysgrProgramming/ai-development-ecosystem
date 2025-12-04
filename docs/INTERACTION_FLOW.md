# INTERACTION_FLOW.md

<!--
このファイルでは、ユーザ視点またはシステム視点の「インタラクション状態遷移」を
mermaid の flowchart で記述する (UML 状態マシン図の簡易版)。

目的:
- システムがどのような状態(State)を持ち、どのイベントで状態が変わるかを俯瞰する。
- USE_CASES.md の States 項目から参照される「状態ID」の定義場所とする。

ルール:
- ノードIDは状態IDとして一意に命名する (例: STATE_TITLE, STATE_IN_GAME, STATE_ERROR)。
- 矢印には「イベント名」をできるだけラベルとして明記する。
  - 例: |start_button_clicked|, |login_success|, |timeout|, |sensor_triggered| など。
- ガード条件や細かい例外条件は、必要に応じて [condition] のような簡潔な表記か、
  USE_CASES.md 側の Precondition / ErrorCases に寄せる。
-->

```mermaid
flowchart TD
  STATE_HOME["STATE_HOME\nホーム（単語帳登録・編集／クイズ入口）"]

  STATE_WORD_BOOK_LIST["STATE_WORD_BOOK_LIST\n単語帳一覧"]
  STATE_WORD_BOOK_EDIT["STATE_WORD_BOOK_EDIT\n単語帳編集（単語追加・編集・削除／単語帳削除）"]

  STATE_WORD_BOOK_DETAIL["STATE_WORD_BOOK_DETAIL\n出題設定（単語帳選択・モード選択・問題数指定）"]
  STATE_QUIZ["STATE_QUIZ\nクイズ進行（一問ずつ出題）"]
  STATE_QUIZ_RESULT["STATE_QUIZ_RESULT\n結果表示（正解数・苦手単語）"]

  %% 単語帳管理フロー
  STATE_HOME -->|word_book_register_edit_clicked| STATE_WORD_BOOK_LIST
  STATE_WORD_BOOK_LIST -->|select_existing_word_book| STATE_WORD_BOOK_EDIT
  STATE_WORD_BOOK_LIST -->|create_new_word_book| STATE_WORD_BOOK_EDIT
  STATE_WORD_BOOK_EDIT -->|back_to_home| STATE_HOME

  %% クイズ実行フロー
  STATE_HOME -->|open_quiz_setup| STATE_WORD_BOOK_DETAIL
  STATE_WORD_BOOK_DETAIL -->|start_normal_quiz| STATE_QUIZ
  STATE_WORD_BOOK_DETAIL -->|start_review_quiz| STATE_QUIZ

  STATE_QUIZ -->|finish_all_questions| STATE_QUIZ_RESULT
  STATE_QUIZ -->|user_cancel_quiz| STATE_QUIZ_RESULT

  STATE_QUIZ_RESULT -->|back_to_home| STATE_HOME
```