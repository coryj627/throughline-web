# Modes

A **mode** picks which kind of primary description the engine produces on the
opening turn of a conversation. Modes are encoded in the `Kind of description:`
line at the top of the opening user message.

| Mode         | When to use                                                      | Source of context              |
|--------------|------------------------------------------------------------------|--------------------------------|
| `OVERVIEW`   | A whole page in a multi-screen design.                           | Figma node tree                |
| `SCREEN`     | One top-level screen or frame.                                    | Figma node tree                |
| `ELEMENT`    | A smaller element nested inside a screen.                        | Figma node tree                |
| `SCREENSHOT` | An arbitrary captured image with no surrounding design context. | The pixels alone (+ optional one-line user-provided context) |

`topic` is **not a mode** — it is a sentinel passed to `describeNode` to
indicate that this call is a deep-dive follow-up. The actual prompt kind for
a topic call is the topic name in uppercase (`COLOR`, `LAYOUT`, `IMAGERY`,
`TEXT`). See `topics.md`.

## Selecting a mode

Mode selection happens in the **caller**, not the engine. Two reference
strategies:

### Figma caller (existing — `server/index.js`)

Depth-based, where depth is the node's position in the accessibility tree:

```js
const mode = topic
  ? 'topic'
  : found.depth === 1 ? 'overview'
  : found.depth === 2 ? 'screen'
  : 'element';
```

### Desktop screenshot caller (new)

Always `SCREENSHOT` for the primary description; `topic` for follow-ups.

```swift
let mode: DescribeMode = topic == nil ? .screenshot : .topic
```

## Adding a new mode

1. Open a PR against this folder. Justify the new mode — what context it
   captures that no existing mode does.
2. Add a block to `prompts/system-prompt.md` under "Forward-compatible
   additions" with the same shape as `SCREENSHOT`.
3. Add a row above with selection guidance for callers.
4. Bump the minor version in `VERSION` and document the change in
   `CHANGELOG.md`.
5. Update at least one reference implementation.
