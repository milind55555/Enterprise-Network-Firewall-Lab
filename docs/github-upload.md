# Upload this project to GitHub

## Option A: Git command line

1. Unzip `enterprise-network-firewall-lab.zip`.
2. Create an empty repository on GitHub named `enterprise-network-firewall-lab` (do not initialize it with a README/license because this project already has those files).
3. Open a terminal in the extracted `enterprise-network-firewall-lab` directory and run:

```bash
git init -b main
git add .
git status
git commit -m "Add enterprise network and firewall lab"
git remote add origin https://github.com/YOUR_USERNAME/enterprise-network-firewall-lab.git
git push -u origin main
```

Replace `YOUR_USERNAME` with your GitHub username. Review `git status` before committing and verify that no passwords, VPN profiles, private keys, or credentials are staged.

## Option B: GitHub website

1. Create a new public or private repository on GitHub.
2. Extract the ZIP on your computer.
3. Open the repository and choose **Add file > Upload files**.
4. Upload the project contents (the top-level folders and files) and commit the upload.
5. For future updates, use Git locally rather than repeatedly uploading ZIP files.

## Before making it public

- Replace all `CHANGE_ME` placeholders only in your local lab configs; do not commit real passwords.
- Ensure `.ovpn`, `.pem`, `.key`, `.p12`, `.pfx`, and `.env` files are not committed.
- Add your actual `.pkt` file only after building and testing it.
- Add screenshots that prove the tests you completed.
- Update the README if you change subnet ranges, VLAN IDs, device models, or firewall behavior.
