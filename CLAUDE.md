# レコルト 自動調理ポット レシピ帳 — 引き継ぎ書

récolte［自動調理ポット RSY-2］用のレシピ帳 Web アプリ。単一 HTML。

- **公開先**：https://konkora780-ux.github.io/recolte-cooking-pot/
- **リポジトリ**：https://github.com/konkora780-ux/recolte-cooking-pot （main / public）
- **編集場所**：`D:\Claudプロジェクト\レコルト自動調理ポット\index.html`
  clone は毎回スクラッチパッドに作る（日本語パスは clone 先に使わない）
- 姉妹アプリ：[Ninja-Crispi](https://github.com/konkora780-ux/Ninja-Crispi)（同じ構造・同じ操作感）

## 機種の仕様（まちがえないこと）

ユーザーの実機は **RSY-2（標準・容量約600ml）**。

- **温度も時間も設定できない。** 決めるのは【SELECT】ボタンで選ぶ**モードだけ**。
  アプリの `min` はタイマー用の目安であって、機械の設定値ではない
- モードは **5種類**：`POTAGE&PASTE` / `SOUP&OKAYU` / `SOYMILK` / `JUICE&CLEAN` / `WARM`
- **RSY-3（ラージ）は別機種**。容量約800ml・7モード・日本語モード名（ポタージュ｜ペースト、
  たべるスープ、スープ、おかゆ、豆乳、ジュース｜CLEAN、保温）・デジタル表示。
  お手本レシピのうち **39件は RSY-3 用**なので `pot:'RSY-3'` を持ち、
  詳細画面に警告と「RSY-2 の分量にする（3/4）」ボタンを出している
- 材料は必ず **液体 → やわらかいもの → かたいもの・肉魚 → 氷** の順。
  逆にすると刃が回らず保護回路で止まる。だから材料リストに**番号を振って**見せている
- 牛乳・豆乳の少量使用、でんぷん質の入れすぎでふきこぼれる。豆乳で代用するときは **豆乳2：水1**
- お手入れは水＋中性洗剤1〜2滴で【JUICE&CLEAN】約3分

## データ

- 保存：localStorage キー `recolte-pot-recipes-v1`
  ／お手本投入フラグ `recolte-pot-seeded-v1`（**お手本を増やしたらこの番号を上げる**）
- 写真は 1枚1キーで別保存 `recolte-pot-photo-<id>`（写真で容量オーバーしてもレシピが壊れないように）
- お手本レシピ **133件**。全件が **recolte-jp.com 公式レシピ**が出典で、`link` に公式ページURLを持つ
  - 📖 の公式一覧は `SEED` から自動生成している（別配列を持っていない）
  - **公式の写真は埋め込まない**。よその写真を再配信することになり、画像URLもいずれ切れるため。
    公式ページへのリンクにしてある。自分で撮った写真は貼れる（長辺900pxのJPEG）

### お手本レシピの作り直し方

スクラッチパッドに残したスクリプトで作った（セッションが変わると消えるので、必要なら再作成）。

1. `https://recolte-jp.com/tags/<tag>/page/N/` を巡回してレシピIDを集める。
   タグは `auto-cooking-pot`, `rsy-3`, `rsy2_potage`, `rsy2_soup`, `rsy2_okayu`,
   `rsy2_soymilk`, `rsy2_juice`, `rsy2_warm`。**1ページ5件・ページ送りあり**
2. 各 `https://recolte-jp.com/recipe/<id>/` を取得。サーバー描画なので curl で読める。
   `.p-recipes_box` が「材料」「材料を入れる順番」「トッピング」「作り方」の箱、
   中の `.p-recipes_check` が各行。材料は `名前…分量` の形
3. モードはタグから：`rsy2-mode01`=SOYMILK / `mode02`=POTAGE&PASTE /
   `mode03`=SOUP&OKAYU / `mode04`=JUICE&CLEAN。
   RSY-3 のページはタグが無いので本文の【　】を日本語モード名から読みかえる
4. カテゴリもタグから：`rsy2_potage`/`_gusoup`/`_currystew`/`_okayurisotto`/
   `_soymilk`/`_pastasaucetare`/`_smoothie`/`_sweets`

NotebookLM ノートブック「レコルト　自動調理ポット」（id `279a3e74-3dd9-451d-9699-a104a5cb85b6`）
にも資料36件がある。中でも生成レポート
「レコルト自動調理ポットにおける調理モード別熱化学特性と応用レシピの多角的研究レポート」が、
モードの仕組み・ふきこぼれ・トラブル対処をよくまとめている（`nlm content source <id>` で読める）。

## 実装のメモ

- 分量の人数倍は `scaleAmount()`。日本語の分量は「大さじ1.5」「小さじ1/2」のように
  数字が先頭に来ないので、文字列中の**最初の数値だけ**を掛ける。
  `fmtNum()` は 10未満なら「2と1/4」、10以上は整数、50以上は5きざみに丸める
  （「112と1/2g」を出さないため）
- タイマーは終了時刻(Date)からの逆算＋`visibilitychange` で描き直し（iOSは裏で止まる）。
  アラームは WebAudio、開始タップで `primeAudio()`
- シートは `transform` でスライドさせているので、**中の固定物は `position:fixed` が効かない**。
  `position:sticky` で作ること。タイマーの高さは `--timerh` で他の固定要素に伝えている
- **file:// で開くと localStorage が使えない環境がある**。確認は `python -m http.server` 経由で

## ビルドと公開

`index.html` を直接編集してよい（テンプレート＋seed.json から生成したが、成果物は単一ファイル）。
アイコン（favicon / apple-touch-icon / icon-192 / icon-512）と manifest.webmanifest は同梱ずみ。

```
git add -A && git commit -m "..." && git push
```

push すると GitHub Pages が数分で更新される。
