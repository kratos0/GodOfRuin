# GodOfRuin — Repo Setup

How to connect this repo on a new machine using the Neo deploy key.

---

## 1. Generate the SSH key

```bash
ssh-keygen -t ed25519 -C "Ehiller0@yahoo.com" -f ~/.ssh/neo -N ""
```

## 2. Add the public key to GitHub

```bash
cat ~/.ssh/neo.pub
```

Copy the output. Go to:
**GitHub → kratos0/GodOfRuin → Settings → Deploy keys → Add deploy key**

- Title: `Neo`
- Key: paste the output from above
- Check **Allow write access**
- Click **Add key**

## 3. Add SSH config entry

Append to `~/.ssh/config` (create the file if it doesn't exist):

```
Host github-godoffruin
    HostName github.com
    User git
    IdentityFile ~/.ssh/neo
    IdentitiesOnly yes
```

## 4. Test the connection

```bash
ssh -T git@github-godoffruin
```

Expected response: `Hi kratos0/GodOfRuin! You've successfully authenticated...`

## 5. Init and connect the repo

```bash
cd "/Users/hillercomputer/Documents/Unreal Projects/GodOfRuin"
git init
git config user.name "kratos0"
git config user.email "Ehiller0@yahoo.com"
git remote add origin git@github-godoffruin:kratos0/GodOfRuin.git
git fetch origin
git checkout -B main origin/main --force
```

## Notes

- The remote URL uses `github-godoffruin` (the SSH config alias), not `github.com`
- Content/ is excluded by .gitignore — 15 GB of binary assets are not tracked
- To enable LFS for assets later: `brew install git-lfs && git lfs install`
