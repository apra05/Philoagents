## 🔧 Troubleshooting & Common Setup Issues

We have identified common configuration issues and provided detailed solutions below.

### 1. Error: `.env` file not found
* **Symptoms**:
  `env file C:\Users\apras\Desktop\Philoagents\philoagents-api\.env not found: The system cannot find the file specified.` when running `docker compose up`.
* **Cause**: Docker Compose looks for `philoagents-api/.env` as defined in `docker-compose.yml` (`env_file: - ./philoagents-api/.env`), but it has not been created yet.
* **Solution**: Create the file from `.env.example`:
  ```powershell
  # Windows Powershell / CMD
  copy philoagents-api\.env.example philoagents-api\.env
  ```
  And populate the required API keys (e.g., `GROQ_API_KEY`, and optionally `OPENAI_API_KEY` or `COMET_API_KEY` for evaluation).

### 2. Issue: Heavy Docker Image Build Overhead (Torch / CUDA wheel download)
* **Symptoms**: When building the `api` container image, the build hangs or takes a very long time during `uv sync --frozen --no-cache` downloading large wheels (`torch`, `nvidia-*` packages totaling ~2GB).
* **Cause**: Docker builds run on a Linux-based virtual machine where `uv sync` installs the dependencies specified in `pyproject.toml`. By default, Python wheel installations for libraries like PyTorch fall back to CUDA wheels on Linux.
* **Solution**: Run the database in Docker, but run the API and UI services locally! This is the recommended lightweight developer workflow:
  1. **Start the database only**:
     ```bash
     docker compose up local_dev_atlas -d
     ```
  2. **Install and run the Backend locally**:
     ```bash
     cd philoagents-api
     uv venv
     .venv\Scripts\activate
     uv pip install -e .
     # Create the database vector memory
     uv run python -m tools.create_long_term_memory
     # Start backend server
     uv run fastapi run src/philoagents/infrastructure/api.py --port 8000
     ```
  3. **Install and run the Frontend UI locally**:
     ```bash
     cd philoagents-ui
     npm install
     npm run dev
     ```

### 3. Error: `make` Command Not Found on Windows
* **Symptoms**: Running commands like `make infrastructure-up` fails with `'make' is not recognized as an internal or external command`.
* **Cause**: GNU Make is not natively included in Windows.
* **Solution**:
  - Download and install **GnuWin32 Make** (typically installs to `C:\Program Files (x86)\GnuWin32\bin`).
  - Add GnuWin32's `bin` folder to your session path in PowerShell:
    ```powershell
    $env:PATH += ";C:\Program Files (x86)\GnuWin32\bin"
    ```
  - Alternatively, bypass `make` and run the commands directly in your terminal. For example, run `docker compose up local_dev_atlas -d` instead of `make infrastructure-up`.

### 4. WSL Requirement for Windows Users
* **Symptoms**: UNIX commands like `cp` or `source ./.venv/bin/activate` fail inside standard cmd.exe.
* **Cause**: Windows cmd/PowerShell uses different command conventions and paths.
* **Solution**: Use Windows Subsystem for Linux (WSL) for a native Linux environment, or run the equivalent Windows commands (e.g. `copy` instead of `cp`, and `.\.venv\Scripts\activate` instead of `source`).
