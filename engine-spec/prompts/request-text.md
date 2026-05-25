# Opening User Message — `buildRequestText`

The opening turn of every conversation is a single user message. When an
image is present, it is the **first** content block of that message and the
text described here is the **second** block. When no image is present, the
text is the only content block.

## Two templates

There are two templates: one for the original Figma-style modes (`OVERVIEW`,
`SCREEN`, `ELEMENT`, and `topic` follow-ups when the prior conversation has
been evicted from the LRU), and one for the new `SCREENSHOT` mode.

Both end with the same image-attached sentinel sentence so the model's
attention bridge between text and image stays consistent.

### Figma node template (verbatim from `server/describe.js`)

```text
Kind of description: <KIND>

Node name: "<node.name>"
Node role: <node.role>

Structure:
<outline>

<image-sentinel>
```

Where:

- `<KIND>` is the uppercased mode name (`OVERVIEW`, `SCREEN`, `ELEMENT`) or
  the topic kind (`COLOR`, `LAYOUT`, `IMAGERY`, `TEXT`) when `mode === 'topic'`.
- `<outline>` is the shallow text outline of the node and the first two
  levels of its children, indented two spaces per level, each line prefixed
  with `- `. Depth is capped at **2 levels of children deep**. Example:

  ```text
  - Account screen [screen]
    - Header [group]
      - Avatar [image]
      - Title [text]
    - Settings list [group]
      - Email row [group]
      - Password row [group]
  ```

- `<image-sentinel>` is one of:
  - `A screenshot of this node is attached above.` — when the image content
    block is present
  - `No screenshot is available for this node; describe it from the structure and names only.` — when no image

### SCREENSHOT template (new in v0.1.0)

For arbitrary captured images with no surrounding tree:

```text
Kind of description: SCREENSHOT

<context-line>

A screenshot is attached above.
```

Where `<context-line>` is a single sentence the host application MAY provide
to ground the model — e.g. the source application name, the window title, or
the user's note. If the host has nothing to add, omit the line entirely
(remove its blank-line separator as well). Examples:

```text
Kind of description: SCREENSHOT

Captured from: Slack, channel #design-review

A screenshot is attached above.
```

```text
Kind of description: SCREENSHOT

A screenshot is attached above.
```

The image content block is **mandatory** for SCREENSHOT requests. An engine
implementation MUST reject `SCREENSHOT` mode without an image with
`reason: 'no-image'` (an eighth reason string reserved for this case).

## Why the templates look the way they do

- **`Kind of description:` is the first line** so the model latches onto the
  expected response shape before reading anything else.
- **Quoting `node.name`** prevents the model from mistaking a name like
  `Settings` for an instruction.
- **The outline is shallow** because deeper trees blow past `max_tokens` and
  give the model too much to summarize at once.
- **The image-sentinel sentence is the last line** so the model's attention
  bridges from text into the attached image without an intervening token.
