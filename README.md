# gh-sibling

Find **sibling-leftover** bugs in a repo's recent merged PRs.

A sibling-leftover is the second half of a symmetric fix that nobody made. Someone
escapes user input in `encode.ts` but forgets `decode.ts`. Someone adds a string to
`en.json` and never touches `ja.json`. Someone fixes the `left` handler and leaves
`right` broken. The PR merges green, and the twin bug sits there waiting.

These are some of the easiest first contributions in open source: the fix already
exists, you just mirror it onto the side they missed. On 2026-06-26 I opened 28 OSS
PRs this exact way. This tool is the part I was doing by eye, made repeatable.

## Use

```sh
node gh-sibling.mjs <owner/repo>
```

It reads the last 25 merged PRs (via your already-authed `gh` CLI), scores each for
how likely it left a mirror un-fixed, and prints them highest-first with the reason:

```
sibling-leftover candidates — nocodb/nocodb

#14140 [3] fix(nc-gui): show date examples in date format dropdown
  by rameshmane7218 · DateOptions.vue, DateTimeOptions.vue, lang/en.json
  · bugfix verb: "fix"
  · focused (3 code files)
```

That `en.json` with no other locale touched is the tell. Open the PR, find the side
they missed, send the mirror.

```sh
node gh-sibling.mjs <owner/repo> --json   # machine-readable, for scripting a queue
node gh-sibling.mjs --selftest            # offline check of the scoring, no gh needed
```

## What it scores on

- symmetry language in the title (`also`, `same for`, `mirror`, `both`, `as well`)
- a bugfix verb (`fix`, `guard`, `escape`, `check`, `validate`) — the kind of change
  that usually needs mirroring
- touching one side of a known opposite pair (`encode`/`decode`, `enable`/`disable`,
  `subscribe`/`unsubscribe`, `show`/`hide`, and more) but not the other
- a bugfix that touched no test
- a small, focused PR (1 to 3 files), which is where a forgotten twin hides
- merged by `web-flow` (the maintainer took it verbatim, so the gap shipped as-is)

None of it is proof. It is a ranked list of where to look first, so you read 5 PRs
instead of 250.

## Requirements

- Node (built-in modules only, zero dependencies)
- [`gh`](https://cli.github.com/) installed and authed (`gh auth status`)

## Privacy

Sends nothing anywhere. The only network calls are the `gh pr list` requests `gh`
itself makes to GitHub. No telemetry, no account, no server.

## License

MIT
