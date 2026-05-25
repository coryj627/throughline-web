# Anthropic API Call

The engine makes exactly one call per `describeNode` invocation, to the
Anthropic Messages API. The shape of that call is part of the spec — changing
any of these parameters changes engine behavior and SHOULD be reviewed as a
spec change.

## Endpoint

```
POST https://api.anthropic.com/v1/messages
```

Headers:

| Header              | Value                                                          |
|---------------------|----------------------------------------------------------------|
| `x-api-key`         | the user's Anthropic API key                                   |
| `anthropic-version` | `2023-06-01`                                                    |
| `content-type`      | `application/json`                                              |

`anthropic-version` is the value used by `@anthropic-ai/sdk@^0.98.0` as of
extraction. Implementations SHOULD pin to the version they tested against
rather than tracking "latest".

## Request body

```jsonc
{
  "model": "claude-opus-4-7",              // configurable; env DESCRIBE_MODEL on web
  "max_tokens": 1024,
  "thinking": { "type": "adaptive" },
  "output_config": { "effort": "medium" },
  "system": [
    {
      "type": "text",
      "text": "<verbatim system prompt — see prompts/system-prompt.md>",
      "cache_control": { "type": "ephemeral" }
    }
  ],
  "messages": [ /* see below */ ]
}
```

Notes:

- **`thinking` and `output_config`** are part of the call as shipped. If the
  Anthropic API version you target rejects either field, drop it cleanly
  rather than failing — but record a `connection`-class error to telemetry so
  the spec can be revisited.
- **`cache_control: { type: 'ephemeral' }` on the system prompt** is what
  makes the byte-identical-prompt discipline economically meaningful.
  Implementations MUST include it; omitting it disables caching for the
  prefix.
- **`max_tokens: 1024`** is chosen for the "tight response" instruction in
  the system prompt. Implementations SHOULD NOT raise it without bumping the
  spec version.

## Messages

### Opening turn — primary description

```jsonc
[
  {
    "role": "user",
    "content": [
      // Image is OPTIONAL for OVERVIEW / SCREEN / ELEMENT; REQUIRED for SCREENSHOT.
      {
        "type": "image",
        "source": {
          "type": "base64",
          "media_type": "image/png",
          "data": "<base64-encoded PNG bytes, no data: prefix>"
        }
      },
      {
        "type": "text",
        "text": "<output of buildRequestText — see prompts/request-text.md>"
      }
    ]
  }
]
```

### Follow-up turn — topic deep-dive

The prior `[user, assistant, user, assistant, ...]` history is replayed
**verbatim**, including the assistant's `content` array with any `thinking`
blocks intact. Then a new user message is appended:

```jsonc
{
  "role": "user",
  "content": "<output of buildTopicFollowup — see prompts/topic-followup.md>"
}
```

Note: the content can be a bare string (as in `server/describe.js` today) or
a single-element array `[{ "type": "text", "text": "..." }]` — both are
equivalent per the Messages API. Pick one and stay consistent inside an
implementation.

## Image constraints

- **Media type:** `image/png` only. Implementations MUST normalize other
  formats (JPEG, HEIC, WebP, etc.) to PNG before sending.
- **Size cap:** the base64-encoded data MUST be ≤ 5,000,000 characters
  (~3.75 MB of binary). If the encoded image exceeds this, the implementation
  MUST either downsample the image until it fits, or omit the image and fall
  back to the no-image sentinel sentence (see `request-text.md`). The web
  reference impl drops the image silently; the macOS implementation
  downsamples instead.

## Response

The response is a standard Anthropic Messages API response. The engine reads
the first text content block from `message.content`, trims it, and returns it.
See `errors.md` for empty-response handling.

## Retries

- SDK-level retries: 4 (the `@anthropic-ai/sdk` default is 2; the engine
  bumps it because each retry is cheap and screenshot uploads occasionally
  flake on flaky networks).
- No engine-level retry beyond the SDK.
- Retries do not bypass the system-prompt cache.
