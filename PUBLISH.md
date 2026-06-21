# Publishing / updating the 5bot plugin

Everything below is run by you, with your own GitHub login. Nobody has to share credentials.

## Where it lives — two repos

5bot is published as **two GitHub repos** with identical content, kept in sync:

| Repo | Visibility | Purpose |
|------|-----------|---------|
| `AngryMunky/5bot-plugin` | **Private** | Source of truth + claude.ai "Sync from GitHub" (Cowork). That dialog only lists private/internal repos. |
| `AngryMunky/5bot` | **Public** | CLI marketplace installs for anyone. |

GitHub visibility is per-repository, **not** per-branch — so a "public branch of a private repo" isn't possible. That's why the public copy is a separate mirror repo, not a branch.

## Installing

**Public CLI (anyone):**
```
/plugin marketplace add AngryMunky/5bot
/plugin install 5bot@lawson-design
```

**Private / Cowork:** in claude.ai, use **Sync from GitHub** and pick `AngryMunky/5bot-plugin` (it syncs `main`).

Then in any project folder: `/5bot-init`, then `/product`.

## Updating (cut a new version)

1. Bump `version` in the canonical spot — `5bot/.claude-plugin/plugin.json` — and mirror the **same** number into the `5bot` entry of `.claude-plugin/marketplace.json`. They must match (a drift between them was the v1.1.0 bug).
2. Commit.
3. Push. The canonical clone's `origin` is configured to **dual-push to both repos**, so one push updates both:
   ```
   git push origin main      # → updates BOTH 5bot-plugin (private) and 5bot (public)
   ```
4. Tag the release and push tags to both:
   ```
   git tag vX.Y.Z
   git push origin --tags
   git push public --tags
   ```

Members with auto-update on pick it up; others run `/plugin update`. Cowork users re-sync via the GitHub dialog.

## Keeping the two repos in sync

The canonical clone (`C:\Users\capta\AppData\Local\Temp\5bot-plugin-release`) has `origin` set to push to **both** repo URLs, so a single `git push origin main` updates both. Fetch still comes only from the private repo. Manual fallback for the public mirror alone: `git push public main`.

⚠️ The clone is in a Windows Temp folder. If it gets cleaned out, the dual-push config goes with it. Rebuild it:
```
git clone https://github.com/AngryMunky/5bot-plugin.git
cd 5bot-plugin
git remote add public https://github.com/AngryMunky/5bot.git
git remote set-url --add --push origin https://github.com/AngryMunky/5bot-plugin.git
git remote set-url --add --push origin https://github.com/AngryMunky/5bot.git
```

## (Optional) Org-wide via the admin console

Org settings → Libraries → Plugins → **Add plugins**. Point it at `AngryMunky/5bot` (public). That registers the `lawson-design` marketplace and the `5bot` plugin for your org. Set the **User access** level (Available to install, or stronger). Org auto-install is smooth in the Claude desktop and web apps; pure Claude Code CLI users may still run the two install commands above.
