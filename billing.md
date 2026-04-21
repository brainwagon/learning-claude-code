# Claude Code: Billing and Account Types

A practical guide to understanding Claude billing options — what each tier includes, how to check your status, and which is right for a recreational programmer on a budget.

---

## Account Types

### Claude.ai Pro ($20/month flat fee)
- Fixed, predictable monthly cost
- Access via web chat, mobile app, and the Claude Code CLI
- High but not unlimited message/rate limits
- No API key or coding required to use
- Usage monitoring is minimal — since it's flat-fee, Anthropic doesn't expose granular stats
- Managed at **claude.ai** in your account settings

### Anthropic API (pay-per-token)
- No flat fee — you pay based on tokens consumed (input + output text)
- Can be very cheap for light use, or expensive for heavy/looping scripts
- Requires coding to use: you write scripts or apps that call the API directly
- Full usage dashboard available at `console.anthropic.com`
- Best for: building apps others use, large batch automation, programmatic integrations

---

## Which Is Right for You?

For a **recreational programmer on a limited budget**, Claude.ai Pro is almost always the right choice:

- **Predictable cost** — no surprise bills
- **Covers Claude Code** — the CLI authenticates directly via your claude.ai login
- **No risk of runaway charges** — a buggy script with a raw API key can generate unexpected costs fast
- **Simpler setup** — no API key management needed

The raw API is worth considering only if you're building apps that other people use, or running large batch jobs where pay-per-token would be cheaper than $20/month.

---

## Checking Your Auth Status

To confirm which account type Claude Code is using:

```
! claude auth status
```

A Claude.ai Pro account looks like:

```json
{
  "loggedIn": true,
  "authMethod": "claude.ai",
  "apiProvider": "firstParty",
  "subscriptionType": "pro"
}
```

A raw API key setup would show `"authMethod": "apiKey"` instead.

---

## Monitoring Usage

**Claude.ai Pro:** No detailed usage dashboard. Since it's flat-fee, the main constraint is rate limits — if you hit one, Claude Code will tell you and you wait before sending more.

Within Claude Code, you can inspect the current session:

- **`/context`** — shows how much of the current conversation's context window is consumed
- **`/status`** — shows active model and connection state

**Raw API:** Full token-level usage stats available at `console.anthropic.com`. Each API response also returns per-call token counts in `response.usage`.

---

## Key Distinction

`console.anthropic.com` is for **raw API accounts only**. If your `claude auth status` shows `"authMethod": "claude.ai"`, you don't have (and don't need) a console account — your billing is managed entirely through claude.ai.
