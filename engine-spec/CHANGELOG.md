# Engine Spec Changelog

All notable changes to the description engine spec are documented here.
The spec follows [Semantic Versioning](https://semver.org/).

## [0.1.0] — 2026-05-23

Initial extraction of the spec from `server/describe.js` as it stood on
[`f85310f`](https://github.com/ryandbrady/throughline-web/commits/main)
(the `main` branch tip at extraction time).

### What this version documents

- The verbatim `SYSTEM_PROMPT` shipped in `describe.js`
- The `buildRequestText` and `buildTopicFollowup` template format
- The four mode names used by the reference impl: `OVERVIEW`, `SCREEN`,
  `ELEMENT`, plus the `topic` meta-mode (which maps to `COLOR`, `LAYOUT`, or
  `IMAGERY` via `TOPIC_KINDS`)
- The Anthropic Messages API call shape: `model`, `max_tokens: 1024`,
  `thinking: { type: 'adaptive' }`, `output_config: { effort: 'medium' }`,
  `cache_control: { type: 'ephemeral' }` on the system prompt
- Base64 PNG only as the image source, 5 MB cap (enforced in
  `server/index.js`, not `describe.js`)
- The per-source conversation LRU (25 entries), with assistant content stored
  verbatim including thinking blocks
- The six error reason strings: `no-api-key`, `bad-api-key`, `rate-limited`,
  `connection`, `api-error`, `empty-response`

### Forward-compatible additions (NOT yet implemented in the web reference impl)

These are part of the spec because the macOS desktop app implements them as
its v1 surface. The web tool may adopt them later; until it does, web
behavior is unchanged.

- **`SCREENSHOT` primary mode** — for describing an arbitrary captured image
  with no surrounding design-system context. See `modes.md`.
- **`TEXT` deep-dive topic** — for focusing on text content within a captured
  image (charts, dashboards, messages). See `topics.md`.

The `request.schema.json` accepts both, and the `request-text.md` template has
a `SCREENSHOT`-specific variant. A spec-conformant engine MUST accept these
two values; a non-conformant engine that only implements the original four
modes / three topics MAY reject them with `reason: 'unsupported-mode'` (a
seventh reason string reserved for this purpose).
