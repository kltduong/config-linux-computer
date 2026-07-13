# Configure Two Git Accounts Automatically

This document configures two Git accounts on Linux/Kubuntu:

- Personal GitHub account
- Work Azure DevOps account

It reuses existing SSH keys. You do **not** need to recreate keys.

---

## Goal

Repositories are separated by directory:

```text
~/data/code/personal/   -> GitHub personal repos
~/data/code/defide/     -> Azure DevOps work repos
```

Git should automatically use the correct:

```text
SSH key
Git user.name
Git user.email
```

---

## Step 0: Verify Existing SSH Keys

First, check whether you already have the SSH key files.

Run:

```bash
ls -l ~/.ssh/github--kltduong@gmail.com \
      ~/.ssh/github--kltduong@gmail.com.pub \
      ~/.ssh/azure--duong.kieu@defide-ix.com \
      ~/.ssh/azure--duong.kieu@defide-ix.com.pub
```

If all 4 files exist, skip SSH key creation and continue to Step 1.

Expected files:

```text
~/.ssh/github--kltduong@gmail.com
~/.ssh/github--kltduong@gmail.com.pub

~/.ssh/azure--duong.kieu@defide-ix.com
~/.ssh/azure--duong.kieu@defide-ix.com.pub
```

The files without `.pub` are private keys. Do **not** share them.

The `.pub` files are public keys. Add these to GitHub and Azure DevOps.

If the files do not exist, create them with these commands.

Create GitHub personal key:

```bash
ssh-keygen -t ed25519 -C "kltduong@gmail.com" -f ~/.ssh/github--kltduong@gmail.com
```

Create Azure DevOps work key:

```bash
ssh-keygen -t ed25519 -C "duong.kieu@defide-ix.com" -f ~/.ssh/azure--duong.kieu@defide-ix.com
```

After creating them, continue to Step 1.

---

## Step 1: Fix SSH Permissions

Run:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/github--kltduong@gmail.com
chmod 600 ~/.ssh/azure--duong.kieu@defide-ix.com
chmod 644 ~/.ssh/github--kltduong@gmail.com.pub
chmod 644 ~/.ssh/azure--duong.kieu@defide-ix.com.pub
```

---

## Step 2: Configure SSH Key Selection

Edit SSH config:

```bash
vim ~/.ssh/config
```

Add this:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github--kltduong@gmail.com
    IdentitiesOnly yes

Host vs-ssh.visualstudio.com
    HostName vs-ssh.visualstudio.com
    IdentityFile ~/.ssh/azure--duong.kieu@defide-ix.com
    IdentitiesOnly yes
```

Then run:

```bash
chmod 600 ~/.ssh/config
```

Explanation:

```text
github.com              -> use GitHub personal SSH key
vs-ssh.visualstudio.com -> use Azure DevOps work SSH key
```

Your Azure clone URL looks like this:

```text
abcdef@vs-ssh.visualstudio.com:v3/...
```

So the SSH config must use:

```text
vs-ssh.visualstudio.com
```

not:

```text
ssh.dev.azure.com
```

---

## Step 3: Configure Directory-Based Git Identity

Edit main Git config:

```bash
vim ~/.gitconfig
```

Add this:

```gitconfig
[core]
    editor = vim
[pull]
    ff = only

# Path-based overrides MUST come last so they win over the default [user] above.
[includeIf "gitdir:~/data/code/personal/"]
    path = ~/.gitconfig-personal
[includeIf "gitdir:~/data/code/defide/"]
    path = ~/.gitconfig-defide
```

Explanation:

```text
Repos inside ~/data/code/personal/ use personal Git identity
Repos inside ~/data/code/defide/ use work Git identity
```

IMPORTANT: Order matters. Git config is "last value wins". The `[includeIf]`
blocks MUST appear AFTER any default `[user]` block. If a hardcoded `[user]`
comes after the includes, it overrides them and every repo uses that identity
regardless of directory.

---

## Step 4: Create Personal Git Config

Edit:

```bash
vim ~/.gitconfig-personal
```

Add:

```gitconfig
[user]
    name = Duong Trieu
    email = kltduong@gmail.com
```

---

## Step 5: Create Work Git Config

Edit:

```bash
vim ~/.gitconfig-defide
```

Add:

```gitconfig
[user]
    name = Duong Kieu
    email = duong.kieu@defide-ix.com
```

---

## Step 6: Create Project Directories

Run:

```bash
mkdir -p ~/data/code/personal
mkdir -p ~/data/code/defide
```

---

## Step 7: Clone GitHub Personal Repositories

Go to the personal directory:

```bash
cd ~/data/code/personal
```

Clone normally:

```bash
git clone git@github.com:OWNER/REPO.git
```

Example:

```bash
git clone git@github.com:kltduong/my-project.git
```

You do **not** need this anymore:

```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/github--kltduong@gmail.com -o IdentitiesOnly=yes"
```

---

## Step 8: Clone Azure DevOps Work Repositories

Go to the work directory:

```bash
cd ~/data/code/defide
```

Clone normally using the Azure URL you already have:

```bash
git clone abcdef@vs-ssh.visualstudio.com:v3/...
```

Example format:

```bash
git clone abcdef@vs-ssh.visualstudio.com:v3/ORG/PROJECT/REPO
```

You do **not** need this anymore:

```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/azure--duong.kieu@defide-ix.com -o IdentitiesOnly=yes"
```

---

## Step 9: Daily Usage

After cloning, use Git normally:

```bash
git status
git pull
git push
git commit
```

No extra SSH command is needed.

---

## Step 10: Test Personal GitHub Repo

Go inside a personal repo:

```bash
cd ~/data/code/personal/REPO
```

Check email:

```bash
git config --show-origin user.email
```

Expected:

```text
kltduong@gmail.com
```

Check name:

```bash
git config --show-origin user.name
```

Expected:

```text
Duong Trieu
```

Test SSH:

```bash
ssh -T git@github.com
```

Expected result is usually similar to:

```text
Hi kltduong! You've successfully authenticated...
```

---

## Step 11: Test Azure DevOps Repo

Go inside a work repo:

```bash
cd ~/data/code/defide/REPO
```

Check email:

```bash
git config --show-origin user.email
```

Expected:

```text
duong.kieu@defide-ix.com
```

Check name:

```bash
git config --show-origin user.name
```

Expected:

```text
Duong Kieu
```

Test SSH using your Azure host:

```bash
ssh -T abcdef@vs-ssh.visualstudio.com
```

Azure may not show the same friendly message as GitHub, but it should not fail with `Permission denied`.

---

## Step 12: Important Rule

This setup separates two jobs:

```text
~/.ssh/config
    Chooses SSH key based on host

~/.gitconfig includeIf
    Chooses Git name/email based on directory
```

So:

```text
GitHub host                  -> GitHub SSH key
vs-ssh.visualstudio.com host -> Azure SSH key

~/data/code/personal/        -> personal Git name/email
~/data/code/defide/          -> work Git name/email
```

---

## Step 13: Do Not Put These in `.zshrc`

Do **not** put this in `.zshrc`:

```bash
eval "$(ssh-agent -s)"
```

Do **not** put this in `.zshrc`:

```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/github--kltduong@gmail.com -o IdentitiesOnly=yes"
```

Do **not** put this in `.zshrc`:

```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/azure--duong.kieu@defide-ix.com -o IdentitiesOnly=yes"
```

With `~/.ssh/config`, this is not needed.

---

## Step 14: Use SSH URLs, Not HTTPS URLs

Good GitHub URL:

```bash
git@github.com:OWNER/REPO.git
```

Good Azure URL for your case:

```bash
abcdef@vs-ssh.visualstudio.com:v3/ORG/PROJECT/REPO
```

Not ideal for this setup:

```bash
https://github.com/OWNER/REPO.git
```

```bash
https://dev.azure.com/ORG/PROJECT/_git/REPO
```

If an existing repo uses HTTPS, check it:

```bash
git remote -v
```

Change GitHub remote to SSH:

```bash
git remote set-url origin git@github.com:OWNER/REPO.git
```

Change Azure remote to SSH:

```bash
git remote set-url origin abcdef@vs-ssh.visualstudio.com:v3/ORG/PROJECT/REPO
```

---

## Final File Structure

```text
~/.ssh/
    github--kltduong@gmail.com
    github--kltduong@gmail.com.pub
    azure--duong.kieu@defide-ix.com
    azure--duong.kieu@defide-ix.com.pub
    config

~/.gitconfig
~/.gitconfig-personal
~/.gitconfig-defide

~/data/code/personal/
    github-repo-1/
    github-repo-2/

~/data/code/defide/
    azure-repo-1/
    azure-repo-2/
```

---

## Final Workflow

Personal GitHub:

```bash
cd ~/data/code/personal
git clone git@github.com:OWNER/REPO.git
```

Work Azure DevOps:

```bash
cd ~/data/code/defide
git clone abcdef@vs-ssh.visualstudio.com:v3/ORG/PROJECT/REPO
```

After cloning:

```bash
git pull
git push
git commit
```

Everything should work automatically.
