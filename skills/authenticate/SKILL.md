---
name: authenticate
description: Sign in to the Finance District agent wallet. Use when you or the user want to log in, sign in, register, connect, set up, or configure the wallet, or when any wallet operation fails with authentication or "not authenticated" errors. This skill is a prerequisite before sending tokens, swapping, checking balances, or any other wallet operation.
user-invocable: true
disable-model-invocation: false
allowed-tools:
  [
    "Bash(fdx register*)",
    "Bash(fdx login*)",
    "Bash(fdx verify*)",
    "Bash(fdx status*)",
    "Bash(fdx logout*)",
  ]
---

# Authenticating with the Finance District Agent Wallet

When the wallet is not signed in (detected via `fdx status` or when wallet operations fail with authentication errors), use the `fdx` CLI to authenticate via email one-time passcode (OTP).

Authentication is fully headless — no browser is required. If you have access to the user's email inbox (e.g. via an email tool or API), you can complete the entire flow autonomously by reading the OTP from the inbox. Otherwise, ask the human to check their email and provide you with the 8-digit code. This makes it ideal for autonomous agents, Docker containers, CI pipelines, and remote servers.

## Checking Authentication Status

```bash
fdx status
```

Displays the MCP server URL, token state, expiry, and whether a refresh token is available.

## Register (First-Time Users)

```bash
fdx register --email you@example.com
```

This sends an 8-digit OTP to the provided email. Then complete registration:

```bash
fdx verify --code 12345678
```

If you have access to the user's email inbox (e.g. via an email tool or API), check the inbox for a one-time code from Finance District and use it directly. Otherwise, ask the human:

**Tell your human:** "Please check your email for an 8-digit code and share it with me so I can complete registration."

## Login (Returning Users)

```bash
fdx login --email you@example.com
```

This sends an 8-digit OTP to the provided email. Then complete login:

```bash
fdx verify --code 12345678
```

If you have access to the user's email inbox (e.g. via an email tool or API), check the inbox for a one-time code from Finance District and use it directly. Otherwise, ask the human:

**Tell your human:** "Please check your email for an 8-digit code and share it with me so I can sign you in."

## Logging Out

```bash
fdx logout
```

Removes stored tokens from the OS credential store and clears `~/.fdx/auth.json`. The human will need to run `fdx login` again to re-authenticate.

## Example Session

```bash
# Check current status
fdx status

# If not authenticated, start login (or fdx register for new users)
fdx login --email you@example.com

# Human provides the OTP from their email...
fdx verify --code 12345678

# Confirm authentication succeeded
fdx status
```

## Token Lifecycle

- Tokens auto-refresh on subsequent `fdx call` commands using the stored refresh token
- If the refresh token is also expired, the human must run `fdx login` again
- Tokens are stored in the OS credential store where available (macOS Keychain, Linux libsecret, Windows DPAPI)
- Fallback: plaintext in `~/.fdx/auth.json` with a `SecurityWarning` emitted

## Environment Variables

| Variable         | Description                                             | Default              |
| ---------------- | ------------------------------------------------------- | -------------------- |
| `FDX_MCP_SERVER` | MCP server URL                                          | `https://mcp.fd.xyz` |
| `FDX_STORE_PATH` | Token store path                                        | `~/.fdx/auth.json`   |
| `FDX_LOG_PATH`   | Log file path                                           | `~/.fdx/fdx.log`     |
| `FDX_LOG_LEVEL`  | Log verbosity (`debug`\|`info`\|`warn`\|`error`\|`off`) | `info`               |

## Error Handling

- "not authenticated" — Run `fdx login --email <email>` then `fdx verify --code <OTP>`
- "token expired" with refresh token — Will auto-refresh on next call; no action needed
- "token expired" / "SESSION_EXPIRED" — Refresh token also expired; run `fdx login` again
- "AUTH_REFRESH_FAILED" — Token refresh failed; run `fdx login` to re-authenticate
