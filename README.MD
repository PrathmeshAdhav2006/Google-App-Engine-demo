# Deploy a Python Hello World API on Google App Engine

A complete beginner-friendly guide with separate instructions for **Windows** and **Ubuntu (Linux)**.

By the end you will have a live API at `https://YOUR_PROJECT_ID.REGION_CODE.r.appspot.com` that returns:

```json
{"message": "Hello, World!"}
```

---

## Table of Contents

1. [Basics: What are we doing?](#1-basics-what-are-we-doing)
2. [Part A: Google Cloud setup (same for everyone)](#2-part-a-google-cloud-setup-same-for-everyone)
3. [Part B: Install tools](#3-part-b-install-tools)
   - [Windows](#31-windows)
   - [Ubuntu](#32-ubuntu)
4. [Part C: Create the project files](#4-part-c-create-the-project-files)
5. [Part D: Test locally](#5-part-d-test-locally)
6. [Part E: Log in and configure gcloud](#6-part-e-log-in-and-configure-gcloud)
7. [Part F: Create the App Engine app](#7-part-f-create-the-app-engine-app)
8. [Part G: Deploy](#8-part-g-deploy)
9. [Part H: Open and verify](#9-part-h-open-and-verify)
10. [Updating your app](#10-updating-your-app)
11. [Troubleshooting (real errors and fixes)](#11-troubleshooting-real-errors-and-fixes)
12. [Cost control and cleanup](#12-cost-control-and-cleanup)
13. [Command cheat sheet](#13-command-cheat-sheet)

---

## 1. Basics: What are we doing?

| Term | Meaning |
|------|---------|
| **Google Cloud Platform (GCP)** | Google's cloud service where your app will run. |
| **Project** | A container on GCP that holds your resources. It has a **Project ID** (text, like `my-hello-12345`) and a **Project Number** (digits, like `123456789012`). |
| **App Engine** | A service that runs your code without you managing servers. |
| **gcloud CLI** | A command-line tool used to control GCP from your computer. |
| **Flask** | A small Python web framework used to build the API. |
| **gunicorn** | A production web server that runs Flask on Google's servers. |
| **app.yaml** | The config file that tells App Engine how to run your app. |

**Flow:** write code, then run `gcloud app deploy`, then Google builds and hosts it, and you get a public URL.

---

## 2. Part A: Google Cloud setup (same for everyone)

1. Go to <https://console.cloud.google.com> and sign in with a Google account.
2. Create a project:
   - Click the project picker at the top, then **New Project**.
   - Give it a name (for example `hello-api-demo`).
   - Click **Create**.
3. Note your **Project ID**. Open the project picker again and look at the **ID** column (not the number).
4. Enable **billing** for the project:
   - Menu, then **Billing**, then link a billing account.
   - App Engine requires billing to be enabled. A tiny Hello World app normally stays inside the free tier, but you must still link a billing account.

> **Important:** Always use the **Project ID** (text) in commands, never the Project Number.

---

## 3. Part B: Install tools

You need two things: **Python 3** and the **Google Cloud CLI (gcloud)**.

### 3.1 Windows

**Install Python**

1. Download from <https://www.python.org/downloads/windows/>.
2. Run the installer and **tick "Add Python to PATH"** on the first screen.
3. Open **PowerShell** and verify:
   ```powershell
   python --version
   ```
   If `python` is not recognized, try `py --version` and use `py` in place of `python` throughout this guide.

**Install the Google Cloud CLI**

1. Download the installer: <https://cloud.google.com/sdk/docs/install#windows>
2. Run `GoogleCloudSDKInstaller.exe` and accept the defaults.
3. **Close and reopen PowerShell** (so `gcloud` is found).
4. Verify:
   ```powershell
   gcloud --version
   ```

### 3.2 Ubuntu

**Install Python and tools**

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv curl apt-transport-https ca-certificates gnupg
python3 --version
```

**Install the Google Cloud CLI (apt method)**

```bash
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list

sudo apt update
sudo apt install -y google-cloud-cli
gcloud --version
```

**Alternative (snap):**

```bash
sudo snap install google-cloud-cli --classic
```

> If the commands above change, the official instructions are at <https://cloud.google.com/sdk/docs/install#deb>.

---

## 4. Part C: Create the project files

You need a folder with exactly **three files**.

### Step 1: Create the folder

**Windows (PowerShell)**
```powershell
mkdir hello-api
cd hello-api
```

**Ubuntu (Terminal)**
```bash
mkdir hello-api
cd hello-api
```

### Step 2: Create the files

Use any editor: VS Code, Notepad, `nano`, and so on.

- **Windows Notepad users:** in "Save as", set **Save as type: All Files** so it does not add `.txt`.
- **Ubuntu:** use `nano main.py`, paste, then press `Ctrl+O`, `Enter`, `Ctrl+X`.

#### `main.py`

```python
from flask import Flask, jsonify

app = Flask(__name__)


@app.route("/")
def hello():
    return jsonify(message="Hello, World!")


if __name__ == "__main__":
    # Only used for local testing. On App Engine, gunicorn runs the app.
    app.run(host="127.0.0.1", port=8080, debug=True)
```

#### `requirements.txt`

```
Flask==3.0.3
gunicorn==22.0.0
```

#### `app.yaml`

```yaml
runtime: python312
entrypoint: gunicorn -b :$PORT main:app

automatic_scaling:
  max_instances: 2
```

What each line means:

- `runtime: python312` tells App Engine to use Python 3.12. (If it is ever rejected, check the supported runtimes in the App Engine docs.)
- `entrypoint` is the command that starts your app. `main:app` means "file `main.py`, variable `app`".
- `max_instances: 2` limits how many copies of your app can run, which keeps costs under control.

### Your folder should look like this

```
hello-api/
├── main.py
├── requirements.txt
└── app.yaml
```

Verify with `dir` (Windows) or `ls` (Ubuntu). On Windows, make sure the files are not named `app.yaml.txt`. Turn on **File name extensions** in File Explorer, under View.

---

## 5. Part D: Test locally

This step is optional but recommended.

### Windows (PowerShell)

```powershell
python -m venv venv
venv\Scripts\Activate.ps1
pip install Flask
python main.py
```

If activation is blocked:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Then try activating again.

> Do **not** run gunicorn locally on Windows; it does not work there. It is only used on Google's servers.

### Ubuntu (Terminal)

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python main.py
```

(On Ubuntu you can also test with gunicorn: `gunicorn -b :8080 main:app`.)

### Check it

Open <http://127.0.0.1:8080> in a browser. You should see:

```json
{"message":"Hello, World!"}
```

Stop the server with `Ctrl+C`, then run `deactivate` to leave the virtual environment.

---

## 6. Part E: Log in and configure gcloud

The commands are the same on both systems.

### Step 1: Log in

```bash
gcloud auth login
```

A browser opens. Sign in and click **Allow**.

> **Ubuntu on a server with no browser?** Use `gcloud auth login --no-launch-browser`, open the printed link on any device, and paste the code back.

### Step 2: List your projects and find the Project ID

```bash
gcloud projects list
```

Example output:

```
PROJECT_ID          NAME            PROJECT_NUMBER
harness-poc-510805  harness-poc     123456789012
```

Copy the value from **PROJECT_ID**.

### Step 3: Set the active project (use the ID, not the number)

```bash
gcloud config set project YOUR_PROJECT_ID
```

### Step 4: Confirm

```bash
gcloud config get-value project
```

It must print a text ID, not a number.

---

## 7. Part F: Create the App Engine app

Run these once per project.

### Step 1: Enable the required APIs

```bash
gcloud services enable appengine.googleapis.com cloudbuild.googleapis.com
```

### Step 2: Create the App Engine application

```bash
gcloud app create --region=asia-south1
```

Choose a region near your users:

| Region | Location |
|--------|----------|
| `asia-south1` | Mumbai, India |
| `asia-southeast1` | Singapore |
| `europe-west1` | Belgium |
| `us-central` | Iowa, USA |

> **Warning:** The region **cannot be changed** after creation.

If you see an error that the project value is a project number, go to [Troubleshooting](#error-1-project-is-set-to-project-number).

### Step 3: Grant permissions (do this proactively on new projects)

Newer Google Cloud projects often have restricted default permissions, which causes a build error on first deploy (see [Error 2](#error-2-service-account-does-not-have-access-to-the-bucket)). Granting these roles up front avoids it.

**Windows (PowerShell)**

```powershell
$PROJECT = "YOUR_PROJECT_ID"
$SA = "$PROJECT@appspot.gserviceaccount.com"

gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/cloudbuild.builds.builder"
gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/storage.objectAdmin"
gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/artifactregistry.writer"
gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/logging.logWriter"
```

**Ubuntu (Bash)**

```bash
PROJECT="YOUR_PROJECT_ID"
SA="$PROJECT@appspot.gserviceaccount.com"

gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/cloudbuild.builds.builder"
gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/storage.objectAdmin"
gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/artifactregistry.writer"
gcloud projects add-iam-policy-binding $PROJECT --member="serviceAccount:$SA" --role="roles/logging.logWriter"
```

Each command prints the updated policy. That is normal. Wait about **1 minute** for the permissions to take effect.

---

## 8. Part G: Deploy

Make sure you are **inside the `hello-api` folder** (where `app.yaml` is).

```bash
gcloud app deploy
```

1. It shows a summary of what will be deployed.
2. Type `Y` and press Enter.
3. Wait a few minutes. At the end you should see `Deployed service [default]`.

---

## 9. Part H: Open and verify

```bash
gcloud app browse
```

This opens your app in the browser. To only print the URL:

```bash
gcloud app browse --no-launch-browser
```

Your URL looks like `https://YOUR_PROJECT_ID.REGION_CODE.r.appspot.com`.

You should see:

```json
{"message":"Hello, World!"}
```

You can also test from the terminal:

**Ubuntu / Windows 10+ (PowerShell has `curl.exe`)**
```bash
curl https://YOUR_PROJECT_ID.REGION_CODE.r.appspot.com
```

---

## 10. Updating your app

1. Edit `main.py`, for example add a new route:
   ```python
   @app.route("/api/hello/<name>")
   def hello_name(name):
       return jsonify(message=f"Hello, {name}!")
   ```
2. Redeploy from the same folder:
   ```bash
   gcloud app deploy
   ```
3. Visit `/api/hello/Sam` on your URL.

Each deploy creates a new **version**. Old versions keep existing until you delete them (see cleanup).

---

## 11. Troubleshooting (real errors and fixes)

### Error 1: Project is set to project number

```
ERROR: (gcloud.app.create) The value of ``core/project'' property is set to project number.
```

**Cause:** You set the project using the numeric Project Number.

**Fix:**
```bash
gcloud projects list
gcloud config set project YOUR_PROJECT_ID
gcloud config get-value project
```
Use the text ID from the `PROJECT_ID` column. Alternatively: `gcloud app create --region=asia-south1 --project=YOUR_PROJECT_ID`.

---

### Error 2: Service account does not have access to the bucket

```
ERROR: (gcloud.app.deploy) Error Response: [13] an internal error has occurred
... invalid bucket "staging.YOUR_PROJECT_ID.appspot.com";
service account YOUR_PROJECT_ID@appspot.gserviceaccount.com does not have access to the bucket
```

**Cause:** The App Engine default service account lacks permission to read the uploaded files during the build.

**Fix:** Run the commands from [Part F, Step 3](#7-part-f-create-the-app-engine-app), wait 1 minute, and run `gcloud app deploy` again.

If it still fails, grant bucket access directly:

```bash
gcloud storage buckets add-iam-policy-binding gs://staging.YOUR_PROJECT_ID.appspot.com --member="serviceAccount:YOUR_PROJECT_ID@appspot.gserviceaccount.com" --role="roles/storage.admin"
```

Also check:
- You have the **Owner** role on the project.
- Your company or organization has no policy blocking IAM changes. If it does, ask your cloud admin.

---

### Error 3: Billing not enabled

**Fix:** In the Cloud Console go to Billing and link a billing account to the project.

---

### Error 4: `gcloud` is not recognized (Windows)

**Fix:** Close and reopen PowerShell. If that does not help, open **Google Cloud SDK Shell** from the Start menu, or re-run the installer.

### Error 5: `gcloud: command not found` (Ubuntu)

**Fix:** Re-run the install steps. If you used the snap, make sure `/snap/bin` is in your `PATH`. Then open a new terminal.

---

### Error 6: "Unable to find app.yaml" / "Descriptor not found"

**Cause:** You are in the wrong folder, or the file is misnamed (`app.yaml.txt`).

**Fix:** Run `dir` (Windows) or `ls` (Ubuntu) and make sure `app.yaml` is listed in your current folder.

---

### Error 7: 500 Internal Server Error or 502 after deploy

**Fix:** Read the logs:

```bash
gcloud app logs tail -s default
```

Common causes:
- Typo in `main.py`.
- `main:app` in `app.yaml` does not match your filename or variable name.
- Missing `gunicorn` or `Flask` in `requirements.txt`.

---

### Error 8: Runtime not supported

**Fix:** Change `runtime:` in `app.yaml` to a currently supported Python version (see the App Engine docs for the list), then redeploy.

---

### Error 9: PowerShell blocks venv activation (Windows)

**Fix:**
```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

---

### Warning: `max_instances` default

```
WARNING: You might be using automatic scaling ... without providing a value for automatic_scaling.max_instances
```

Harmless. It disappears once you add `automatic_scaling: max_instances` to `app.yaml` (already included in this guide).

---

## 12. Cost control and cleanup

- Keep `max_instances` low in `app.yaml`.
- Set a **budget alert** in the Console: Billing, then Budgets & alerts.
- List versions: `gcloud app versions list`
- Delete an old version:
  ```bash
  gcloud app versions delete VERSION_ID
  ```
- **If this was only a test, delete the whole project** to guarantee no more charges:
  ```bash
  gcloud projects delete YOUR_PROJECT_ID
  ```
  (This is permanent after the recovery period. Make sure you want that.)

---

## 13. Command cheat sheet

| Task | Command |
|------|---------|
| Check gcloud version | `gcloud --version` |
| Log in | `gcloud auth login` |
| List projects | `gcloud projects list` |
| Set project | `gcloud config set project PROJECT_ID` |
| Show current project | `gcloud config get-value project` |
| Enable APIs | `gcloud services enable appengine.googleapis.com cloudbuild.googleapis.com` |
| Create app (once) | `gcloud app create --region=asia-south1` |
| Deploy | `gcloud app deploy` |
| Open in browser | `gcloud app browse` |
| Print URL only | `gcloud app browse --no-launch-browser` |
| Live logs | `gcloud app logs tail -s default` |
| List versions | `gcloud app versions list` |
| Delete a version | `gcloud app versions delete VERSION_ID` |

---

## Quick Windows vs Ubuntu differences

| Item | Windows | Ubuntu |
|------|---------|--------|
| Terminal | PowerShell | Bash |
| Python command | `python` (or `py`) | `python3` |
| Activate venv | `venv\Scripts\Activate.ps1` | `source venv/bin/activate` |
| Install gcloud | `.exe` installer | `apt` or `snap` |
| Run gunicorn locally | Not supported | Supported |
| Set variable | `$PROJECT = "id"` | `PROJECT="id"` |

---

**Done!** You now have a repeatable process for deploying any simple Python API to Google App Engine.