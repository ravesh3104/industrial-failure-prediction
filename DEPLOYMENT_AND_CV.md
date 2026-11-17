# 🚀 Deploy this publicly (and put it on your CV)

Two jobs in this guide:

1. **Part A** — get a public URL in ~15 minutes (free).
2. **Part B** — turn that URL into a CV entry that survives a recruiter's 30-second scan.

---

# Part A — Deploy

## Choose your platform

| Platform | Free? | Sleeps when idle? | Best for | Difficulty |
|---|---|---|---|---|
| **Streamlit Community Cloud** ⭐ | Yes | Yes (after ~15 min idle) | **Recommended.** Made for Streamlit, connects straight to GitHub, no Docker | ⭐ Easiest |
| **Hugging Face Spaces** | Yes | No (spaces stay up) | Same app, no sleep, plus a nicer landing page | ⭐⭐ Easy |
| **Render** | Yes (with card) | Yes (free tier) | More control, custom domains | ⭐⭐ Easy |
| **Railway** | Trial credit | No | Fast deploys, good logs | ⭐⭐ Easy |
| **Google Cloud Run** | Pay-per-use (generous free tier) | Scales to zero | "Real cloud" credibility on a CV | ⭐⭐⭐⭐ Advanced |

> **My recommendation:** **Streamlit Community Cloud** first (fastest, no Docker), then mirror it
> on **Hugging Face Spaces** as a backup link. Mention whichever you used on your CV — the
> platform matters far less than the fact that the URL works.

Everything below assumes the project is in a folder called `industrial-failure-prediction`.

---

## Step 1 — Make sure the repo is deploy-ready (already done for you)

The project already contains everything a host needs:

| File | Purpose |
|---|---|
| `requirements-app.txt` | **Minimal** runtime deps (5 packages, no matplotlib/Jupyter). Hosts should use this. |
| `requirements.txt` | Full deps, only for rebuilding the project locally. |
| `.python-version` | Pins `3.11` so the host picks a known-good interpreter. |
| `artifacts/` | Pre-trained models + pre-rendered charts — **committed on purpose**, so nothing is retrained at deploy time. |
| `Procfile` | Tells Heroku/Railway how to start the app. |

Verified: the app boots and all four views render with **only** pandas, numpy, scikit-learn,
joblib and streamlit installed.

> ⚠️ **Do not** add `matplotlib`, `seaborn` or `jupyter` to `requirements-app.txt`. They add
> ~150 MB, make builds slow, and are pointless at runtime — the charts are pre-rendered PNGs.

---

## Step 2 — Push to GitHub

If you've never done this, run these commands **inside the project folder**:

```bash
git init
git add .
git commit -m "Industrial equipment failure prediction: ML dashboard"
git branch -M main
```

Now create an empty repo:

1. Go to <https://github.com/new>
2. Name: `industrial-failure-prediction`
3. **Keep it Public** (required for free hosting on Streamlit Cloud)
4. **Do not** tick "Add a README" — you already have one
5. Click **Create repository**

GitHub then shows you commands like these — copy *your* exact ones and run them:

```bash
git remote add origin https://github.com/YOURNAME/industrial-failure-prediction.git
git push -u origin main
```

If it asks for a password, use a **Personal Access Token**, not your account password:
GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) →
Generate new token → tick **repo** → copy the token and paste it as the password.

✅ **Checkpoint:** refreshing your GitHub repo page shows `app/`, `src/`, `artifacts/`,
`README.md`.

> 🔒 **Before pushing anything public**, check you did not commit secrets:
> `git status` should not list `.streamlit/secrets.toml` (it's gitignored already).

---

## Step 3 — Deploy on Streamlit Community Cloud ⭐ (recommended)

1. Go to **<https://share.streamlit.io>**
2. Click **Sign in with GitHub** and authorise it.
3. Click **Create app** → **Deploy a public app from GitHub**.
4. Fill in the form:

| Field | Value |
|---|---|
| **Repository** | `YOURNAME/industrial-failure-prediction` |
| **Branch** | `main` |
| **Main file path** | `app/streamlit_app.py` |
| **App URL** | pick something like `equipment-failure-ml` |
| **Advanced settings → Python version** | **3.11** |

5. In **Advanced settings → Requirements file**, if your Python version is ≥ 3.12 you may see
   a `pyarrow` error. If so, this two-line `requirements-app.txt` override fixes it — our file
   already handles it, but on some versions you should add:

```
pyarrow>=14.0
```

6. Click **Deploy**. First build takes **3–6 minutes**.

✅ **Checkpoint:** you get a URL like

```
https://equipment-failure-ml.streamlit.app
```

Open it. You should see the dark dashboard with the pill navigation.

### If the build fails

Click **Manage app → Logs** and read the last 30 lines. The usual suspects:

| Log message | Fix |
|---|---|
| `ModuleNotFoundError: No module named 'src'` | Main file path must be `app/streamlit_app.py` (not `streamlit_app.py`). The app adds the repo root to `sys.path` itself — keep the folder structure intact. |
| `ERROR: Could not find a version that satisfies...` | A package in `requirements.txt` is too old/new. Deploy using `requirements-app.txt` instead (Settings → change the requirements file, or rename it). |
| `FileNotFoundError: artifacts/models.joblib` | `artifacts/` was not committed. Run `git add artifacts -f` (the folder may be caught by a broad ignore rule), commit, push. |
| `OSError: [Errno 98] Address already in use` | You passed `--server.port`; remove any custom command. Community Cloud manages the port. |
| Build succeeds but app is blank | Hard-refresh (`Ctrl+Shift+R`). If still blank, check **Logs** for a Python exception. |
| App "goes to sleep" | **Normal on the free tier.** First visit after idle takes ~30 s to wake. Mention this to interviewers, or pin a note in your CV link. |

---

## Step 4 (optional, recommended) — Mirror on Hugging Face Spaces

A second link costs 10 minutes and guarantees at least one is up.

1. Go to **<https://huggingface.co/new-space>**
2. Space name: `equipment-failure-prediction` · **SDK: Streamlit** · Hardware: **CPU basic (free)** ·
   Visibility: **Public**
3. Create the Space, then either:
   - **Upload files** via the web UI: drop your whole project folder, or
   - **Push via git** (replace `USER`/`SPACE`):

```bash
git remote add hf https://huggingface.co/spaces/USER/SPACE
git push hf main
```

4. HF reads `README.md` as the Space card. Add this block at the very top of the README
   so it displays properly:

```yaml
---
title: Industrial Equipment Failure Prediction
emoji: ⚙️
colorFrom: blue
colorTo: teal
sdk: streamlit
sdk_version: 1.30.0
app_file: app/streamlit_app.py
pinned: false
---
```

5. HF installs `requirements.txt` by default — that's the big one. Either rename
   `requirements-app.txt` → `requirements.txt` in the Space, or accept a slower build.

✅ **Checkpoint:** `https://huggingface.co/spaces/USER/SPACE` shows the running app.

---

## Step 5 (optional) — Password-protect the demo

Public demos on free tiers get scraped, and every visitor consumes your free hours. A shared
password keeps it safe while still proving it works.

Create `.streamlit/secrets.toml` **locally** (never committed):

```toml
APP_PASSWORD = "pick-something-strong"
```

On **Streamlit Community Cloud**: Manage app → **Settings** → **Secrets** → paste the same line.
On **Hugging Face**: Settings → *Repository secrets* → name `APP_PASSWORD`.

Put the password in your CV/portfolio text, e.g. *"live demo (password: `demo2026`)"*.

> If you leave `APP_PASSWORD` unset, the app is fully public. That's fine for a portfolio.

---

## Step 6 — Claim your free extras

**A screenshot for the CV** (since free apps sleep):

```bash
# with the app running locally
pip install playwright && python -m playwright install chromium
python - <<'PY'
from playwright.sync_api import sync_playwright
import time
with sync_playwright() as p:
    b = p.chromium.launch()
    pg = b.new_page(viewport={"width":1440,"height":900}, device_scale_factor=2)
    pg.goto("http://localhost:8501", wait_until="load")
    time.sleep(10)
    pg.screenshot(path="dashboard_screenshot.png")
    b.close()
PY
```

**A badge for your README:**

```markdown
[![Live Demo](https://img.shields.io/badge/🚀_Live_Demo-streamlit.app-FF4B4B?style=for-the-badge)](https://YOUR-APP.streamlit.app)
[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.6-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)](https://scikit-learn.org)
```

---

# Part B — Putting it on your CV

## The rule that matters most

**Never claim more than the model does.** A recruiter who clicks your link and reads
"simulated dataset · 3.4% failure rate" will respect you far more than one who catches you
claiming "predicts equipment failures with 99% accuracy" when the honest number is F1 0.86.

Your project has two genuinely strong, verifiable claims:

- **F1 0.855 / PR-AUC 0.883** on a held-out set, with the *stricter* time-ordered result
  (**F1 0.727**) also reported. Showing both is a mark of maturity.
- You **found and removed a data-leakage trap** (the 5 failure-mode columns) and
  **documented label noise**. That is exactly the judgement senior engineers look for.

---

## CV entry (copy-paste ready)

### Condensed — one or two lines, for a dense CV

> **Industrial Equipment Failure Prediction** — ML decision-support app · [live demo](URL) · [code](URL)
> Built an end-to-end predictive-maintenance pipeline (cleaning → feature engineering → model
> selection → evaluation → Streamlit dashboard) on 10,000 machine records. Compared Logistic
> Regression, Decision Tree and Random Forest with class-weighted training and threshold tuning;
> best model reached **F1 0.86 / PR-AUC 0.88** (F1 0.73 under a stricter time-ordered split).
> Detected and removed failure-mode columns that leaked the target.

### Standard — three bullets, for a normal CV

> **Industrial Equipment Failure Prediction** | Python, scikit-learn, Streamlit
> [Live demo](URL) · [Source](URL)
>
> - Built an end-to-end ML pipeline predicting industrial equipment failure from sensor data
>   (temperature, rotational speed, torque, tool wear) across 10,000 machine records; shipped as
>   an interactive Streamlit dashboard with single-machine scoring, batch CSV scoring and live
>   threshold control.
> - Engineered physics-motivated features (mechanical power, temperature differential,
>   wear–torque interaction) and added engineering review edits; compared Logistic Regression,
>   Decision Tree and Random Forest using class weighting, 5-fold cross-validation and
>   bootstrap confidence intervals. Best model: **F1 0.855, PR-AUC 0.883**, with a stricter
>   time-ordered split (F1 0.727) reported alongside.
> - Identified that five failure-mode columns were **leaking the target** and excluded them;
>   documented label noise and unlearnable failure modes rather than inflating results, and
>   explained predictions with permutation importance, partial dependence and LR coefficients.

### Detailed — for a portfolio page or a project-focused CV

> **Industrial Equipment Failure Prediction**
> *Solo project · Python, pandas, scikit-learn, Streamlit · [Live demo](URL) · [GitHub](URL)*
>
> **Problem.** Unplanned equipment failure is expensive; maintenance teams need to know which
> machines deserve inspection first. Built a decision-support tool that estimates failure
> probability from five sensor channels and explains why.
>
> **What I built.** A reproducible pipeline (`src/`) covering ingestion, a reusable cleaning
> module validated against a deliberately corrupted dataset, physics-motivated feature
> engineering, model comparison with imbalance handling, and a four-view Streamlit dashboard
> using the *same* feature code as training to eliminate train/serve skew.
>
> **Results.** Random Forest achieved **F1 0.855 / PR-AUC 0.883 / precision 0.946** on a held-out
> test set at a tuned threshold; **F1 0.727** under a stricter time-ordered split. Logistic
> Regression plateaued at F1 0.475, which identified that the failure boundary is multiplicative
> in speed and torque.
>
> **Engineering judgement.** Found that five failure-mode columns reconstructed the target
> (data leakage) and excluded them; discovered 27 internally inconsistent labels and preserved
> the published target instead of silently rewriting it; quantified uncertainty with bootstrap
> CIs and stated all limitations up front.
>
> **Honest framing.** Trained on the *simulated* AI4I 2020 dataset; outputs are probabilities for
> prioritising inspections, not guarantees of failure.

---

## Make the CV entry credible in 4 clicks

A recruiter will check the link. Make those 60 seconds count:

1. **Pin the repo** on your GitHub profile (Profile → Customize your pins).
2. **Put the screenshot at the top of the README** — most people never scroll.
3. **Add the live-demo badge** to the README (see Step 6).
4. **Name the dataset and its limitation in the README's first 5 lines.** "Simulated data" is a
   credibility signal, not a weakness — it shows you know what your numbers mean.

---

## Talking about it in an interview

Rehearse these five, because they're what you'll actually be asked.

**"Why only F1 0.86? Couldn't you get higher?"**
> "That's the honest ceiling here. 3.4% of records fail, so 68 of 2,000 test rows are failures,
> and the 95% confidence interval on F1 is about ±0.07. Also 19 records are labelled 'random
> failure' with no sensor signature — no model can catch them. Chasing higher scores would mean
> fitting noise."

**"How did you handle the class imbalance?"**
> "`class_weight='balanced'` instead of SMOTE — I didn't want to invent synthetic sensor readings
> that never occurred physically. I also tuned the decision threshold on out-of-fold training
> predictions only, so the test set stayed untouched."

**"What would you do with more time?"**
> "Per-machine time-series data. Every row here is an independent run, so real degradation trends —
> the strongest signal in a real plant — can't be modelled. That's the biggest gap to something
> deployable, and I'd add drift monitoring and a cost-sensitive threshold using actual inspection
> and downtime costs."

**"What was the hardest part?"**
> "Spotting the leakage. The dataset has five failure-mode columns that look like useful features
> but only exist *because* a failure happened — they'd have given a meaningless 100% model.
> While removing them I found the flags don't perfectly reconstruct the target: 18 rows flag a
> mode but are labelled healthy, and 9 failures have no flag. My first version silently relabelled
> those rows, which would have rewritten the ground truth. I reverted it and documented the
> inconsistency as label noise instead."

**"Is this production-ready?"**
> "No, and I say so in the report. It's trained on simulated data and has no sensor-health checks
> upstream — a broken sensor could masquerade as a failing machine. It's decision support for
> prioritising inspections, not a safety system."

---

## Where to link it

| Place | What to put |
|---|---|
| CV header | `Portfolio: yourname.dev` — a one-page site beats three links |
| CV project entry | Live demo + GitHub, both clickable |
| GitHub profile README | Badge + screenshot + one-line result |
| LinkedIn **Featured** section | Link to the live demo with the screenshot as the thumbnail |
| LinkedIn project section | The three-bullet version |
| Cover letters | One sentence: *"…an ML app I deployed, which predicts equipment failure from sensor data (F1 0.86) — [link]"* |

**Never** paste a `localhost` link, and always check your public URL in a private/incognito
window before sending it to anyone.

---

## Deployment checklist

- [ ] Pushed to a **public** GitHub repo
- [ ] `artifacts/` committed (models + figures present in the repo)
- [ ] Deployed with `requirements-app.txt`, main file `app/streamlit_app.py`, Python 3.11
- [ ] Public URL opens in an **incognito** window, all four views work
- [ ] Opened it on a **phone** (recruiters do this) — the layout should still be readable
- [ ] Password set via Secrets (if you want to limit access) and noted on the CV
- [ ] Screenshot taken and embedded near the top of the README
- [ ] Live-demo badge in the README
- [ ] Repo pinned on your GitHub profile
- [ ] `README.md` states the dataset and its limitations in the first few lines
- [ ] No `localhost` links anywhere
- [ ] No secrets committed (`git status` is clean of `secrets.toml`)

---

## Final word on framing

Your strongest asset isn't the F1 score — plenty of candidates have one. It's that you
**removed a feature that made your model look better**, **reported the split that made it look
worse**, and **put the limitations on the first screen of the app**.

That is the difference between someone who can run scikit-learn and someone who can be trusted
with a model that affects real equipment. Say that out loud in interviews.
