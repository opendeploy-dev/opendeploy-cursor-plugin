# AI API detection and OpenDeploy AI API opt-in

OpenDeploy can provision a project-scoped AI API token when the agent submits
the exact placeholder value in service runtime env:

```text
{{MINIONS_AI_API_KEY}}
```

The public base URL for OpenDeploy AI API is:

```text
https://api.opendeploy.dev/v1
```

## Product contract

- The analyzer emits `ai_config` with:
  - `requires_ai_api_key`
  - `detected_providers`
  - `api_key_vars`
  - `base_url_vars`
- The dashboard's guided deploy defaults detected AI key vars to OpenDeploy AI
  API and sends `{{MINIONS_AI_API_KEY}}`.
- The backend expands that placeholder during project env save and service
  runtime-variable create/update. It provisions or reuses a project token,
  stores the real token securely, creates/updates AIHubKey tracking, and forces
  paired base URL vars to `https://api.opendeploy.dev/v1`.
- This is a runtime env contract. Do not rely on it for `build_variables`.

## Detection

Prefer the CLI/analyzer `ai_config` when present, then verify high-confidence
evidence locally:

- AI SDK dependencies or imports such as `openai`, `@ai-sdk/openai`,
  `@anthropic-ai/sdk`, `@google/generative-ai`, `@ai-sdk/google`,
  `@ai-sdk/anthropic`, `@openrouter/ai-sdk-provider`, `groq-sdk`, `mistralai`,
  `together-ai`, `cohere`, `replicate`, LangChain, LlamaIndex, or provider
  aliases surfaced by the analyzer.
- Env key names with provider prefixes and AI key suffixes, for example
  `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`,
  `GOOGLE_GENERATIVE_AI_API_KEY`, `OPENROUTER_API_KEY`, `GROQ_API_KEY`,
  `MISTRAL_API_KEY`, `DEEPSEEK_API_KEY`, `TOGETHER_API_KEY`,
  `PERPLEXITY_API_KEY`, `DASHSCOPE_API_KEY`, `QWEN_API_KEY`, `ARK_API_KEY`, or
  `XAI_API_KEY`.
- Paired base URL keys such as `OPENAI_BASE_URL`, `OPENAI_API_BASE`,
  `ANTHROPIC_BASE_URL`, `GEMINI_BASE_URL`, `OPENROUTER_BASE_URL`, or any
  analyzer-provided `base_url_vars`.
- If an OpenAI-compatible key is detected but no base URL var is listed, verify
  whether the SDK/framework supports a standard base URL env (`OPENAI_BASE_URL`,
  `OPENAI_API_BASE`, provider-specific equivalent). Add a base URL var only with
  source/docs/SDK evidence; otherwise tell the user OpenDeploy AI API may not be
  routable for that app shape and offer user-owned keys.

Reduce false positives:

- Do not treat generic `GITHUB_TOKEN`, OAuth client secrets, webhook secrets,
  payment keys, SMTP passwords, storage keys, or unrelated `_TOKEN` values as AI
  API keys unless provider/package evidence ties them to AI.
- If an app supports many optional providers, include only keys that source,
  docs, env examples, or analyzer output mark as required for the selected
  deployment mode. Optional integrations can stay unset or use boot-safe
  placeholders only if the app boots without them.

## User question

When AI API is required or likely startup-critical, ask before cloud mutation:

```text
question: "This project uses AI API env keys: <KEYS>. How should OpenDeploy configure them?"
header:   "AI API"
options:
  - label: "Use OpenDeploy AI API"
    description: "Recommended. OpenDeploy provisions a project AI token and wires the detected runtime vars; no provider key paste needed."
  - label: "Use my own AI keys"
    description: "You provide the real provider key values through the normal key-only env-upload flow."
  - label: "Continue without AI"
    description: "Only valid when source evidence says AI is optional for boot and the first smoke test."
```

If the app cannot boot without AI keys, replace `Continue without AI` with
`Pause before deploy`.

## Applying the decision

For `Use OpenDeploy AI API`:

- Add each `ai_config.api_key_vars[]` key to the service `runtime_variables`
  map with value `{{MINIONS_AI_API_KEY}}`.
- Add each `ai_config.base_url_vars[]` key to the service `runtime_variables`
  map with value `https://api.opendeploy.dev/v1`.
- If source evidence proves a standard provider base URL var is supported but
  the analyzer omitted it, add that key to `base_url_vars` before applying the
  managed choice and record the evidence.
- Mark the API key vars as sensitive in any local/body metadata when the shape
  supports it. Base URL vars are not secrets.
- Keep these keys out of `build_variables` unless source evidence proves the
  same key is also read at build time. Even then, do not use the placeholder in
  build vars; ask for a user-provided build-time key or patch the build so AI
  calls happen at runtime.
- For existing services, patch runtime env and create a new deployment. A
  restart is not a reliable way to refresh app-visible env.

For `Use my own AI keys`:

- Use the normal `opendeploy-env` key-only consent flow.
- Accept values via structured secret input or a local `0600` file/body.
- Never print provider key values.

For `Continue without AI`:

- Set no fake provider key unless the app specifically requires a syntactically
  valid placeholder and source evidence proves it will not call the provider
  during boot/smoke test.
- Record a caveat that AI features will fail until real/OpenDeploy-managed AI
  env is configured and redeployed.

## Attempt record

Record AI decisions with env key names only:

```json
{
  "ai_api": {
    "requires_ai_api_key": true,
    "detected_providers": ["openai"],
    "api_key_vars": ["OPENAI_API_KEY"],
    "base_url_vars": ["OPENAI_BASE_URL"],
    "evidence": ["package @ai-sdk/openai", ".env.example lists OPENAI_API_KEY"],
    "user_choice": "opendeploy_ai_api",
    "runtime_variables_set": ["OPENAI_API_KEY", "OPENAI_BASE_URL"],
    "build_time_blockers": []
  }
}
```
