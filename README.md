# Kokoro OS tools

**Decide → build → check, from your terminal.** Eight small command-line tools from [Kokoro OS](https://usekokoro.com) for people who build apps with an AI coding tool (Claude Code and others).

Nothing to install — run them with `npx`. Free, no sign-up. The tools that use AI run on **your own Anthropic API key**; the rest use no AI at all.

[日本語は下にあります](#日本語)

## The flow

```
  kokoro-baton  ──▶  your AI builds it  ──▶  kokoro-pipeline after  ──▶  kokoro-tuner
  decide first        (Claude Code etc.)      tables, specs, checks        tune the look
```

1. **Decide before you build.** `npx kokoro-baton "what you want to make"` asks what is still undecided — as choices with consequences — and writes `kokoro-spec.md`.
2. **Hand the spec to your AI coding tool** and let it build.
3. **Check what was built.** `npx kokoro-pipeline after` runs the follow-up tools in one go. It tells you the cost and asks before any step that uses AI.
4. **Tune the look by hand** with `npx kokoro-tuner`, and keep your own design laws in one file.

## The tools

| Tool | What it does | AI |
|---|---|---|
| [kokoro-baton](https://www.npmjs.com/package/kokoro-baton) | Answer a few choice-based questions and get a spec file (`kokoro-spec.md`) for your AI coding tool. `gaps` checks a spec you already have. | your key |
| [kokoro-aegis](https://www.npmjs.com/package/kokoro-aegis) | Find the holes in AI-made apps before you ship: static checks plus a hand-check table from your code and spec. Your code never leaves your machine. | none |
| [kokoro-pipeline](https://www.npmjs.com/package/kokoro-pipeline) | After the build, run the follow-up tools in one go, and see which documents are out of date (`status`). | asks first |
| [kokoro-polish](https://www.npmjs.com/package/kokoro-polish) | Claude Fable reads the built app against your spec and writes `kokoro-fixes.md`: the holes, heaviest first, with how to fix them. | your key |
| [kokoro-tuner](https://www.npmjs.com/package/kokoro-tuner) | Tune colors, sizes, spacing and radii with sliders, live in your own dev server. | none (law candidates: your key) |
| [kokoro-writer](https://www.npmjs.com/package/kokoro-writer) | Turn rough notes into a finished document (spec, minutes, manual, report…) in Markdown. | your key |
| [kokoro-kami](https://www.npmjs.com/package/kokoro-kami) | Make a table from one sentence (Markdown + CSV). `params` turns the tunable numbers in your code into an Excel sheet and back. | your key (`params`: none) |
| [kokoro-slide](https://www.npmjs.com/package/kokoro-slide) | Turn what you want to get across into a 6-slide talk (Markdown for Marp) with speaker notes. | your key |

Each tool's page on npm has the full guide.

## Try it

```bash
npx kokoro-baton "A flashcard app: English on the front, meaning on the back"
```

- **Needs Node.js 22 or later.**
- **For the AI tools, set `ANTHROPIC_API_KEY`** in your system's environment variables once. Do not paste your key into an AI chat — it stays in the conversation history.
- **English or Japanese** — the tools follow your system language. Force one with `--lang en` or `--lang ja`.

## Files each tool reads and writes

Everything is plain files in your project folder. The documents are Markdown, so they diff and review like code.

| Command | Reads | Writes |
|---|---|---|
| `npx kokoro-baton "idea"` | your answers (and `~/.kokoro/laws.md`, if you keep design laws) | `kokoro-spec.md` |
| `npx kokoro-baton gaps <file>` | a spec you already have | report in the terminal |
| `npx kokoro-aegis . --out aegis-report.md` | `.js .ts .jsx .tsx .html .vue .svelte` (skips `node_modules`, `dist`, `build`, `.next`), `kokoro-spec.md` | `aegis-report.md` |
| `npx kokoro-polish` | `check.html` (an unminified copy of the app), `kokoro-spec.md` | `kokoro-fixes.md` |
| `npx kokoro-kami params <file>` | the `PARAMS` block in your code | `<name> params.xlsx` (`--apply` writes edited numbers back into `PARAMS`) |
| `npx kokoro-writer spec-doc` | `kokoro-spec.md`, `PARAMS` | `spec-for-readers.md` |
| `npx kokoro-writer system-docs` | your code, `kokoro-spec.md` | `system-specs/1_<name>.md`, … |
| `npx kokoro-tuner .` | the `STYLE` block in your code | the same `STYLE` values, edited in place |
| `npx kokoro-pipeline after .` | the project folder | runs the steps above in order |
| `npx kokoro-pipeline status .` | the documents above | which documents are out of date |

**Re-running never deletes.** Before `kokoro-pipeline` rewrites a document, it moves the old one to `previous/<date_time>/`. The latest always has the same name.

**"Out of date" is decided by content, not timestamps.** Each document starts with a small marker holding fingerprints of the spec and code it was made from; `status` compares them with the current files.

## What leaves your machine

| Tool | Sends | To |
|---|---|---|
| kokoro-aegis | **nothing** (if you have API keys set, it only asks each provider for its list of model names; `--offline` turns that off) | — |
| kokoro-kami `params`, kokoro-tuner (sliders), kokoro-pipeline `status` | **nothing** | — |
| kokoro-baton, kokoro-writer, kokoro-kami, kokoro-slide, kokoro-polish, kokoro-tuner `candidates` | your text, spec or code, with your API key | Kokoro OS (`usekokoro.com/api/v1/…`), which calls Anthropic with your key |

- **Kokoro OS does not store your text, code or key.** It holds the prompts on its side, so the tools stay small; the request passes through and the answer comes back.
- For rate limits, it keeps only a short hash of your key (or of your IP, when there is no key). Those rows are deleted after 2 days.
- AI costs are billed by Anthropic to **your** key. Kokoro OS charges nothing.

## Using it with Claude Code (or another AI coding tool)

Most commands can be run by your AI. Paste these as they are:

```
Run npx kokoro-aegis . --out aegis-report.md in this folder and show me the table.
```

```
Run npx kokoro-baton gaps kokoro-spec.md in this folder and show me which points are not answered.
```

```
Run npx kokoro-pipeline after . --only params,aegis --yes in this folder. These steps use no AI.
```

```
Before you run anything that uses AI (kokoro-polish, kokoro-writer, or the AI steps of kokoro-pipeline), tell me the cost and wait for my yes.
```

**Exception: the questions of `kokoro-baton "idea"`.** They are for you to answer — an AI running it would answer them itself. Run that one yourself.

`kokoro-spec.md` already ends with an "After building" section: when your AI finishes, it runs the no-AI checks on its own and asks you before the AI steps.

## Costs and limits

Measured on a small sample game in October 2026. Billed to your Anthropic key.

| Step | Cost |
|---|---|
| kokoro-baton (one round of questions) | under 1¢ |
| kokoro-aegis, `kami params`, kokoro-tuner sliders | free (no AI) |
| `kokoro-pipeline after .`, all five steps | about $2–2.70 in total |
| kokoro-polish alone | about $0.70–1 per run |

- The costly steps (kokoro-polish, `kokoro-writer system-docs`, and the AI steps of `kokoro-pipeline`) say what they will send and roughly what it will cost, then ask — unless you pass `--yes`.
- Rate limits on the Kokoro OS side: for example, kokoro-polish runs up to 6 times per hour per key.

## Guides (Japanese)

- [Basic: Baton → AI → Aegis](https://www.northernsadnessproductions.com/kokoro-os/npx-basic)
- [All eight tools in one flow, with a sample project to download](https://www.northernsadnessproductions.com/kokoro-os/npx-pipeline)

## About this repository

This repository is the signboard for the tools. **The source code is not published here** — the tools are distributed on npm.

Want to do this on screen instead of in a terminal? Kokoro OS has Baton, Builder and Aegis as web apps: [usekokoro.com](https://usekokoro.com)

## License

[PolyForm Shield 1.0.0](LICENSE) — free to use, including at work. You may not use the tools to build a product that competes with them.

---

## 日本語

**決める → 作る → 確かめる を、ターミナルから。** [Kokoro OS](https://usekokoro.com) の、AI（Claude Code など）にアプリを作らせる人のための小さな道具 8 本です。

入れる手間はありません。`npx` で動きます。無料・登録なし。AI を使う道具は **あなたの Anthropic の API キー** で動き、ほかの道具は AI を使いません。

### 流れ

1. **作る前に決める。** `npx kokoro-baton "作りたいもの"` が、まだ決まっていないことを「選ぶと、こうなる」の選択肢で聞き、`kokoro-spec.md` を書きます。
2. **仕様を AI に渡して作ってもらう。**
3. **作ったものを確かめる。** `npx kokoro-pipeline after` が、あとの道具をまとめて動かします。AI を使う段は、費用を言ってから聞きます。
4. **見た目を手で整える。** `npx kokoro-tuner` で、自分のデザインの法則も 1 つのファイルに残せます。

### 道具

| 道具 | すること | AI |
|---|---|---|
| [kokoro-baton](https://www.npmjs.com/package/kokoro-baton) | 選ぶ形の問いに答えて、AI に渡す仕様のファイル（`kokoro-spec.md`）を作る。`gaps` は手持ちの仕様の抜けを調べる | あなたのキー |
| [kokoro-aegis](https://www.npmjs.com/package/kokoro-aegis) | AI が作ったアプリの穴を、出す前に探す（読む検査＋コードと仕様から作る、手で確かめる表）。コードはパソコンの外に出ない | なし |
| [kokoro-pipeline](https://www.npmjs.com/package/kokoro-pipeline) | 作ったあとの道具をまとめて動かす。どの紙が古くなったかも見られる（`status`） | 先に聞く |
| [kokoro-polish](https://www.npmjs.com/package/kokoro-polish) | Claude Fable が、作ったアプリを仕様と照らして読み、穴を重い順に、直し方つきで `kokoro-fixes.md` に書く | あなたのキー |
| [kokoro-tuner](https://www.npmjs.com/package/kokoro-tuner) | 色・大きさ・余白・角の丸みを、スライダーで、自分の開発サーバーの上で動かして整える | なし（法則の候補はあなたのキー） |
| [kokoro-writer](https://www.npmjs.com/package/kokoro-writer) | 走り書きのメモを、仕上がった文書（仕様書・議事録・説明書・報告書…）にする | あなたのキー |
| [kokoro-kami](https://www.npmjs.com/package/kokoro-kami) | 一文から表を作る（Markdown と CSV）。`params` はコードの中の調整する数を Excel の表にし、直した数を書き戻す | あなたのキー（`params` はなし） |
| [kokoro-slide](https://www.npmjs.com/package/kokoro-slide) | 伝えたいことを、話す人のメモつきの 6 枚のスライド（Marp で開ける Markdown）にする | あなたのキー |

使い方の詳しい説明は、npm の各道具のページにあります。

### 試す

```bash
npx kokoro-baton "単語カード：表に英語、裏に意味"
```

- **Node.js 22 以上が要ります。**
- **AI を使う道具は、`ANTHROPIC_API_KEY` をパソコンの環境変数に一度入れておきます。** キーを AI との会話に貼らないでください（会話の記録に残ります）。
- **日本語と英語** に対応しています。パソコンの言語に合わせます。`--lang ja` / `--lang en` で決めることもできます。

### 詳しい使い方

- [基本の使い方 ― Baton → AI → Aegis](https://www.northernsadnessproductions.com/kokoro-os/npx-basic)
- [道具 8 本をつないだ流れ（作例「のこれ！」のダウンロードつき）](https://www.northernsadnessproductions.com/kokoro-os/npx-pipeline)

読むファイル・書くファイル、パソコンの外に出るもの、AI への頼み方、費用の目安は、上の英語の節にまとめてあります。要点は次のとおりです。

- **Aegis・パラメータ表・Tuner のスライダー・status は、どこにも送りません。**
- ほかの道具は、文章やコードをあなたのキーと一緒に Kokoro OS に送り、Kokoro OS があなたのキーで Anthropic を呼びます。**文章・コード・キーは保存しません。** 回数の上限のために、キー（無ければ IP）の短い写しだけを残し、2 日で消します。
- 費用のかかる段（研磨・仕組みの仕様書・pipeline の AI の段）は、動かす前に費用を言って聞きます。

### このリポジトリについて

ここは道具の看板です。**ソースコードは置いていません。** 道具は npm から配っています。

ターミナルでなく画面で使いたいときは、Kokoro OS の Baton・Builder・Aegis があります：[usekokoro.com](https://usekokoro.com)

### ライセンス

[PolyForm Shield 1.0.0](LICENSE)。仕事も含めて無料で使えます。この道具と競合する製品を作るためには使えません。
