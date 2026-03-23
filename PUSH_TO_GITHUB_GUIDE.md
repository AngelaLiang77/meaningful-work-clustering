# How to Push This Project to GitHub (Mac)

A step-by-step guide for first-time GitHub users.

---

## Step 1: Open Terminal

Press **Cmd + Space**, type **Terminal**, and hit Enter.

---

## Step 2: Check if Git is installed

Type this and press Enter:

```bash
git --version
```

If you see a version number (like `git version 2.39.0`), you're good — skip to Step 3.

If it asks you to install developer tools, click **Install** and wait for it to finish.

---

## Step 3: Set up your Git identity (one-time only)

Replace the name and email with your own (use the email tied to your GitHub account):

```bash
git config --global user.name "Jung Shan Liang"
git config --global user.email "itsliang900717@gmail.com"
```

---

## Step 4: Navigate to the project folder

You need to `cd` into wherever the `meaningful-work-clustering` folder is on your Mac. For example, if it's in your Downloads:

```bash
cd ~/Downloads/meaningful-work-clustering
```

To verify you're in the right place:

```bash
ls
```

You should see: `README.md`, `notebooks/`, `data/`, `reports/`, `requirements.txt`, etc.

---

## Step 5: Create a new repository on GitHub (in your browser)

1. Go to [https://github.com/new](https://github.com/new)
2. Fill in:
   - **Repository name:** `meaningful-work-clustering`
   - **Description:** `Dual clustering analysis comparing psychometric and narrative measurement of meaningful work`
   - **Visibility:** Public (so employers can see it)
   - **DO NOT** check "Add a README" or ".gitignore" (we already have these)
3. Click **Create repository**
4. You'll see a page with setup instructions — keep this page open, you'll need the URL

---

## Step 6: Initialize Git and push (back in Terminal)

Run these commands one at a time. Copy-paste each line, press Enter, and wait for it to finish before running the next one:

```bash
git init
```

```bash
git add .
```

```bash
git commit -m "Initial commit: dual clustering analysis of meaningful work"
```

```bash
git branch -M main
```

Now connect to your GitHub repo. Replace `YOUR_USERNAME` with your actual GitHub username:

```bash
git remote add origin https://github.com/YOUR_USERNAME/meaningful-work-clustering.git
```

Finally, push:

```bash
git push -u origin main
```

---

## Step 7: Authenticate

The first time you push, GitHub will ask you to log in.

**Option A — Browser pop-up (most common):**
A browser window may open asking you to authorize Git. Just click **Authorize** and follow the prompts.

**Option B — If it asks for a password in Terminal:**
GitHub no longer accepts regular passwords. You need a Personal Access Token:

1. Go to [https://github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Give it a name like "my mac"
4. Check the **repo** scope (full control of repositories)
5. Click **Generate token**
6. **Copy the token** (it starts with `ghp_...`)
7. Back in Terminal, paste the token when it asks for your password (you won't see it as you paste — that's normal)

---

## Step 8: Verify it worked!

Go to `https://github.com/YOUR_USERNAME/meaningful-work-clustering` in your browser.

You should see your README rendered beautifully with badges, the pipeline diagram, and all your project files.

---

## Bonus: Update the README

Now that your repo is live, update the clone URL in the README:

1. Open `README.md` in any text editor
2. Find `YOUR_USERNAME` and replace it with your actual GitHub username
3. Save, then run:

```bash
git add README.md
git commit -m "Update README with correct GitHub username"
git push
```

---

## Quick Reference: Common Git Commands

| What you want to do | Command |
|---|---|
| Check what's changed | `git status` |
| Stage all changes | `git add .` |
| Commit with a message | `git commit -m "your message"` |
| Push to GitHub | `git push` |
| Pull latest from GitHub | `git pull` |

---

You're all set! Your project is now live on GitHub and ready to share with employers.
