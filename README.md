# GitHub Actions Release Workflow

This project provides a complete GitHub Actions pipeline for managing automatic versioning, previews for pull requests, and releases with ZIP artifacts. It is designed for repositories that need structured and secure release workflows.

---

## 🧰 Workflows Included

### 1. `tag-and-release.yml`
Triggered when changes are pushed to the `master` branch (after PR merges).

**Branch naming rules to determine versioning:**
- `release/*`: increments **major** version: `X.0.0`
- `feature/*`: increments **minor** version: `X.Y.0`
- `bugfix/*`: increments **patch** version: `X.Y.Z`

**Actions:**
- Determines the new version based on branch type
- Updates the `version` field in `manifest.json`
- Commits the updated manifest
- Tags the repository with the new version
- Creates a ZIP artifact (excluding `.gitignore`, `afile.js`, and `src/another/file/toignore.html`)
- Publishes a GitHub Release containing the ZIP file

---

### 2. `pr-preview.yml`
Triggered on pull requests targeting the `master` branch from these branches:
- `feature/*`
- `bugfix/*`
- `release/*`

**Actions:**
- Generates a temporary preview ZIP artifact
- Uses current timestamp to suffix artifact name (e.g., `preview-20250607103000-b`)
- Useful for testing the artifact before merging

---

### 3. `pr-preview-cleanup.yml`
Triggered when a pull request is **closed** (merged or canceled).

**Actions:**
- Automatically deletes the preview artifact (matching pattern `preview-*`) associated with the closed PR

---

### 4. `custom-publish.yml`
Manually triggered via GitHub Actions UI.

**Inputs:**
- `version`: e.g., `2.3.1`

**Actions:**
- Updates the `manifest.json` version field
- Commits and pushes the change
- Tags the repository with the custom version
- Creates a ZIP release and publishes it

---

### 5. `cleanup-orphan-previews.yml`
Scheduled to run every Sunday at 03:00 AM (UTC) or manually.

**Actions:**
- Deletes all preview artifacts (`preview-*`) older than **3 days**
- Helps maintain a clean workspace by removing orphaned previews

---

## 📁 Installation Guide

1. **Download and extract the workflow files into your repository**:
   ```
   .github/workflows/
   ```

2. **Create a `.compact-ignore` file in your repository root** (with patterns to exclude from your ZIP/release):
   ```
   .gitignore
   afile.js
   src/another/file/toignore.html
   ```

3. **Configure `branch protection rules` in your GitHub repository settings**:
   - Protect the `master` branch
   - Require pull requests before merging
   - Prevent direct pushes

4. **Create a Personal Access Token (`PAT_TOKEN`)** with full repository permissions:

   🔐 This is necessary for the workflow to push commits and tags via GitHub Actions.

   ### Step-by-step to create the PAT:

   1. Go to your GitHub profile: [https://github.com/settings/tokens](https://github.com/settings/tokens)
   2. Click on **"Fine-grained tokens"** or **"Classic tokens"** (classic is simpler and sufficient).
   3. Click **"Generate new token"**:
      - **Token name**: `workflow-push-token` (or anything you prefer)
      - **Expiration**: Choose an appropriate expiration date or "No expiration" for long-term use
      - **Select scopes**:
        - ✅ `repo` (Full control of private repositories)
        - ✅ `workflow` (if interacting with workflows via API)
   4. Click **"Generate token"** and **copy it**. ⚠️ You will not be able to see it again!
   5. Go to your repository:
      - Acesse **Settings > Secrets and variables > Actions > New repository secret**
      - **Name**: `PAT_TOKEN`
      - **Value**: paste your generated token
      - Save

5. **Commit the workflows and push to your repository**:
   ```bash
   git add .github/workflows .compact-ignore
   git commit -m "Add GitHub Actions release workflow v3"
   git push origin main
   ```

---

## 📝 Notes

- These workflows rely on the `jq` CLI tool, which is pre-installed on GitHub-hosted runners.
- To manually publish a version using `custom-publish.yml`, go to the `Actions` tab and select the workflow.

---

## 👨‍🔧 Author & License

Maintained by ~ChatGPT~ andrebnassis

MIT License.

