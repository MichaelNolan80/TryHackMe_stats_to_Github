
My project to get updated stats from TryHackMe to show on my Githun. 
# Automated TryHackMe Profile Screenshot for GitHub README

## Project Summary

This project automates the process of taking a screenshot of a TryHackMe profile, cropping the useful profile/statistics section, saving it as a PNG file, and publishing it to a GitHub repository so it can be displayed inside a GitHub README.

The final working setup runs from a Linux/Kali machine using a cron job. The cron job runs a Python script, which opens the TryHackMe profile page, captures a screenshot, crops it, saves the cropped image, copies it into a local GitHub repository, commits the change, and pushes it to GitHub automatically.

> **Note:** All personal details, usernames, repository names, SSH keys, profile URLs, and local account details have been replaced with placeholders.

---

## Original Plan

The original plan was to:

1. Take a screenshot of a TryHackMe profile page.
2. Crop the screenshot so only the important profile/statistics section was visible.
3. Save the cropped image as a PNG file.
4. Add the PNG image to a GitHub repository.
5. Display the image in a GitHub README.
6. Automate the whole process so the image updates without manual input.

Expected README output:

```markdown
![TryHackMe Profile](images/tryhackme-profile-cropped.png)
```
![TryHackMe Profile](https://github.com/MichaelNolan80/MichaelNolan80/raw/main/images/tryhackme-profile-cropped.png)

---
## Expected Outcome

The intended final workflow was:

```text
Automated scheduled task
→ Open TryHackMe profile page
→ Take screenshot
→ Crop profile/statistics area
→ Save PNG locally
→ Copy image into GitHub repository
→ Commit updated image
→ Push to GitHub
→ README displays latest profile image
```

---

## What Changed During the Project

The original idea was to use **GitHub Actions** to run the screenshot process directly inside GitHub.

That approach did not work because the TryHackMe website, protected by browser verification/security checks, blocked the GitHub Actions cloud runner. Instead of receiving the real profile page, the workflow received a browser verification failure.

The working approach changed to:

```text
Run the screenshot script locally on Kali
→ Use cron for automation
→ Push the finished image to GitHub
```

This worked because the Kali machine could access and screenshot the TryHackMe profile page successfully.

---

## Final Working Design

Final automation flow:

```text
Kali Linux cron job
→ Runs Python script
→ Opens TryHackMe profile page using Playwright
→ Takes full screenshot
→ Crops image using Pillow
→ Saves cropped PNG locally
→ Copies PNG into local GitHub repo
→ Runs git add
→ Runs git commit
→ Runs git pull --rebase
→ Runs git push
→ GitHub README updates with latest image
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux | Local machine running the automation |
| Python 3 | Main scripting language |
| Python virtual environment | Isolated Python package environment |
| Playwright | Opens the web page and takes the screenshot |
| Chromium | Browser used by Playwright |
| Pillow | Crops the screenshot image |
| cron | Runs the script automatically on a schedule |
| Git | Commits and pushes the image to GitHub |
| SSH key | Allows GitHub push without entering a password |
| GitHub README | Displays the final image |

---

## Final File Structure

Project script folder:

```text
/home/<linux-user>/scripts/tryhackme-screenshot/
├── venv/
├── screenshot_profile.py
├── test_crop.py
├── tryhackme-profile-full.png
├── tryhackme-profile-cropped.png
└── cron.log
```

Local GitHub repository:

```text
/home/<linux-user>/<github-repo>/
├── README.md
├── images/
│   └── tryhackme-profile-cropped.png
└── .git/
```

README image reference:

```markdown
![TryHackMe Profile](images/tryhackme-profile-cropped.png)
```

---

## Redacted Placeholders

| Placeholder | Meaning |
|---|---|
| `<linux-user>` | Local Linux username |
| `<tryhackme-profile-url>` | Public TryHackMe profile URL |
| `<github-username>` | GitHub username |
| `<github-repo>` | GitHub repository name |
| `<github-repo-path>` | Local path to cloned GitHub repository |
| `<ssh-public-key>` | Public SSH key added to GitHub |
| `<ssh-private-key>` | Private SSH key, never shared |
| `<cron-time>` | Scheduled cron time |

No real usernames, passwords, tokens, or SSH keys are included in this document.

---

# Step-by-Step Build Guide

## 1. Create the Project Folder

Create a folder to hold the automation files:

```bash
mkdir -p /home/<linux-user>/scripts/tryhackme-screenshot
cd /home/<linux-user>/scripts/tryhackme-screenshot
```

---

## 2. Create a Python Virtual Environment

Kali uses an externally managed Python environment, so Python packages should be installed inside a virtual environment instead of globally.

Install the required system packages:

```bash
sudo apt update
sudo apt install -y python3-venv python3-full
```

Create and activate the virtual environment:

```bash
cd /home/<linux-user>/scripts/tryhackme-screenshot
python3 -m venv venv
source venv/bin/activate
```

After activation, the prompt should show `(venv)`.

---

## 3. Install Playwright and Pillow

Install the Python packages inside the virtual environment:

```bash
pip install playwright pillow
```

Install Chromium for Playwright:

```bash
python -m playwright install chromium
```

If Kali requires additional browser dependencies, install them with:

```bash
sudo apt install -y libnss3 libnspr4 libatk1.0-0t64 libatk-bridge2.0-0t64 libcups2t64 libdrm2 libxkbcommon0 libxcomposite1 libxdamage1 libxfixes3 libxrandr2 libgbm1 libasound2t64
```

---

## 4. Create the Screenshot Script

Create the main script:

```bash
nano /home/<linux-user>/scripts/tryhackme-screenshot/screenshot_profile.py
```

Paste the following redacted version:

```python
from playwright.sync_api import sync_playwright
from pathlib import Path
from PIL import Image
import subprocess
import time

PROFILE_URL = "<tryhackme-profile-url>"

BASE_DIR = Path.home() / "scripts" / "tryhackme-screenshot"

FULL_SCREENSHOT = BASE_DIR / "tryhackme-profile-full.png"
CROPPED_SCREENSHOT = BASE_DIR / "tryhackme-profile-cropped.png"

GITHUB_REPO = Path("/home/<linux-user>/<github-repo>")
GITHUB_IMAGE = GITHUB_REPO / "images" / "tryhackme-profile-cropped.png"

BASE_DIR.mkdir(parents=True, exist_ok=True)
GITHUB_IMAGE.parent.mkdir(parents=True, exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(
        headless=True,
        args=["--disable-dev-shm-usage"]
    )

    page = browser.new_page(
        viewport={"width": 1400, "height": 1000}
    )

    page.goto(
        PROFILE_URL,
        wait_until="domcontentloaded",
        timeout=60000
    )

    time.sleep(8)

    page.screenshot(
        path=str(FULL_SCREENSHOT),
        full_page=True
    )

    browser.close()

img = Image.open(FULL_SCREENSHOT)

left = 0
top = 210
right = 1380
bottom = 470

cropped = img.crop((left, top, right, bottom))
cropped.save(CROPPED_SCREENSHOT)

GITHUB_IMAGE.parent.mkdir(parents=True, exist_ok=True)
cropped.save(GITHUB_IMAGE)

subprocess.run(
    ["git", "add", "images/tryhackme-profile-cropped.png"],
    cwd=GITHUB_REPO,
    check=True
)

commit_result = subprocess.run(
    ["git", "commit", "-m", "Update TryHackMe profile image"],
    cwd=GITHUB_REPO,
    text=True,
    capture_output=True
)

if commit_result.returncode == 0:
    subprocess.run(
        ["git", "pull", "--rebase", "origin", "main"],
        cwd=GITHUB_REPO,
        check=True
    )

    subprocess.run(
        ["git", "push"],
        cwd=GITHUB_REPO,
        check=True
    )

    print("Updated image pushed to GitHub.")
else:
    print("No image changes to commit.")

print(f"Saved full screenshot to: {FULL_SCREENSHOT}")
print(f"Saved cropped screenshot to: {CROPPED_SCREENSHOT}")
print(f"Saved GitHub image to: {GITHUB_IMAGE}")
```

Save and exit:

```text
Ctrl + O
Enter
Ctrl + X
```

---

## 5. Test the Screenshot Script

Run the script manually:

```bash
/home/<linux-user>/scripts/tryhackme-screenshot/venv/bin/python /home/<linux-user>/scripts/tryhackme-screenshot/screenshot_profile.py
```

Check that the image was created:

```bash
ls -lh /home/<linux-user>/scripts/tryhackme-screenshot/tryhackme-profile-full.png
```

Open the image:

```bash
xdg-open /home/<linux-user>/scripts/tryhackme-screenshot/tryhackme-profile-full.png
```

---

# Offline Crop Testing

## Why This Was Needed

The screenshot worked, but the useful part of the image needed to be cropped. Instead of loading the TryHackMe page every time, a separate offline crop test script was created.

This allowed the crop values to be adjusted quickly using an existing screenshot.

---

## 6. Create the Offline Crop Test Script

Create the file:

```bash
nano /home/<linux-user>/scripts/tryhackme-screenshot/test_crop.py
```

Paste:

```python
from PIL import Image, ImageDraw
from pathlib import Path

BASE_DIR = Path.home() / "scripts" / "tryhackme-screenshot"

INPUT_FILE = BASE_DIR / "tryhackme-profile-full.png"

CROP_PREVIEW_FILE = BASE_DIR / "crop-preview-box.png"
CROPPED_FILE = BASE_DIR / "tryhackme-profile-cropped.png"

left = 0
top = 210
right = 1380
bottom = 470

img = Image.open(INPUT_FILE)

preview = img.copy()
draw = ImageDraw.Draw(preview)
draw.rectangle((left, top, right, bottom), outline="red", width=6)
preview.save(CROP_PREVIEW_FILE)

cropped = img.crop((left, top, right, bottom))
cropped.save(CROPPED_FILE)

print(f"Input image: {INPUT_FILE}")
print(f"Preview with crop box: {CROP_PREVIEW_FILE}")
print(f"Cropped image: {CROPPED_FILE}")
print()
print("Crop values:")
print(f"left={left}, top={top}, right={right}, bottom={bottom}")
```

Run it:

```bash
/home/<linux-user>/scripts/tryhackme-screenshot/venv/bin/python /home/<linux-user>/scripts/tryhackme-screenshot/test_crop.py
```

Open the preview image:

```bash
xdg-open /home/<linux-user>/scripts/tryhackme-screenshot/crop-preview-box.png
```

Open the cropped image:

```bash
xdg-open /home/<linux-user>/scripts/tryhackme-screenshot/tryhackme-profile-cropped.png
```

---

## Final Crop Values

The final crop values used were:

```python
left = 0
top = 210
right = 1380
bottom = 470
```

These values were copied into the main `screenshot_profile.py` script.

---

# GitHub Actions Attempt

## What Was Attempted

A GitHub Actions workflow was created so GitHub could run the screenshot process directly.

Expected workflow:

```text
GitHub Actions
→ Install Python
→ Install Playwright
→ Take TryHackMe screenshot
→ Commit PNG into repo
```

Example workflow file:

```yaml
name: Update TryHackMe Screenshot

on:
  schedule:
    - cron: "0 7 * * *"
  workflow_dispatch:

permissions:
  contents: write

jobs:
  screenshot:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install Python packages
        run: |
          pip install playwright pillow
          python -m playwright install --with-deps chromium

      - name: Take TryHackMe screenshot
        run: |
          python scripts/screenshot_profile.py

      - name: Commit updated screenshot
        run: |
          git config user.name "github-actions"
          git config user.email "github-actions@github.com"
          git add images/tryhackme-profile.png
          git diff --cached --quiet || git commit -m "Update TryHackMe screenshot"
          git push
```

## Why GitHub Actions Was Not Used

GitHub Actions was blocked by the website’s browser verification/security page. The GitHub cloud runner did not receive the real profile page.
<img width="1299" height="727" alt="image" src="https://github.com/user-attachments/assets/b4eb0e2e-05ea-453d-9095-d8022e20a666" />


The final decision was:

```text
Do not use GitHub Actions for the screenshot capture.
Use Kali cron instead.
```

---

# GitHub Repository Setup on Kali

## 7. Install Git

The first attempt to clone the repository failed because Git was not installed.

Install Git:

```bash
sudo apt update
sudo apt install -y git
```

Confirm installation:

```bash
git --version
```

---

## 8. Clone the GitHub Repository Locally

Clone the repository:

```bash
cd /home/<linux-user>
git clone https://github.com/<github-username>/<github-repo>.git
```

This creates:

```text
/home/<linux-user>/<github-repo>
```

---

## 9. Copy the Cropped Image into the Local Repo

```bash
mkdir -p /home/<linux-user>/<github-repo>/images
cp /home/<linux-user>/scripts/tryhackme-screenshot/tryhackme-profile-cropped.png /home/<linux-user>/<github-repo>/images/tryhackme-profile-cropped.png
```

---

# SSH Authentication Setup

## Why SSH Was Needed

The first `git push` used HTTPS and asked for a username and password. This was not suitable for cron automation.

SSH was used because it allows the Kali machine to push to GitHub without asking for credentials interactively.

---

## 10. Create an SSH Key

```bash
ssh-keygen -t ed25519 -C "<github-username> GitHub"
```

For automation, the passphrase was left empty.

> **Important:** Never share the private key file.

Private key:

```text
~/.ssh/id_ed25519
```

Public key:

```text
~/.ssh/id_ed25519.pub
```

---

## 11. Display the Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

Example format:

```text
ssh-ed25519 <ssh-public-key> <github-username> GitHub
```

Only the public key should be added to GitHub.

---

## 12. Add the Public SSH Key to GitHub

On the GitHub website:

```text
Profile picture
→ Settings
→ SSH and GPG keys
→ New SSH key
```

Use:

```text
Title: Kali Purple
Key type: Authentication Key
Key: <ssh-public-key>
```

---

## 13. Change the Git Remote from HTTPS to SSH

Inside the local repository:

```bash
cd /home/<linux-user>/<github-repo>
git remote set-url origin git@github.com:<github-username>/<github-repo>.git
```

Check the remote:

```bash
git remote -v
```

Expected output:

```text
origin  git@github.com:<github-username>/<github-repo>.git (fetch)
origin  git@github.com:<github-username>/<github-repo>.git (push)
```

---

## 14. Test GitHub SSH Access

```bash
ssh -T git@github.com
```

Expected result:

```text
Hi <github-username>! You've successfully authenticated...
```

---

## 15. Commit and Push the Image

```bash
cd /home/<linux-user>/<github-repo>
git add images/tryhackme-profile-cropped.png
git commit -m "Add TryHackMe profile image"
git push
```

If Git reports that the local branch is ahead of the remote, run:

```bash
git push
```

If Git rejects the push because the remote contains changes, run:

```bash
git pull --rebase origin main
git push
```

---

# Cron Job Setup

## 16. Add the Cron Job

Open crontab:

```bash
crontab -e
```

Add:

```bash
0 7 * * * /home/<linux-user>/scripts/tryhackme-screenshot/venv/bin/python /home/<linux-user>/scripts/tryhackme-screenshot/screenshot_profile.py
```

This runs the script every day at 07:00.

---

## 17. Save the Cron Job

If the editor is Vim, save and exit with:

```text
Esc
:wq
Enter
```

Confirm the cron job:

```bash
crontab -l
```

Expected output:

```bash
0 7 * * * /home/<linux-user>/scripts/tryhackme-screenshot/venv/bin/python /home/<linux-user>/scripts/tryhackme-screenshot/screenshot_profile.py
```

---

## 18. Optional: Add Logging to Cron

Recommended cron line with logging:

```bash
0 7 * * * /home/<linux-user>/scripts/tryhackme-screenshot/venv/bin/python /home/<linux-user>/scripts/tryhackme-screenshot/screenshot_profile.py >> /home/<linux-user>/scripts/tryhackme-screenshot/cron.log 2>&1
```

Check the log:

```bash
cat /home/<linux-user>/scripts/tryhackme-screenshot/cron.log
```

---

# Final README Entry

Add this to `README.md`:

```markdown
![TryHackMe Profile](images/tryhackme-profile-cropped.png)
```

---

# Issues Encountered and Fixes

| Issue | Cause | Fix |
|---|---|---|
| `externally-managed-environment` | Kali blocks global pip installs | Used Python virtual environment |
| Playwright timeout | `networkidle` waited too long | Changed to `domcontentloaded` |
| Missing dependency | `libasound2` not available | Used `libasound2t64` |
| Script duplicated | File pasted incorrectly | Overwrote file cleanly |
| Heredoc markers in Python file | Shell command pasted into nano | Removed `<< 'EOF'` and `EOF` |
| Syntax error | `import time` joined to another line | Put import on its own line |
| Git not found | Git not installed | Installed Git with apt |
| GitHub asked for password | Remote used HTTPS | Switched to SSH |
| Git push rejected | Remote had changes not local | Added `git pull --rebase origin main` before push |
| GitHub Actions blocked | Website browser verification | Switched to local Kali cron job |

---


# Final Result

The project was successful.

The final system:

```text
Runs automatically from Kali
Takes a TryHackMe profile screenshot
Crops the useful profile section
Saves it as a PNG
Copies it into a GitHub repository
Commits the update
Pushes it to GitHub
Displays it in the README
```

The automation works without manual input, provided that:

```text
Kali is powered on
Internet access is available
TryHackMe profile page loads correctly
GitHub SSH authentication still works
Cron is active
```

---

# Final Verification Commands

Check cron:

```bash
crontab -l
```

Run the script manually:

```bash
/home/<linux-user>/scripts/tryhackme-screenshot/venv/bin/python /home/<linux-user>/scripts/tryhackme-screenshot/screenshot_profile.py
```

Check output image:

```bash
ls -lh /home/<linux-user>/scripts/tryhackme-screenshot/tryhackme-profile-cropped.png
```

Check GitHub repo status:

```bash
cd /home/<linux-user>/<github-repo>
git status
```

Check GitHub remote:

```bash
git remote -v
```

Expected remote format:

```text
git@github.com:<github-username>/<github-repo>.git
```

Check cron log:

```bash
cat /home/<linux-user>/scripts/tryhackme-screenshot/cron.log
```

---

# Suggested Future Improvements

Possible future improvements:

1. Add better logging with timestamps.
2. Add error handling if TryHackMe fails to load.
3. Add a backup image if the screenshot fails.
4. Add a notification when cron fails.
5. Store the crop values in a config file.
6. Add a second image for another profile/stat section.
7. Add automatic image compression before pushing to GitHub.

---

## Project Status

```text
Status: Complete
Automation: Working
GitHub README image: Working
Manual input required: No, after setup
```
