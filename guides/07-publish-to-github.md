# 07 — Publish this lab on GitHub (no Git commands required)

[← Linux Agent exercise](06-next-steps-linux-agents.md) · [← Home](../README.md)

## First: what will other people see?

GitHub automatically displays `README.md` when someone visits your repository. Each `.md` guide is a clickable page. This is easier to follow than asking every beginner to open a Word document.

## Beginner method: use the GitHub website

1. Sign in to [github.com](https://github.com) and click **New repository**.
2. Name it `elasticsearch-learning-lab`. Description idea: **Beginner learning notes on a 3-node Elasticsearch cluster, Kibana, Fleet Server, and secure SSH access.**
3. Set visibility to **Public** if you want the tutorials viewable by everyone. **Don't** initialize it with a README if you're uploading this ready-made folder as its first commit.
4. Unzip the downloadable repository package on your laptop.
5. In the empty GitHub repository, click **uploading an existing file** or **Add file → Upload files**.
6. Drag the **contents** of the `elasticsearch-learning-lab` folder (not the outer folder itself) to the upload page, preserving the subfolders. If your browser struggles with folders, use Git locally instead (below).
7. Write your commit message: `Add beginner learning lab documentation` → **Commit changes**.
8. Open your public repository page. The README should appear; click into **guides/** to verify links work.

### Easier for large folder structures: Git from the terminal

Create an **empty GitHub repository first**, then in your extracted folder:

```bash
git init
git add .
git status        # REVIEW: make sure no secrets or private files are listed
git commit -m "Add beginner learning lab documentation"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/elasticsearch-learning-lab.git
git push -u origin main
```

Requires Git to be installed and GitHub authentication (e.g., a credential manager). `YOUR_GITHUB_USERNAME` is a placeholder; use your own. **Never put a GitHub access token inside the remote URL or README.**

## Security checklist before pressing Commit

- [ ] No real server IP, tailnet IP, public IP, VPN hostname or account email
- [ ] No `.ppk`, `.pem`, `.key`, `.p12`, `.pfx`, `.jks`, tokens, password files or enrollment codes
- [ ] No unredacted screenshots showing your environment or credentials
- [ ] No raw agent logs or Kibana exports with secrets
- [ ] All teaching network addresses are clearly marked **examples**
- [ ] All "completed" claims match the actual lab results
- [ ] Every link in the README opens correctly

**Warning:** `.gitignore` reduces accidental additions, but it does **not** remove secrets that were already committed or uploaded. If a real secret is published, **revoke/rotate it immediately** and clean Git history according to GitHub guidance.

More help: [GitHub docs — adding files](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository).
