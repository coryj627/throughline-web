# Throughline Engine Specification

This folder is the **canonical contract** for Throughline's description engine —
the prompts, the API call shape, the conversation model, and the error
vocabulary that every implementation of the engine must honor.

It exists so that the engine can be re-implemented faithfully in other
languages and runtimes — starting with the macOS desktop app — without each
implementation guessing at what the others do.

The reference implementation lives at `../server/describe.js`. This spec
describes that file plus two forward-compatible extensions (the `SCREENSHOT`
primary mode and the `TEXT` deep-dive topic) which the macOS app is the first
to implement and which the web tool may adopt later.

## Layout

```
engine-spec/
├── VERSION                       semver, source of truth for spec compatibility
├── CHANGELOG.md                  human-readable history of spec changes
├── README.md                     this file
├── prompts/
│   ├── system-prompt.md          the verbatim system prompt
│   ├── request-text.md           the opening user-message template
│   └── topic-followup.md         the "tell me more" continuation template
├── schema/
│   ├── node.schema.json          accessibility tree node shape
│   ├── request.schema.json       describeNode() input
│   └── response.schema.json      describeNode() output
├── modes.md                      OVERVIEW / SCREEN / ELEMENT / SCREENSHOT
├── topics.md                     COLOR / LAYOUT / IMAGERY / TEXT
├── api-call.md                   exact Anthropic Messages API call shape
└── errors.md                     the six reason strings
```

## How implementations stay in sync

1. **Bump `VERSION`** whenever any prompt text, schema field, or API call
   parameter changes. Treat the system prompt as if it were a public API — even
   whitespace changes break Anthropic's prompt cache and force re-billing of
   the prefix tokens, so changes are not free.

2. **Quote prompt text byte-for-byte** in your implementation. The throughline
   macOS app ships a `PromptParityTests` test that hashes its embedded system
   prompt against `prompts/system-prompt.md` and fails the build on drift.
   Implementations in other languages are strongly encouraged to do the same.

3. **Open a PR against this spec first** when you want to add a mode, topic,
   schema field, or API parameter. Once the spec lands, port to the
   implementations.

## What is NOT in this spec

The reference implementation also contains Figma-specific glue —
`build-a11y-tree.js`, `figma-images.js`, `figma-comments.js`, the WebSocket
relay in `index.js`, the Figma plugin in `figma-bridge/`. None of that is part
of the engine. Each implementation is free to produce the `node` argument and
the PNG screenshot however it wants — Figma in the web case, ScreenCaptureKit
in the macOS case, something else later.

## License

MIT — same as the rest of the repository.
