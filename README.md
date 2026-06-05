# hanif-skills — Claude Code plugin marketplace

A git-based [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin
marketplace. Currently ships one plugin:

## `publish-markdown`

Publish markdown to **[markdown.hanif.app](https://markdown.hanif.app)** and get
back a shareable short-link URL — straight from Claude Code. Create, update (same
link), and list your docs.

### Install

```bash
/plugin marketplace add hanifwdyt/hanif-skills-marketplace   # or the full git URL
/plugin install publish-markdown@hanif-skills
```

### Configure credentials

The plugin needs your markdown.hanif.app account. Pick one:

**Option A — environment variables** (recommended)
```bash
export MARKDOWN_HANIF_EMAIL="you@example.com"
export MARKDOWN_HANIF_PASSWORD="your-password"
```

**Option B — config file**
```bash
mkdir -p ~/.config/markdown-hanif
cp plugins/publish-markdown/account.example.json ~/.config/markdown-hanif/account.json
# then edit it with your real email + password
```

> Your password is **never** committed. `account.json` is gitignored; the repo only
> ships `account.example.json` as a template.

### Use

Just ask, e.g.:
- "publish this as markdown online"
- "buat markdown online, kasih link"
- "jadiin link markdown dengan slug rilis-notes"

Claude confirms once before the first network call, then pushes and returns the URL.
Say "update it" / "edit" to replace the content while keeping the same link.

### API reference

Full API docs: <https://markdown.hanif.app/api>

| Action | Endpoint |
|--------|----------|
| Auth (token, 30d) | `POST /api/v1/auth/token` |
| Create | `POST /api/v1/docs` |
| List | `GET /api/v1/docs` |
| Get one | `GET /api/v1/docs/<id-or-slug>` |
| Replace/edit | `PUT /api/v1/docs/<id-or-slug>` |

## License

MIT
