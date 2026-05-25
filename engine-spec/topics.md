# Topics

A **topic** picks which deep-dive angle the engine takes on a follow-up turn.
Topics are encoded as the URL parameter `?topic=color` (or `layout`, etc.) in
the web reference impl, and as a `DescribeTopic` enum value in the macOS
implementation.

Topics correspond one-to-one with KIND tokens in the system prompt:

| Topic     | KIND token | Maps to       | Notes                                                        |
|-----------|------------|---------------|--------------------------------------------------------------|
| `color`   | `COLOR`    | both surfaces | Color palette + accessibility focus on color-only meaning.   |
| `layout`  | `LAYOUT`   | both surfaces | Spatial hierarchy, eye-flow vs. reading order.                |
| `imagery` | `IMAGERY`  | both surfaces | Images, icons, illustrations and whether they carry info.    |
| `text`    | `TEXT`     | macOS v1; web later | Text content in captured images; summarize or quote.   |

The topic value is lowercase on the wire (URL, JSON), uppercase in the prompt
(the `Kind of description:` line and the `<KIND>` substitution in
`topic-followup.md`).

## Adding a new topic

Same process as adding a mode (see `modes.md`):

1. PR against this folder with justification.
2. Append a block to `prompts/system-prompt.md` describing the topic's focus.
3. Add a row to the table above.
4. Bump the minor version in `VERSION`, document in `CHANGELOG.md`.
5. Update at least one reference implementation.

Topics are intentionally orthogonal — each one focuses on a single angle. If
you find yourself adding a topic that overlaps two existing ones, the right
fix is usually to refine the existing topics' boundaries instead.
