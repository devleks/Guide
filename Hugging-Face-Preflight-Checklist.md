## Dataset Upload — Pre-Flight Checklist

### 1. Data Audit & Format
- [ ] **Identify data type**: images, text, audio, CSV, JSON, JSONL, Parquet, etc.
- [ ] **Use hub-compatible formats**: Prefer Parquet (best compression, rich typing, large dataset support). Use CSV/JSONL for smaller tabular data. Compress text files (`.zip` or `.gz`) if needed.
- [ ] **Check file extensions**: Ensure files use supported extensions (`.csv`, `.json`, `.jsonl`, `.parquet`, `.txt`, `.mp3`, `.jpg`, etc.).
- [ ] **Avoid custom/proprietary formats** unless documented thoroughly.

### 2. Repository Structure & Naming
- [ ] **File naming for splits**: Use split names (`train`, `test`, `validation`) delimited by non-word characters.
  - Valid: `train.csv`, `my_train_file.csv`, `train1.csv`
  - Invalid: `trainfile.csv` (won't be detected)
- [ ] **Directory splits**: Or organize into folders named `train/`, `test/`, `validation/`.
- [ ] **Equivalent split names allowed**: `dev` = validation, `eval` = test, `training` = train.
- [ ] **Custom splits**: If not using train/test/validation, use the format: `splitname-00000-of-00001.csv`.
- [ ] **No more than 10,000 files per folder** — this is a critical hard limit.
- [ ] **Include a `README.md`** dataset card at the root (required for the Hub page).

### 3. Local Validation (Critical)
- [ ] **Test loading locally before uploading**:
  ```python
  from datasets import load_dataset
  dataset = load_dataset("imagefolder", data_dir="./data")  # or csv, json, etc.
  print(dataset)
  ```
- [ ] **Verify features**: Check that column types and special features (e.g., `Image`, `Audio`) work correctly.
  ```python
  print(dataset[0])               # Check first example
  print(dataset.features)         # Verify feature types
  ```
- [ ] **For image datasets**: Ensure `metadata.csv` has a `file_name` column with relative paths (no leading slashes).
- [ ] **Check dataset size**: Confirm `len(dataset)` matches expectations.

### 4. Size & Storage Limits
- [ ] **Repository limits**: Respect HF storage/file count limits. For very large datasets (>100GB or >10k files), plan to use `upload_large_folder()` rather than the UI.
- [ ] **Shard large datasets**: If using `push_to_hub()`, set `max_shard_size="500MB"` or `"5GB"` to avoid memory/timeout issues.

### 5. Authentication
- [ ] **Login**: Run `huggingface-cli login` or ensure `HF_TOKEN` is set.
- [ ] **Token permissions**: Confirm your token has write access.

### 6. Upload Method Selection

| Scenario | Recommended Method |
|---|---|
| Small files (<1GB), simple format | Hub UI drag-and-drop |
| Built-in loader available | `load_dataset()` → `push_to_hub()` |
| Large files (>100GB) / >10k files | `upload_large_folder()` |
| Streaming media | WebDataset format |

### 7. Post-Upload Verification
- [ ] **Dataset Viewer**: Check `https://huggingface.co/datasets/username/dataset-name` — wait 5–10 min for processing.
- [ ] **Test remote loading**: `load_dataset("username/dataset-name")`
- [ ] **Verify features preserved**: `print(dataset.features)`
- [ ] **Troubleshoot viewer**: If broken, check README YAML for `viewer: false`, missing `config_name`, or incorrect file naming patterns.

---

## Space/App Deployment — Pre-Flight Checklist

### 1. Project Structure
- [ ] **`app.py` at root**: Must be in the root directory (default entry point). If using another name, configure `app_file` in README YAML frontmatter.
- [ ] **`requirements.txt` present**: All dependencies listed with pinned versions where possible.
- [ ] **`README.md` present**: Required for Space configuration (YAML frontmatter with `sdk`, `sdk_version`, etc.).
- [ ] **No missing modules**: Ensure all imported files/submodules are included in the repo.

### 2. Local Testing (Do This First)
- [ ] **Install dependencies**: `pip install -r requirements.txt`
- [ ] **Run locally**: `python app.py`
- [ ] **Test in browser**: Open `http://localhost:7860` (or your app's port) and verify all features work.
- [ ] **No syntax errors**: Check that all imports resolve and the Gradio/Streamlit/FastAPI app launches correctly.

### 3. Space Configuration (README.md YAML)
- [ ] **SDK declared**: `sdk: gradio` (or `docker`, `static`)
- [ ] **Python version**: Set appropriately (e.g., `python_version: "3.11"`)
- [ ] **App file path**: If not `app.py`, set `app_file: "main.py"`
- [ ] **Preload models** (optional): Use `preload_from_hub` to download large models during build time instead of at runtime.

### 4. Code Quality & Dependencies
- [ ] **No hardcoded secrets**: API keys/tokens must NOT be in the code. Use Space Settings → Secrets/Variables.
- [ ] **Environment variables configured**: Add any needed `HF_TOKEN`, API keys, or config values in the Space settings before/after push.
- [ ] **Docker check** (if using Docker): `Dockerfile` must be named exactly `Dockerfile`, expose port `7860`, and use proper startup commands.
- [ ] **Hardware requirements**: Confirm your app fits in the selected tier (CPU basic = 16GB RAM free tier). Upgrade if using large models.

### 5. Git & File Handling
- [ ] **File size check**: Any file >10MB must be tracked with Git LFS. Verify `.gitattributes` is configured.
- [ ] **`.gitignore`**: Exclude `.env`, local cache, `__pycache__`, virtual envs.
- [ ] **Clean git status**: Ensure no uncommitted changes or large accidentally-committed binaries.

### 6. Authentication & Permissions
- [ ] **HF CLI login**: `huggingface-cli login` with a write token.
- [ ] **Organization uploads**: If pushing to an org repo, specify `organization/repo_id` explicitly.

### 7. Deployment Steps
- [ ] **Create Space** on huggingface.co/spaces (select correct SDK and hardware).
- [ ] **Clone the Space repo**: `git clone https://huggingface.co/spaces/username/space-name`
- [ ] **Copy files**: `app.py`, `requirements.txt`, `README.md`, and any supporting files.
- [ ] **Commit and push**:
  ```bash
  git add .
  git commit -m "Initial deployment"
  git push
  ```

### 8. Post-Deployment Verification
- [ ] **Build logs**: Check the "Logs" tab for dependency/installation errors.
- [ ] **App loads**: Visit the Space URL and test the UI.
- [ ] **Health checks**: If implemented, test `/health` or similar endpoints.
- [ ] **Secrets loaded**: Verify any API-dependent features work (proves secrets are injected).
- [ ] **Sleep behavior**: Remember free-tier Spaces go to sleep after inactivity — this is normal.

---

## Common Gotchas Reference

| Issue | Prevention |
|---|---|
| Dataset viewer not showing | Wait 5–10 min; check split naming delimiters; verify no `viewer: false` in README |
| Repository not found / auth errors | Re-run `huggingface-cli login`; check token has write permission |
| Build fails (Space) | Pin dependencies; ensure `app.py` exists at root; check Docker port is 7860 |
| Out of memory on push | Use `max_shard_size="500MB"` for datasets; reduce model size or upgrade hardware for Spaces |
| File too large for git | Use Git LFS for files >10MB; use `upload_large_folder()` for massive datasets |
