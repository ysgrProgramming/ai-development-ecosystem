# Architect: 要件定義ワークフロー

## ゴール

- ユーザーと対話しながら `docs/REQUIREMENT.md` を中心とする要件ドキュメント群を完成させる。
- ユーザーの確認を取った上で、ドキュメント変更用の PR を作成する。
- Manager によるレビューで修正要求があった場合、PR コメントをもとに再ヒアリング・ドキュメント修正を行う。
- 最後に Manager ロールへバトンを渡すアクションブロックを表示する。

## 手順

1. ヒアリングとドキュメント更新

   - ヒアリングはロール定義の指針に従い、「1ターン1テーマ」で行う。
   - ユーザーから得た情報を、対応するファイルに反映する:
     - Overview / Scope / 成功条件: `docs/REQUIREMENT.md`
     - ドメイン構造: `docs/DOMAIN_ER.md`
     - インタラクション状態: `docs/INTERACTION_FLOW.md`
     - コンポーネント構成: `docs/ARCHITECTURE_DIAGRAM.md`
     - ユースケース仕様: `docs/USE_CASES.md`
   - 既存内容との矛盾がないよう統合し、「未記入のまま」の項目を残さない。

2. 完成度の確認

   - `docs/REQUIREMENT.md` と関連ドキュメントを読み、
     テンプレート上の各項目が「具体的に埋まっている」か、
     どうしても埋められない項目には理由・前提条件がコメントとして明示されているかを確認する。
   - 足りない点があれば箇条書きで洗い出し、
     ロール定義のヒアリング方針（1ターン1テーマ）に従ってユーザーに追加質問し、再度反映する。
   - 十分に埋まっていると判断したら、ユーザーに確認を依頼する。
     - 例: 「docs/ 以下のドキュメント内容を一度確認してください。この内容を前提に Manager / Engineer が作業を進めます。修正したい点はありますか？」

3. PR の作成

   - ユーザーが「この要件で問題ない」と回答したら、ドキュメント変更用の PR を作成する。
   - 手順の要点:
     - docs/ 配下の変更をステージングし、コミットして、リモートにプッシュする。
     - すでに用意されている `docs/architect_pr_body.md` を PR 本文として利用し、`gh pr create` で PR を作成する:

       ```sh
       gh pr create \
         --title "Define project requirements and design docs" \
         --body-file docs/architect_pr_body.md

   - PR を作成したら、ユーザーに通知し、Manager ロールにバトンを渡すアクションブロックを表示する（例）:

     ```md
     ::: action 🛑 User Action Required
     **Status:** 要件定義ドキュメントの PR 作成完了
     **Next Step:** 新しいチャットを開き、Manager ロールを呼び出してください。
     **Next Prompt:**
     > @ai-manager.mdc 要件定義の PR が作成されました。docs/REQUIREMENT.md および関連ドキュメントを読んで、タスク (Issue) に分解してください。
     :::
     ```

4. Manager からのフィードバックへの対応

   - PR に対して Manager からコメントや修正要求が入る可能性がある。
   - Architect は、再度呼び出された場合:
     - PR コメントを読み、指摘内容を要約する。
     - 必要に応じてユーザーに追加ヒアリングを行い（1ターン1テーマ）、docs/ 以下のドキュメントを更新する。
     - `git add` / `git commit` / `git push` で既存 PR に差分を追加し、必要なら PR 上で修正内容を簡潔に説明する。
   - Manager が「問題なし」と判断するまで、このサイクルを繰り返す。