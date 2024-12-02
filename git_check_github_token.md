To check the access and scopes of a Personal Access Token (PAT) in GitHub, follow these steps:

---

### 1. Navigate to Personal Access Tokens
1. Log in to your GitHub account.
2. Click on your profile picture in the top-right corner and select **Settings**.
3. Scroll down to **Developer settings** (on the left-hand side).
4. Under **Personal access tokens**, select **Tokens (classic)**.

---

### 2. View Existing Tokens
Here, you'll see a list of your generated tokens (if any). For each token, you can see:
- **Token Name**: A descriptive name for the token.
- **Creation Date**: When the token was created.
- **Expiration**: The expiry date, if any.

---

### 3. Check Scopes of a Token
Unfortunately, GitHub does not allow you to view the scopes of an existing token for security reasons. However, you can see the scopes when you initially create the token. If you no longer remember the scopes and need to confirm them, you should regenerate a new token with the required scopes.

---

### 4. Revoking or Regenerating a Token
If you suspect the token has insufficient access or has been compromised:
1. In the same **Tokens (classic)** section, click **Delete** next to the token to revoke it.
2. Generate a new token with the required scopes.

---

### 5. Token Scopes
When creating or regenerating a token, you'll see a list of scopes to grant specific permissions:
- **repo**: Full control of private repositories.
- **read:org**: Read-only access to organization details.
- **workflow**: Access GitHub Actions.
- **admin:org**: Administrative access to organizations.

Select the scopes that match the required access for your workflow.

---

For **fine-grained personal access tokens** (a newer feature), you can also manage and view token access more granularly by specifying repository and permission levels directly. These are available under the **Fine-grained personal access tokens** section in **Developer settings**.
