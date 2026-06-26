# SETUP.md — Nanmin Game Studio org profile

This doc is for future-you (or anyone else who touches this). It explains how
this repo is structured, how the automation works, and the exact commands to
rebuild it from zero.

---

## 1. What lives where

```
.github/                          <- the repo itself is named ".github"
├── profile/
│   └── README.md                 <- renders on github.com/Nanmin-Game-Studio
├── .github/
│   └── workflows/
│       ├── 3d-contrib.yml        <- generates cube.svg
│       └── snake.yml             <- generates pipeline.svg / pipeline-dark.svg
├── README.md                     <- explains the repo itself (not the org homepage)
└── SETUP.md                      <- this file
```

Yes, there's a `.github` folder *inside* the `.github` repo. That's not a typo
— GitHub Actions always looks for workflows at `.github/workflows/` relative
to repo root, regardless of what the repo itself is named.

## 2. Important technical reality check

GitHub does not have a "contribution graph" for organizations — only for
individual user accounts. `cube.svg` and `pipeline.svg` will always reflect
**your personal account's** commit activity, not some aggregate "org
activity." There is no way around this; it's a platform limitation, not a
config option. Set `SOURCE_NAME` / `github_user_name` in both workflow files
to your personal GitHub username, not the org name.

## 3. One-time manual setup (can't be done via git)

These live in GitHub's web UI, not in any file:

1. **Allow workflows to push:**
   Repo → Settings → Actions → General → Workflow permissions →
   select **"Read and write permissions"** → Save.
   Without this, both workflows will fail on `git push` with a 403.

2. **Org bio / location / contact email shown at the top of the org page:**
   https://github.com/organizations/Nanmin-Game-Studio/settings/profile
   These are account-level settings fields, not files — no git command
   touches them.

## 4. Full rebuild — wipe and recreate from scratch

Run this locally on your machine (needs `gh` CLI authenticated — check with
`gh auth status` first).

```bash
# 0. Back up the current repo before nuking it. Five seconds, no excuse to skip this.
gh repo clone Nanmin-Game-Studio/.github ~/nanmin-backup

# 1. Delete the existing repo
gh repo delete Nanmin-Game-Studio/.github --yes

# 2. Build the new structure locally
mkdir -p ~/projects/nanmin-github/profile
mkdir -p ~/projects/nanmin-github/.github/workflows
cd ~/projects/nanmin-github

# 3. Place these files (download from the conversation, then move them in):
#    profile-README.md  -> profile/README.md
#    repo-README.md      -> README.md
#    SETUP.md            -> SETUP.md
#    3d-contrib.yml       -> .github/workflows/3d-contrib.yml
#    snake.yml             -> .github/workflows/snake.yml
#
# Example, once downloaded into ~/Downloads:
mv ~/Downloads/profile-README.md profile/README.md
mv ~/Downloads/repo-README.md README.md
mv ~/Downloads/SETUP.md SETUP.md
mv ~/Downloads/3d-contrib.yml .github/workflows/3d-contrib.yml
mv ~/Downloads/snake.yml .github/workflows/snake.yml

# 4. IMPORTANT: edit both workflow files first —
#    replace YOUR_GITHUB_USERNAME with your actual personal GitHub username
#    in 3d-contrib.yml and snake.yml before committing.

# 5. Init and push as a new repo
git init -b main
git add .
git commit -m "Fresh start: Nanmin Game Studio org profile, Helapidi-focused"
gh repo create Nanmin-Game-Studio/.github --public --source=. --remote=origin --push

# 6. Go set the workflow permission (step 3.1 above) — do this before
#    triggering the workflows or the push step inside them will fail.

# 7. Manually trigger both workflows once so cube.svg / pipeline.svg exist
gh workflow run "3D Contribution Graph" -R Nanmin-Game-Studio/.github
gh workflow run "Contribution Snake (pipeline.svg)" -R Nanmin-Game-Studio/.github

# 8. Check they succeeded
gh run list -R Nanmin-Game-Studio/.github
```

If a run fails, check the Actions tab in the repo on github.com — the logs
will tell you exactly which step broke. Usually it's either the username
placeholder not being replaced, or the workflow permission step being
skipped.

## 5. Adding Hēlapidi's repo (and any future game) to the org

```bash
cd /path/to/your/game/project
gh repo create Nanmin-Game-Studio/helapidi --public --source=. --remote=origin --push
```

For any future game beyond Hēlapidi: don't add it to `profile/README.md`
until it's a real, confirmed, shippable thing you can link a working repo
to. Right now this org is single-game focused on purpose — keep it that way
until there's a second game worth showing.
