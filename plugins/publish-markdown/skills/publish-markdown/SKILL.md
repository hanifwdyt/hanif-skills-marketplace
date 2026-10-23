---
name: publish-markdown
description: >-
  Publish markdown to tools.noesis.nz/markdown (the old markdown.hanif.app
  domain is dead; same app, same slugs) and return a shareable short-link URL —
  not just produce a .md file, but actually push it online and hand back the link.
  Use when the user asks to "put in markdown site", "buat markdown online",
  "publish markdown", "push ke markdown", "markdown noesis", "markdown hanif app", "kasih link markdown",
  "jadiin link", "markdown online", or wants any markdown content turned into a
  live URL. Also handles updating an existing doc (same link) and listing docs.
  WHEN NOT: the user only wants a local .md file on disk and explicitly does not
  want it published.
---

# Publish Markdown → tools.noesis.nz/markdown

Turn markdown content into a live, shareable short link via the tools.noesis.nz/markdown
API. The deliverable is the **URL**, not just the markdown.

## Credentials (set these up first)

Read credentials in this order:
1. Env vars `MARKDOWN_HANIF_EMAIL` + `MARKDOWN_HANIF_PASSWORD`, else
2. `~/.config/markdown-hanif/account.json` → `{ "email": "...", "password": "..." }`

Never print the password. If neither is set, tell the user how to configure it
(see the README) and stop — don't guess.

## API (Base URL: https://tools.noesis.nz/markdown)

1. **Auth** — `POST /api/v1/auth/token` body `{email, password}` → `{ token, ... }` (Bearer, 30d).
2. **Push (create)** — `POST /api/v1/docs` header `Authorization: Bearer <token>`, body:
   - `content` (string, required) — the markdown
   - `slug` (optional, 3–50 chars `a-z 0-9 -`) — custom link
   - `theme`/`font` (optional, default `light`/`serif`), `passcode` (optional)
   → `201 { id, url, slug, has_passcode }`. Slug taken → `409`. Bad slug → `400`.
3. **List** — `GET /api/v1/docs` → `{ count, docs:[{id,title,url,slug,views,...}] }`.
4. **Get one** — `GET /api/v1/docs/<id-or-slug>` → includes raw `content`.
5. **Replace/edit (same link)** — `PUT /api/v1/docs/<id-or-slug>` body `{ content, theme?, font? }` → `200`.

## Workflow

1. **Assemble the markdown** to publish (a file the user points to, content from the
   conversation, or something you generate). Write it to a temp file.
2. **Confirm once before the first network hit.** Say plainly, e.g.
   *"Gue udah ready buat push ke tools.noesis.nz/markdown — approve?"* and wait. After the
   user approves within a session, no need to re-ask for further pushes that turn.
3. **Auth** to get a token.
4. **Push** the content. If the user gave a custom name, pass it as `slug`. On `409`
   (slug taken), tell them and offer to `PUT` (overwrite) or pick another slug.
5. **Return the `url`** clearly as the result. For "update it"/"edit", use `PUT` on the
   same slug/id so the link stays the same.

## Implementation (Bash + curl)

JSON-encode the markdown safely (newlines/quotes/backticks) — build the body with
`jq -Rs` or Python; never string-interpolate raw markdown into JSON.

```bash
EMAIL="${MARKDOWN_HANIF_EMAIL:-$(python3 -c "import json,os;p=os.path.expanduser('~/.config/markdown-hanif/account.json');print(json.load(open(p))['email'])" 2>/dev/null)}"
PASS="${MARKDOWN_HANIF_PASSWORD:-$(python3 -c "import json,os;p=os.path.expanduser('~/.config/markdown-hanif/account.json');print(json.load(open(p))['password'])" 2>/dev/null)}"

# 1) token
TOKEN=$(curl -s -X POST https://tools.noesis.nz/markdown/api/v1/auth/token \
  -H 'Content-Type: application/json' \
  -d "$(python3 -c "import json,sys;print(json.dumps({'email':sys.argv[1],'password':sys.argv[2]}))" "$EMAIL" "$PASS")" \
  | python3 -c "import sys,json;print(json.load(sys.stdin).get('token',''))")

# 2) push (SLUG optional; leave empty for auto /d/<id> link)
python3 -c "import json,sys;print(json.dumps({'content':open(sys.argv[1]).read(), **({'slug':sys.argv[2]} if len(sys.argv)>2 and sys.argv[2] else {})}))" /tmp/doc.md "$SLUG" > /tmp/doc_body.json
curl -s -X POST https://tools.noesis.nz/markdown/api/v1/docs \
  -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  --data @/tmp/doc_body.json
# → {"id":...,"url":"https://tools.noesis.nz/markdown/<slug>","slug":...}
```

Report the returned `url`. On `401` from auth, the credentials are wrong/expired —
tell the user, don't loop.
