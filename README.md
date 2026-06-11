# Lnkgo

Branded tracked links, QR codes, analytics, and custom-domain short URLs for
agents and developers.

- Product: https://lnkgo.app
- API: https://api.lnkgo.app
- Default short-link domain: `lnkgo.app`
- npm CLI: https://www.npmjs.com/package/lnkgo

## Install The Agent Skill

Install through the open skills CLI:

```bash
npx --yes skills add citedy/lnkgo --skill lnkgo -a codex -g -y
npx --yes skills add citedy/lnkgo --skill lnkgo -a claude-code -g -y
npx --yes skills add citedy/lnkgo --skill lnkgo -a cursor -g -y
```

Or install the bundled skill from the npm CLI:

```bash
npx --yes lnkgo skill install --target codex
npx --yes lnkgo skill install --target claude
npx --yes lnkgo skill install --target cursor
npx --yes lnkgo skill install --target project
```

`project` writes the skill to `.codex/skills/lnkgo/SKILL.md` in the current
working directory.

## Use The CLI

```bash
npx --yes lnkgo init --email you@example.com
export LNKGO_VERIFICATION_CODE="<email-code>"
export LNKGO_API_KEY="$(
  npx --yes lnkgo init --email you@example.com --code "$LNKGO_VERIFICATION_CODE" \
    | jq -r '.api_key'
)"

npx --yes lnkgo create --url https://example.com/launch --tag product-hunt
npx --yes lnkgo qr <link-id> --output lnkgo-qr.png
npx --yes lnkgo analytics <link-id>
```

Default-domain links do not need `--domain`. Use `--domain` only for an active
custom domain owned by the account.

## Use The REST API

```bash
curl -sS -X POST https://api.lnkgo.app/v1/links \
  -H "Authorization: Bearer $LNKGO_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen | tr '[:upper:]' '[:lower:]')" \
  -d '{"url":"https://example.com/launch","tags":["product-hunt"]}'
```

## Skill Source

The public skill lives at:

```text
skills/lnkgo/SKILL.md
```

It tells compatible agents to use `https://api.lnkgo.app`, `lnkgo.app`, and
`LNKGO_API_KEY` for Lnkgo workflows.
