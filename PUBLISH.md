# Publishing the Five-Bot plugin

Everything below is run by you, with your own GitHub login. Nobody has to share credentials.

## 1. Push to GitHub (your AngryMunky account, public)

From this folder (`C:\AI Bins\Code\5bot-plugin`):

```
git init
git add .
git commit -m "five-bot plugin v1.1.0"
git branch -M main
```

Then create the public repo and push. With the GitHub CLI:

```
gh repo create AngryMunky/5bot-plugin --public --source=. --remote=origin --push
```

Or, without `gh`: create an empty PUBLIC repo named `5bot-plugin` at github.com/AngryMunky (no README), then:

```
git remote add origin https://github.com/AngryMunky/5bot-plugin.git
git push -u origin main
```

## 2. (Optional) Test it on your own machine first

```
/plugin marketplace add AngryMunky/5bot-plugin
/plugin install five-bot@lawson-design
```

Then in a throwaway project: `/5bot-init`, then `/product`.

## 3. Publish org-wide via the admin console

Org settings -> Libraries -> Plugins -> **Add plugins**. Point it at the repo `AngryMunky/5bot-plugin`. That registers the `lawson-design` marketplace and the `five-bot` plugin for your org. Set the **User access** level for five-bot (Available to install, or a stronger/required option if the dropdown offers one).

Reminder: org auto-install is smooth in the Claude desktop and web apps; pure Claude Code CLI users may still need to run the two install commands in step 2.

## Updating later

Bump `version` in `five-bot/.claude-plugin/plugin.json` and in the `five-bot` entry of `.claude-plugin/marketplace.json`, then commit and push. Members with auto-update on pick it up; others run `/plugin update`.
