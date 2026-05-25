# Topic Follow-up — `buildTopicFollowup`

When a user asks for a deep-dive on a source they have already had described,
the engine MUST continue the existing conversation rather than start a new
one. The image and the prior description are already in the conversation
history; re-sending them would waste tokens and risk drift from the prior
turn.

The follow-up is sent as a single user message with one text content block.
No image. No structure block.

## Verbatim template

```text
Now give the <KIND> description of this same item, following the <KIND> instructions above. Build on what you already described — go deeper on this angle, and do not repeat what you have already said.
```

Where `<KIND>` is substituted twice with the topic kind in uppercase:
`COLOR`, `LAYOUT`, `IMAGERY`, or `TEXT`.

Example for `LAYOUT`:

```text
Now give the LAYOUT description of this same item, following the LAYOUT instructions above. Build on what you already described — go deeper on this angle, and do not repeat what you have already said.
```

## Fallback when the conversation has been evicted

The per-source LRU is bounded at 25 entries. If the user requests a topic
follow-up for a source that has been evicted from the LRU (or the engine has
restarted), the implementation MUST fall back to the opening template from
`request-text.md`, with the `topic` mode. The image is re-attached. This is
what `server/describe.js` does today.
