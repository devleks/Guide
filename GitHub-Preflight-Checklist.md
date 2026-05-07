## Security & Secrets (Critical — Do First)

- [ ] **Scan for secrets**: Run a tool like `git-secrets`, `truffleHog`, or GitHub secret scanning locally:
  ```bash
  # Example with truffleHog
  truffleHog filesystem .
  ```
- [ ] **Check for API keys and tokens**: Search your codebase for patterns like:
  - `api_key`, `apikey`, `password`, `secret`, `token`
  - `AWS_ACCESS_KEY_ID`, `OPENAI_API_KEY`, `HF_TOKEN`
  - Hardcoded database connection strings
- [ ] **Move secrets to environment variables**: Use `.env` files (and add them to `.gitignore`).
- [ ] **Check committed history**: If you ever committed a secret, assume it is leaked. Rotate the credential immediately, then purge it from Git history with `git filter-repo` or BFG Repo-Cleaner.
- [ ] **No personal info**: Remove names, emails, phone numbers, or internal hostnames that shouldn't be public.

## Repository Structure & Files

- [ ] **Logical project layout**:
  ```
  my-app/
  ├── src/ or app/           # Application code
  ├── data/                  # Sample/public data (if small)
  ├─��� tests/                 # Test suite
  ├── docs/                  # Documentation
  ├── scripts/               # Utility scripts
  ├── .github/               # Workflows, issue templates
  ├── .gitignore             # Ignore rules
  ├── README.md              # Main documentation
  ├── LICENSE                # License file
  ├── requirements.txt       # or pyproject.toml, package.json, etc.
  └── Dockerfile (if needed)
  ```
- [ ] **Entry point clear**: Ensure the main executable file (e.g., `app.py`, `main.py`, `index.js`) is obvious or documented.
- [ ] **No absolute paths**: Replace any local file paths (e.g., `/Users/yourname/...`, `C:\Users\...`) with relative paths or environment variables.
- [ ] **No OS-specific files**: Remove `.DS_Store`, `Thumbs.db`, `desktop.ini`.

## .gitignore Audit

- [ ] **Environment files**: `.env`, `.env.local`, `.env.production`
- [ ] **Virtual environments**: `venv/`, `.venv/`, `env/`, `node_modules/`
- [ ] **IDE / editor files**: `.vscode/`, `.idea/`, `*.swp`, `*.swo`
- [ ] **Compiled / cached files**:
  - Python: `__pycache__/`, `*.pyc`, `*.pyo`, `*.egg-info/`, `.pytest_cache/`, `.mypy_cache/`
  - JS/TS: `dist/`, `build/`, `.next/`, `*.tsbuildinfo`
  - Java: `target/`, `*.class`
- [ ] **Data / model artifacts** (if large): `*.pt`, `*.pth`, `*.h5`, `*.ckpt`, `checkpoints/`, `*.sqlite` (unless it's a small demo DB)
- [ ] **Logs**: `*.log`, `logs/`
- [ ] **Secrets**: `*.pem`, `*.key`, `secrets/`, `credentials/`

## Dependencies & Environment

- [ ] **Lockfile present and up to date**:
  - Python: `requirements.txt` (pinned versions recommended: `package==1.2.3`) or `poetry.lock` / `Pipfile.lock`
  - Node: `package-lock.json` or `yarn.lock`
  - Rust: `Cargo.lock`
- [ ] **No unused dependencies**: Prune packages you aren't using (reduces attack surface and install time).
- [ ] **Python version declared**: Include `.python-version` (pyenv) or specify `python_requires` in `pyproject.toml`.
- [ ] **Node version declared**: Include `.nvmrc` or specify `engines` in `package.json`.

## Data Handling (If Uploading Data to GitHub)

- [ ] **Size check — GitHub hard limits**:
  - **100 MB per file** (hard limit via web/browser)
  - **~2 GB per file** (hard limit via Git LFS)
  - Repositories bloat significantly with large binary files
- [ ] **Use Git LFS for large files**: If you must track large files/models/datasets:
  ```bash
  git lfs track "*.csv"
  git lfs track "*.parquet"
  git lfs track "models/*.pt"
  ```
- [ ] **Prefer data hosting elsewhere** for large datasets:
  - Hugging Face Datasets, Kaggle, Zenodo, S3, GCP, Azure Blob
  - Provide download scripts or `wget`/`curl` commands in README instead of storing raw data
- [ ] **Sample data only**: If the full dataset is huge, commit only a small representative sample (e.g., `data/sample/`) and document how to get the full set.
- [ ] **Data license compliance**: Ensure you have rights to redistribute the data and include the data source's license if required.
- [ ] **Anonymization**: Remove PII (Personally Identifiable Information) from any datasets.
- [ ] **Sensitive data**: Never commit production databases, user data, or confidential records.

## Code Quality & Testing

- [ ] **Runs locally**: Test the full startup sequence on a clean machine or fresh virtual environment:
  ```bash
  python -m venv venv_test
  source venv_test/bin/activate
  pip install -r requirements.txt
  python app.py
  ```
- [ ] **Linting / formatting passes**:
  - Python: `ruff check .`, `black --check .`, `mypy .`
  - JS/TS: `eslint .`, `prettier --check .`
- [ ] **Tests pass**: `pytest`, `npm test`, `cargo test`, etc.
- [ ] **No debug artifacts**: Remove `console.log`, `print()`, `debugger` statements, and TODO/FIXME hacks (unless tracked in issues).

## Documentation

- [ ] **README.md is complete**:
  - [ ] Project title and one-line description
  - [ ] Installation instructions (step-by-step)
  - [ ] Usage examples (copy-paste ready)
  - [ ] Environment variables list (names only, no values)
  - [ ] Data setup/download instructions
  - [ ] Contributing guidelines (optional but recommended)
  - [ ] License badge / link
- [ ] **CHANGELOG.md** (if not the first release): Document what's new/changed/fixed.
- [ ] **LICENSE file**: Choose an appropriate open-source license (MIT, Apache-2.0, GPL, etc.) or mark it proprietary if private.

## CI/CD & Automation (Optional but Recommended)

- [ ] **GitHub Actions workflows** in `.github/workflows/`:
  - [ ] Lint / format check
  - [ ] Unit tests
  - [ ] Build verification (Docker build if applicable)
- [ ] **Workflow passes locally** (if possible): Tools like `act` can test GitHub Actions locally.
- [ ] **Branch protection rules** (for team repos): Enable via Settings → Branches before going live.

## Git Hygiene Final Checks

- [ ] **Clean working tree**:
  ```bash
  git status
  ```
  Only files you intend to commit should be staged. Untracked files should either be committed or covered by `.gitignore`.
- [ ] **Commit history is clean**:
  - Use meaningful commit messages (e.g., `feat: add user authentication`, `fix: resolve data loading bug`)
  - Squash "WIP" or "fix typo" commits if desired using `git rebase -i`
- [ ] **Repository name is appropriate**: No spaces, descriptive, lowercase-with-hyphens preferred.
- [ ] **Remote URL correct**: `git remote -v` points to the right GitHub repo.
- [ ] **Repository visibility**: Double-check if it should be **Public** or **Private** before pushing.
- [ ] **Repo size sanity check**:
  ```bash
  du -sh .git
  ```
  If `.git` is unexpectedly huge, you may have accidentally committed large binaries.

## Final Push Commands

```bash
# 1. Final review of what will be pushed
git status

# 2. Review the actual diff
git diff --cached

# 3. Commit with a descriptive message
git commit -m "feat: initial app release with sample data"

# 4. Push
git push origin main
```

## Post-Push Verification

- [ ] **GitHub page renders**: Check the repo homepage for formatting errors in README.
- [ ] **Clone test**: In a fresh directory, run:
  ```bash
  git clone https://github.com/username/repo.git
  cd repo
  # Follow your own README instructions exactly
  ```
- [ ] **No secrets alert**: GitHub will email/notify if it detects a secret post-push. Act immediately if triggered.
- [ ] **Releases / Tags** (optional): If this is a versioned release, create a Git tag:
  ```bash
  git tag -a v1.0.0 -m "Release version 1.0.0"
  git push origin v1.0.0
  ```

## Top 3 Deal-Breakers to Avoid

| Mistake | Why It Hurts | Prevention |
|---|---|---|
| Committing secrets/API keys | Permanent public exposure; bots scrape these in seconds | `.gitignore` + env vars + secret scanner |
| Pushing large models/data directly | Repo becomes unusable; clones take forever | Git LFS or external hosting |
| No `requirements.txt` / lockfile | App won't run for anyone else | Generate from clean venv; include lockfile |
