# Configure Klaatu in OpenCode

KlaatAI exposes an OpenAI-compatible Chat Completions API. OpenCode can connect to it directly using the `@ai-sdk/openai-compatible` provider; no proxy is required.

## Prerequisites

You need:

- [OpenCode](https://opencode.ai/docs/) installed
- A valid KlaatAI API key
- Access to `https://api.klaatai.com/v1`

## 1. Add your KlaatAI API key

Run:

```bash
opencode auth login
```

When prompted, enter:

```text
Provider: Other
Provider ID: klaatai
API key: <your KlaatAI API key>
```

The provider ID must be exactly `klaatai`. Paste only the API key, without a `Bearer` prefix.

OpenCode stores the credential in its local auth store. On Unix-like systems, the default location is:

```text
~/.local/share/opencode/auth.json
```

Verify that it was saved:

```bash
opencode auth list
```

You should see `klaatai` under **Credentials**.

## 2. Configure the provider and model

Create an `opencode.json` file. For project-specific configuration, place it in the project root:

```text
your-project/opencode.json
```

For global configuration, use:

```text
~/.config/opencode/opencode.json
```

Add this configuration:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "klaatai/klaatu",
  "default_agent": "klaatu",
  "provider": {
    "klaatai": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "KlaatAI",
      "options": {
        "baseURL": "https://api.klaatai.com/v1"
      },
      "models": {
        "klaatu": {
          "name": "Klaatu"
        }
      }
    }
  },
  "agent": {
    "klaatu": {
      "description": "General-purpose coding agent powered by the Klaatu model router",
      "mode": "primary",
      "model": "klaatai/klaatu"
    }
  }
}
```

This defines:

| Setting | Value |
|---|---|
| Provider ID | `klaatai` |
| Provider name | `KlaatAI` |
| API base URL | `https://api.klaatai.com/v1` |
| API protocol | OpenAI-compatible Chat Completions |
| Model ID | `klaatu` |
| OpenCode model ID | `klaatai/klaatu` |
| Default agent | `klaatu` |

OpenCode uses `@ai-sdk/openai-compatible` for providers that implement `/v1/chat/completions`. See OpenCode's [custom provider documentation](https://opencode.ai/docs/providers/#custom-provider) for more detail.

## 3. Verify the configuration

Confirm that OpenCode recognizes the provider and model:

```bash
opencode models klaatai
```

The output should include:

```text
klaatai/klaatu
```

Confirm that the Klaatu agent exists:

```bash
opencode agent list
```

You can also inspect the final merged configuration:

```bash
opencode debug config
```

## 4. Start OpenCode

Start normally:

```bash
opencode
```

Or explicitly select the agent and model:

```bash
opencode --agent klaatu --model klaatai/klaatu
```

Inside OpenCode, the status line should show:

```text
Klaatu · Klaatu · KlaatAI
```

These values represent `Agent · Model · Provider`.

## 5. Start a new session

If OpenCode offers these choices:

```text
New session
Continue the existing OpenCode session
```

Select **New session**. Existing sessions preserve the agent that was active when they were created, so continuing an older session may still show a different agent even after changing `default_agent`.

You can also press Tab inside OpenCode to cycle between available primary agents.

## Troubleshooting

### “Model klaatai/klaatu is not valid”

OpenCode did not load the custom provider configuration.

Check that the model is registered:

```bash
opencode models klaatai
```

Check the resolved configuration:

```bash
opencode debug config
```

If you use project configuration, ensure `opencode.json` is in the directory where you run OpenCode or in the project root. You can explicitly specify its path:

```bash
OPENCODE_CONFIG="$PWD/opencode.json" \
  opencode --agent klaatu --model klaatai/klaatu
```

Ensure that the schema and base URL are plain URLs rather than Markdown links.

Correct:

```json
"$schema": "https://opencode.ai/config.json"
```

Incorrect:

```json
"$schema": "[https://opencode.ai/config.json](https://opencode.ai/config.json)"
```

### “Missing or invalid Authorization header”

OpenCode has not stored a credential for the `klaatai` provider.

Run `opencode auth login`, choose **Other**, use `klaatai` as the provider ID, and paste your API key. Then verify it with:

```bash
opencode auth list
```

The provider ID used during authentication must match the provider key in `opencode.json`.

### OpenCode still shows another agent

The first value in OpenCode's status line is the agent name. If the status line shows another agent followed by `Klaatu · KlaatAI`, Klaatu is already being used as the model but an older agent remains active.

To switch:

- Start a new session.
- Press Tab until the `Klaatu` agent is selected.
- Or launch explicitly with `opencode --agent klaatu --model klaatai/klaatu`.

The `default_agent` setting applies when creating a new session; it does not rewrite the agent stored in existing sessions.
