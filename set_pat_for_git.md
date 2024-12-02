To set a Personal Access Token (PAT) for Git on Windows so that it is stored securely and used automatically for Git actions, follow these steps:

---

### 1. Configure a Credential Manager
Windows supports Git Credential Manager (GCM) to securely store your credentials, including PATs. If you're using Git for Windows, GCM is likely installed by default.

#### Check if Credential Manager is Enabled
Run the following command to check:
```cmd
git config --global credential.helper
```

If it returns `manager-core` or similar, GCM is enabled. If not, enable it:
```cmd
git config --global credential.helper manager-core
```

---

### 2. Push or Pull Once to Trigger Credential Storage
1. Perform a Git action (e.g., `git push`).
2. When prompted for your credentials:
   - Use your GitHub **username** for the username field.
   - Use the **Personal Access Token (PAT)** as the password.

The credential manager will securely store the PAT, and you won't need to input it again for subsequent Git actions.

---

### 3. Alternative: Manually Store Credentials
If you prefer to store the credentials manually (not recommended for shared systems), you can use the `store` helper.

#### Set the Credential Helper to `store`:
```cmd
git config --global credential.helper store
```

#### Perform a Git Action:
The next time you run `git push`, enter your username and PAT when prompted. These will be saved in plaintext in a file located in your user directory (`~/.git-credentials` on Windows).

---

### 4. Directly Set the Remote URL with PAT
Another way is to embed the PAT in the remote URL (use this method with caution, especially on shared systems):

```cmd
git remote set-url origin https://<PAT>@github.com/USERNAME/REPOSITORY.git
```

- Replace `<PAT>` with your token, `USERNAME` with your GitHub username, and `REPOSITORY` with your repository name.

---

### 5. Verify
After completing any of the above methods, perform a Git action like `git pull` or `git push`. It should work without asking for credentials.

For maximum security, use the credential manager (`manager-core`). It integrates with Windows and stores your PAT securely.
