# CLAUDE.md

## プロジェクト概要

入力したキーワードの埋め込みベクトルを 3D で可視化する、単一 HTML のデモ（`index.html`）。
初学者向け勉強会「はじめてのベクトルデータ」の補助教材として使う。
UI の文言は日本語で書く。

## 方針

- **ワンソースを維持する。** `index.html` 1 ファイルで完結させ、ビルド工程を持ち込まない。
  - 外部依存は CDN から読み込む。three.js は importmap、Transformers.js は dynamic import。
- 計算はすべてブラウザ内で行う。外部 API や API キーは使わない。
- 見た目は勉強会スライドの配色に合わせる。CSS 変数は `:root` に定義している。
  - navy `#1B2340` / cream `#F7F5EF` / accent `#B5531F` / stage `#141B33`
- ライト / ダークの 2 テーマに対応している。
  - 色を足すときは直書きせず、CSS 変数を `:root`、`:root[data-theme="dark"]`、`@media (prefers-color-scheme: dark)` の 3 か所に定義する。
  - 3D ステージはどちらのテーマでも暗色のまま。
  - テーマ切り替えのコードはモジュールとは別の通常 `<script>` に置く。CDN の読み込みに失敗しても切り替えが動くようにするため。
  - 選択は `localStorage` の `kv3d-theme` に保存する。`<head>` の小さなスクリプトで描画前に反映し、ちらつきを防いでいる。

## コード構成（`index.html` の `<script type="module">`）

| ブロック | 内容 |
| --- | --- |
| 状態 | `state.words`（追加順）、`state.vecs`（word → 正規化ベクトル）、`state.items`（word → mesh / label / pos / target） |
| モデル | `getExtractor()` でパイプラインを遅延ロード（進捗表示つき）。`embedMissing()` で未計算の語だけベクトル化する |
| 計算 | `pca3()`：グラム行列のべき乗法による PCA（点が少ない前提）。`layout()`：前回の位置に合わせて軸の符号をそろえ、半径 10 に正規化する |
| 描画 | `createItem()` / `disposeItem()` で点を生成・破棄する。`tick()` で目標位置へ lerp アニメーションする |
| UI | `renderChips` / `renderRanking` / `renderMatrix` |
| 処理の直列化 | 非同期の更新は `chain` にキューして直列化する（`schedule()`、`clearAll`） |

## 注意点

- 点を消すときは必ず `disposeItem()` を通す。
  - CSS2DObject を mesh の子にしているため、`scene.remove(mesh)` だけではラベルの DOM 要素が残る（過去のバグ）。
- 3D 座標は PCA による近似。正確な値はランキングとマトリクスに出す。この区別を UI 上で崩さない。
- 動作確認はブラウザで行う。構文チェックだけなら、モジュール部分を抜き出して `node --check` にかける。
