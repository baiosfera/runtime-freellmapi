# runtime-freellmapi

FreeLLMAPI multi-provider `$0` router on Zerops (`ubuntu/nodejs@24`) with persistent SQLite (`/mnt/localstorage/freellmapi/data/freeapi.db`) and declarative JSON startup provisioning.

## Zerops & Declarative Config Precedence

Configured via root [`zerops.yaml`](zerops.yaml):
1. `FREEAPI_CONFIG_JSON` (`${FREEAPI_CONFIG_JSON_SECRET}`) takes precedence over `FREEAPI_CONFIG_PATH` (`/mnt/localstorage/freellmapi/config.json`).
2. `FREEAPI_UNIFIED_API_KEY` (`${FREELLMAPI_CLIENT_KEY}`) secures `/v1/*` inference endpoints.
3. `ENCRYPTION_KEY` (`${FREEAPI_ENCRYPTION_KEY}`) encrypts stored provider keys in SQLite.

## Supported Declarative Keys (`declarativeConfigSchema`)

Valid top-level keys (`.strict()` schema):
- `admin`: `{ email, password (min 8) }` — creates initial dashboard user only if the `users` table is empty.
- `unifiedApiKey`: string (min 8 chars).
- `license`: optional license key (min 8 chars).
- `keys`: built-in provider keys (`{ platform, key?, label?, baseUrl?, enabled? }`).
  - Built-in **keyless** platforms (`kilo`, `ovh`, `aihorde`) require no `key` field.
  - Note: custom endpoint entries under `customProviders` require a non-empty `apiKey`.
- `models`: per-model overrides (`enabled`, `fallbackEnabled`, `sizeLabel`, `contextWindow`, `tpmLimit`, `intelligenceRank`, `speedRank`).
  - Disable safety-classifier/guard and broken catalog entries (`openai/gpt-oss-safeguard-20b`, `allam-2-7b`, `meta-llama/llama-prompt-guard-*`, `ovh/Qwen3Guard-Gen-*`, `nvidia/*guard*`).
  - For Hermes / CrewAI tool-calling agents, keep `sizeLabel` (`Medium`/`Large`/`Frontier`) and `contextWindow >= 65536` accurate.
- `fallback`: explicit priority list (`[{ platform, modelId, priority, enabled }]`).
- `routing`:
  - `strategy`: `'priority' | 'balanced' | 'smartest' | 'fastest' | 'reliable' | 'custom'`
  - `keySelectionStrategy`: `'auto' | 'least-remaining'` (orthogonal to `strategy`).
