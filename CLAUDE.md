# CraftShelf v03 サービス版 開発メモ

- **ここは `mesamaru/craftshelf-server`(v03 の多くの人が使うサービス版)**。自分で入れて使う版は `mesamaru/craftshelf` の `dev`(v02 は同じリポジトリの `main`)。
- **v03 は `craftshelf` の `dev` を中心に作る**: 利用者が `dev` を見て意見を出す → `dev` に作る → 同じ機能をこのサービス版にも入れる。サービス版は `dev` に合わせて更新する。
- **`dev` から取り込む手順**: 手元の作業フォルダーには `origin`(craftshelf)と `server`(craftshelf-server)の 2 つのリモートをつないでおく。`git switch server` → `git cherry-pick <dev のコミット>` で取り込み、`VERSION`・`CHANGELOG.md` はサービス版の番号で書き直してからコミットする。サービス版にだけ要るもの(チームの上限・公開の入口など)は、サービス版だけで作る。`dev` の `DATA_FORMAT`(v02 のパネル向けの目印)と `.github/workflows/image.yml`(自分用の版のイメージ・プレリリースを作る)はサービス版には要らないので取り込まない。
- **動き方は 2 つ**(`CRAFTSHELF_MODE`、`src/server/env.ts`): コードは `dev` と同じで、サービス版は `service` で動かす(`docker-compose.service.yml`。`.env` の `COMPOSE_FILE` で選ぶ)。`self`(自分用)だけの処理(v02 からの取り込み・版の切り替え・家の中のアドレスへの接続)はサービス版では動かない。どちらでも動くように作る。
- 版は `0.xx.xx`(大型.メジャー.マイナー。`dev` と同じ形で、0.04.00 から。03.29.00 までは `03.xx.xx`。番号は `dev` とは別に数える)。タグは `v0.04.01` の形(リリースの仕組みはまだ無い)。

- 利用者への説明・UI の文言は日本語。利用者が入力した名前・メモなどには `translate="no"` を付ける。
- 設計と方針は `docs/DESIGN.md`。構成を変えたらそちらも直す。
- **変更をコミットするたびに `VERSION` を上げ、`CHANGELOG.md` の先頭に見出しを追加する。** 形式は `0.00.00`(大型.メジャー.マイナー。0.04.00 から。大型は 1 けた、メジャー・マイナーは 2 けたで、修正の桁は無い)、見出しは `## 0.04.01 - YYYY-MM-DD タイトル`。
  - マイナー: コミットごとに上げる(機能の追加も修正も)。
  - メジャー: 大きな区切りで上げ、マイナーを 00 に戻す(マイナーが 99 になったときも)。自動アップデートで入る。
  - 大型: 製品の世代。上げた版には自動アップデートで入らない。上げるのはオーナーが決めたときだけ。
  - 03.29.00 までは旧の形 `03.xx.xx`(メジャー.マイナー.修正)。版は `src/lib/version.ts` の `compareVersion` で比べる(文字のまま比べない。`0.04.00` は `03.29.00` の次)。
- 構成: TanStack Start(`src/routes/` がページとサーバーのルート)・Drizzle + PostgreSQL(`src/server/db/`)・Better Auth(`src/server/auth.ts`)。サーバーだけで動く処理は `src/server/` に置き、画面側のファイルから直接 import しない(`createServerFn` の中で動的に import する)。
- **データベースの変更は「足すだけ」**(表・項目を消したり、名前や意味を変えたりしない)。スキーマを変えたら `npm run db:generate` で `drizzle/` に移行の SQL を足す。認証の表は `npm run auth:schema` で作る(`auth-schema.ts` を手で書き換えない)。
- **テスト**: コミットの前に `npm run typecheck`・`npm test`・`npm run build` を通す。テストは `tests/*.test.ts`(PGlite を使うので PostgreSQL は要らない)。
- 認証の確認は、画面の `beforeLoad` だけでなく、サーバー側の処理ごとに必ず行う(`src/server/guard.ts` の `requireUser`・`requireTeam`)。権限は `src/lib/permissions.ts` に足す。データベースの読み書きは、選んでいるチームの ID で必ず絞る。
- **利用者が入力した URL へサーバーからつなぐときは、必ず `src/server/net.ts` の `safeFetch` を使う**(LAN の中へつながせない)。利用者から預かるキーは `src/server/secrets.ts` の `seal` で暗号化して保存する。
- 画面を変えたら、ビルドしたサーバーを `PORT=3123` で動かして `npm run e2e` も通す(Edge で操作する。画面の写しは `.e2e/`)。
- コミットの作成者は `mesamaru <74803890+mesamaru@users.noreply.github.com>`。コード・例・コミットメッセージに、実際の IP アドレス・共有名・ドメインなど個人の環境の情報を書かない。
- **開発が終わったら、版と変更内容を利用者に報告し、公開(push・タグ)してよいか確認してから公開する。**
- デザインは Apple の Liquid Glass を参考にする(半透明のガラス素材・グループリスト・カプセル型ボタン)。
