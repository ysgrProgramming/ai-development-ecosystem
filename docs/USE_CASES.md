# USE_CASES.md

<!--
このファイルでは、ユースケース仕様をテキストで記述する。
UMLの Fully Dressed Use Case を簡略化し、Domain / State / Operations と
トレースしやすい構造にしている。

関係:
- States: INTERACTION_FLOW.md の状態IDを参照する。
- Operations: REQUIREMENT.md の Public Operations / APIs の OP_ID を参照する。
- Domain: DOMAIN_ER.md のエンティティ・ルールと整合している必要がある。

ルール:
- 各ユースケースは UC-ID 単位で 1 ブロックとして記述する。
- Precondition / Postcondition はできる限り Domain の用語で書く。
- States / Operations は配列のように [] 内にIDを列挙する。
- ErrorCases は代表的なものだけでよいが、「どう扱うか」も簡潔に書くこと。
-->

## Use Cases

### UC-1

<!--
例:
- UC_ID        : UC_UPDATE_PROFILE
- Title        : プロフィール編集
- Actor        : 一般ユーザ
-->

- UC_ID: UC_CREATE_WORD_BOOK_AND_REGISTER_WORD
- Title: 自分専用の単語帳を作成し、単語を登録する
- Actor: User  <!-- 自分専用で利用するユーザー -->

- Goal: ユーザーが自分専用の単語帳を作成し、その中に単語・意味・任意のメモを登録できるようにする。

- Precondition: ユーザーがブラウザから本アプリを開いており、新しく作成したい単語帳名を決めている。
- Postcondition: 指定した名前の単語帳が1つ作成され、その単語帳の中に重複のない単語レコード（単語・意味・任意メモ）が複数登録されている。

- States: [STATE_HOME -> STATE_WORD_BOOK_DETAIL]  
  <!-- INTERACTION_FLOW.md の状態ID列。例: [STATE_HOME -> STATE_PROFILE_EDIT -> STATE_PROFILE_VIEW] -->

- Operations: [OP_CREATE_WORD_BOOK, OP_CREATE_WORD]  
  <!-- REQUIREMENT.md の OP_ID や主要な操作名を列挙する。 -->

- ErrorCases:
  - 必須項目（単語・意味）の入力漏れがある場合、エラーメッセージを表示し、その単語は登録しない。
  - 同じ単語帳内で、すでに登録済みの単語と同じ文字列の単語を登録しようとした場合、エラーメッセージを表示し、その単語は登録しない。

---

### UC-2

- UC_ID: UC_RUN_QUIZ_FROM_WORD_BOOK
- Title: 単語帳からクイズ（通常／復習）を実行する
- Actor: User

- Goal: ユーザーが選択した単語帳を対象に、通常出題または復習出題でクイズを行い、正解数と苦手単語を把握できるようにする。

- Precondition: ユーザーがブラウザから本アプリを開いており、少なくとも1つ以上の単語帳が存在している。
- Postcondition: ユーザーが指定した問題数（最大その単語帳の対象単語数）についてクイズが実行され、結果画面に正解数・不正解数・不正解だった単語一覧が表示される。不正解だった単語は苦手単語として記録され、苦手単語が2回連続で正解された場合は苦手単語から除外される。

- States: [STATE_HOME -> STATE_WORD_BOOK_DETAIL -> STATE_QUIZ -> STATE_QUIZ_RESULT]  

- Operations: [OP_START_NORMAL_QUIZ, OP_START_REVIEW_QUIZ, OP_UPDATE_QUIZ_PROGRESS, OP_FINISH_QUIZ]  

- ErrorCases:
  - 単語帳が1つも存在しない状態で出題モードを操作しようとした場合、出題開始はできず「単語帳がありません。まず単語帳を作成してください。」というメッセージを表示する。
  - 選択した単語帳に登録された単語が1件もない状態で出題開始しようとした場合、「この単語帳には単語がありません。単語を登録してください。」というメッセージを表示し、出題は開始しない。
  - 復習出題を選択したが、対象の単語帳に苦手単語が1件も存在しない場合、出題は開始せず「苦手な単語がありません。まず通常出題で学習してください。」というメッセージを表示する。

---

### UC-3

- UC_ID: UC_MANAGE_WORD_BOOK_AND_WORDS
- Title: 単語帳と単語を登録・編集・削除する
- Actor: User

- Goal: ユーザーが単語帳一覧から単語帳を選び、単語帳名や登録済みの単語を編集・削除したり、新しい単語帳や単語を追加できるようにする。

- Precondition: ユーザーがブラウザから本アプリを開いている。
- Postcondition: 単語帳一覧と各単語帳内の単語が、ユーザーの編集・削除・追加内容に応じて更新されている。単語帳を削除した場合、その単語帳に紐づく単語・クイズ履歴・苦手単語情報もすべて削除されている。

- States: [STATE_HOME -> STATE_WORD_BOOK_LIST -> STATE_WORD_BOOK_EDIT]  

- Operations: [OP_OPEN_WORD_BOOK_LIST, OP_CREATE_WORD_BOOK, OP_UPDATE_WORD_BOOK, OP_DELETE_WORD_BOOK, OP_CREATE_WORD, OP_UPDATE_WORD, OP_DELETE_WORD]  

- ErrorCases:
  - 単語帳名を編集または新規作成する際に、他の単語帳と同じ名前を設定しようとした場合、エラーメッセージを表示し、保存しない。
  - 単語を編集・追加する際に、「単語」または「意味」が未入力の場合、エラーメッセージを表示し、保存しない。
  - 単語を編集・追加する際に、同じ単語帳内で他の単語と同じ「単語」になってしまう場合、エラーメッセージを表示し、保存しない。