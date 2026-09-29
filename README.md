# doc-skills

文書づくりのための agent skill 集。Claude Code と Codex で使える。

## 何ができるか

| skill | 内容 |
|---|---|
| `svg-figure` | 文書に載せる説明図を `figures/<name>.svg` に手座標の SVG で描き、必要なら PNG も書き出す。md・html・docx のどこからでも同じ図を参照できる。「/svg-figure」「svg-figure を使って」と明示したときだけ動く |

## install

```sh
npx skills add nnno/doc-skills -g -a claude-code -a codex -y
```

更新:

```sh
npx skills update -g
```

### Claude Code の plugin として入れる

```sh
claude plugin marketplace add nnno/doc-skills
claude plugin install doc-skills@doc-skills
```

Claude Code のセッション内なら `/plugin marketplace add nnno/doc-skills` → `/plugin install doc-skills@doc-skills` でも同じ。plugin として入れた場合、skill は `/doc-skills:svg-figure` で呼ぶ。

## 案件ごとの色表を指定する

`svg-figure` は既定の配色を持っている。案件のブランド色に合わせたいときは、そのプロジェクトの `AGENTS.md`（または `CLAUDE.md`）に色表のファイルを指定する。

```markdown
<!-- AGENTS.md -->
svg-figure の色表は `docs/brand-colors.md` を使う。
```

色表のファイルには、構造の 4 スロットだけを置く。

```markdown
<!-- docs/brand-colors.md -->
| スロット | 値 |
|---|---|
| 主線 | `#0d74ce` |
| 強調枠 | `#113264` |
| 淡い塗り | `#e6f4fe` |
| 面 | `#f9f9fb` |
```

状態色（成功・注意・危険）と文字色は、意味を保つため skill の既定値のまま使う。

## PNG 化には Chrome が要る

Google Docs・Keynote・PowerPoint に貼る場合や docx に落とす場合は、SVG を PNG にする。PNG 化は Google Chrome（または Chromium）の headless モードで行うので、Chrome をインストールしておくこと。SVG を描くだけなら不要。

## License

MIT
