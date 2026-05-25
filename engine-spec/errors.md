# Errors

The engine returns one of two shapes:

```jsonc
{ "ok": true,  "mode": "screenshot", "topic": null, "description": "..." }
{ "ok": false, "reason": "<reason-string>" }
```

The `reason` strings are stable identifiers — UI implementations key off them
to show human-readable error messages and to choose the right remediation
(prompt for a new API key, suggest waiting and retrying, etc.).

## Reason strings

| Reason              | When                                                                                | Suggested user-facing copy                                                            |
|---------------------|-------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| `no-api-key`        | No API key was supplied and no `ANTHROPIC_API_KEY` env fallback exists.            | "Add your Anthropic API key in Settings to start describing screenshots."             |
| `bad-api-key`       | The Anthropic API rejected the key with 401/403 (`AuthenticationError`).            | "Your API key was rejected. Open Settings to update it."                              |
| `rate-limited`      | The Anthropic API returned a rate-limit error (`RateLimitError`).                    | "Anthropic is throttling requests. Try again in a moment."                            |
| `connection`        | A network-level failure reached the SDK (`APIConnectionError`).                      | "Couldn't reach Anthropic. Check your connection and try again."                      |
| `api-error`         | Any other SDK-thrown error.                                                          | "Something went wrong describing this image. Try again, or check the diagnostics log."|
| `empty-response`    | The model returned no text content (e.g. content filter or model returned only thinking blocks). | "The model didn't return a description. Try capturing the image again."   |

## Reserved reasons (v0.1.0 additions)

These two reasons are reserved for the SCREENSHOT/TEXT extensions. An engine
implementation that adds either extension MUST use these reason strings
rather than inventing new ones:

| Reason              | When                                                                  |
|---------------------|-----------------------------------------------------------------------|
| `unsupported-mode`  | The caller requested a mode (or topic) the engine doesn't implement. |
| `no-image`          | `SCREENSHOT` mode was requested without an image content block.       |

## Logging

Implementations SHOULD log every error with at least:

- the reason string
- the HTTP status (when applicable)
- the SDK error class name (when applicable)
- the error message

Implementations MUST NOT log the API key, the image bytes, the system
prompt, or the description text in error logs. The image bytes in particular
are the user's data — not the engine's to retain past the call.
