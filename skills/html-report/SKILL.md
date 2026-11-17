---
name: html-report
description: Explicit-only skill. 人が読んで判断する文書（提案・設計の選択肢・調査結果など）を、図と操作が主の単一 HTML ページにする。読ませずに、触って、選んで判断できる形にする。原本の Markdown は残し、HTML は派生物として扱う。ユーザーが「/html-report」「html-report を使って」のように明示的に指定した時のみ使用する。「HTML にして」「見やすくして」と言われただけでは自動起動しない。記録の原本、装飾だけの静的資料、数値ダッシュボード、文章中心のページには使わない。
---

# html-report

人が判断する文書を、**読まずに、触って、選んで**決められる単一 HTML にする。主役は図と操作で、文章は図の注釈。文章は、図にも操作にもならない節だけに使う。

## 縛り（先に置く）

- 原本は Markdown のまま残す。HTML は派生物。
- 図と操作が主。文章は注釈。
- 例示値は **「例」と明示すれば**、原本に無い具体値を置いてよい。原本に無い**主張・結論・推奨**は足さない。

## 手順

### 1. 原本から取り出す

問い／選択肢と評価基準／状態・数量・時間の規則／構成要素の関係／推奨と根拠（あれば）／未確認事項。

### 2. 節ごとに「どの形か」を先に書き出す

| 原本の内容 | 表現 |
|---|---|
| 状態・数量・時間の規則 | **動くモデル**（値・操作・ログ） |
| 選択肢 | **切り替わる図** + 評価表（推奨が原本にあればそれを強調） |
| 構成要素の関係（4 つ以上） | 図 |
| どれにも当たらない | 文章（この節だけ） |

- 原本に推奨が無い: 推奨欄は出さず、「原本に推奨の記載なし」と 1 行だけ書く。
- 移行段階は時間の規則として動くモデルに入れる（段階を進める操作と「次に進む条件」）。失敗モードは失敗イベントとして動くモデルのログに出す。どの形にも当たらない時だけ文章にする。

### 3. ページ構成

最初の画面は図か操作（brief は 1〜2 文）→ 問いごとのブロック → 文章の節 → 未確認事項 → 答えの欄 → フッター（原本のパスと「記述の正は原本」）。

### 4. 図の作り方

図は HTML/CSS の箱（flex/grid、折り返しあり）。矢印は `↓ →` の文字。SVG は使わない。

- 箱は `.node`（枠 1.5px、角丸 8）。種類は枠線で区別し、文言も併記する: 新規 = 実線太、既存 = 実線細、無し・廃止 = 点線。
- 色（`--ok --warn --bad --accent`）は記号（✓ ! ✕）とセットで使う。

```html
<style>
  :root { --bg:#fff; --fg:#1c2024; --muted:#60646c; --line:#c9c9d0; --accent:#0d74ce; --ok:#18794e; --warn:#ad5700; --bad:#cd2b31; }
  @media (prefers-color-scheme: dark) {
    :root { --bg:#111113; --fg:#edeef0; --muted:#b0b4ba; --line:#4a4f52; --accent:#70b8ff; --ok:#3dd68c; --warn:#ffa057; --bad:#ff9592; }
  }
  body { background:var(--bg); color:var(--fg); max-width:48rem; margin:0 auto; padding:0 16px; }
  .row { display:flex; flex-wrap:wrap; gap:8px; align-items:center; }
  .node { border:1.5px solid var(--line); border-radius:8px; padding:6px 10px; }
  .node.new { border:3px solid var(--fg); }
  .node.none { border-style:dotted; color:var(--muted); }
  .tbl { overflow-x:auto; }
</style>
```

### 5. 動くモデルの作り方

状態（例示値を 1 オブジェクト）、操作（時間を進める・イベントのボタン）、`apply(state, event)`、ログ、不変条件の表示、境界ケースへ跳ぶボタン。

- **不変条件・境界ケースは原本の規則から導けるものだけ。** 導けないものは「例」と明示する。
- 規則に反する操作はボタンを `disabled` にして理由を表示する。
- 例示値（総数・期間など原本に無い値）は画面上で「例」と明示する。

```html
<output id="st"></output><ol id="log"></ol>
<button data-ev="tick">時間を進める</button> <button data-ev="ok">成功</button>
<script>
let s = { stock: 3 /* 例 */, t: 0 }, log = [];
const rules = { tick: s => ({ ...s, t: s.t + 1 }), ok: s => ({ ...s, stock: s.stock - 1 }) };
function apply(ev) {
  s = rules[ev](s); log.push(ev + " → " + JSON.stringify(s)); render();
}
function render() {
  st.textContent = JSON.stringify(s);               // 値
  document.getElementById("log").innerHTML = log.map(l => "<li>" + l + "</li>").join("");
  // 不変条件（例: 在庫 = 総数 − 確定 − 仮押さえ）をここで表示し、違反する操作は disabled にする
}
document.querySelectorAll("[data-ev]").forEach(b => b.onclick = () => apply(b.dataset.ev));
render();
</script>
```

### 6. 切り替わる図の作り方

選択肢ごとにノードの状態（新規 / 変更 / 既存 / 無し）をデータで持ち、ラジオで class と文言を入れ替える。評価表の該当列と推奨（あれば）を同じ画面で強調する。

```html
<label><input type="radio" name="opt" value="A" checked> A</label>
<label><input type="radio" name="opt" value="B"> B</label>
<div class="row" id="fig"></div>
<script>
const opts = { A: { api: "既存", queue: "新規" }, B: { api: "既存", queue: "無し" } };
const cls = { 新規: "new", 既存: "", 無し: "none" };
function draw() {
  const v = document.querySelector("[name=opt]:checked").value;
  fig.innerHTML = Object.entries(opts[v]).map(([k, st]) =>
    `<span class="node ${cls[st]}">${k}（${st}）</span>`).join(" → ");
  document.querySelectorAll("[data-col]").forEach(c => c.classList.toggle("on", c.dataset.col === v));
}
document.querySelectorAll("[name=opt]").forEach(r => r.onchange = draw);
draw();
</script>
```

### 7. 答えを残す

問いごとにラジオ + 要望 `<textarea>`。**「テキストをコピー」ボタンは常に置く**（Markdown を生成）。

```js
async function copyText(text, area) {
  try { await navigator.clipboard.writeText(text); return; } catch {}
  area.value = text; area.select();
  if (!document.execCommand("copy")) area.focus(); // 失敗時は選択済みのまま手動コピー
}
```

Claude Code で Artifact 公開が使えるなら公開してよい。`artifact-capabilities` の保存機能が使えればそれで残してよい。コピーは公開版にも残す。

### 8. 技術の縛り

- 単一 HTML。外部依存なし（CDN・外部フォント・外部画像を読まない）。
- `prefers-color-scheme` でライト / ダーク。色は CSS 変数。
- 幅 400px で横スクロール無し。表は `.tbl`（`overflow-x:auto`）で包む。
- 色だけで意味を伝えない（記号か文言を併記）。
- JS が無効でも本文が読める。

### 9. 確認

- Playwright が使えれば viewport 幅 400 と `colorScheme: 'dark'` を直接指定して見る。
- 使えない headless Chrome（約 500px 未満のウィンドウを作れない）は、幅 400px の `<iframe srcdoc>` 内で `documentElement` の `scrollWidth <= clientWidth` を見る。
- 操作を一通り触る（モデルが動く、ラジオで図が変わる、コピーが生成される）。
- 原本照合: 例示値以外の主張が原本にある。

## やらないこと

- 原本の代替にする（HTML だけを残して Markdown を捨てる）
- 原本に無い主張・結論・推奨を足す
- 装飾だけの部品
- 数値ダッシュボード
- 文章中心のページ
