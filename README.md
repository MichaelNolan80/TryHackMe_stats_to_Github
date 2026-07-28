# TryHackMe Stats Automation

This project is being built to capture my TryHackMe profile progress and display it as an image that can be used on GitHub, LinkedIn, or a personal website.

The aim is to create a simple, repeatable way to show up-to-date TryHackMe progress without relying on the older TryHackMe profile badge system or needing a numerical public badge ID.

## Project Goal

The goal of this project is to:

- Open my public TryHackMe profile page
- Capture the visible stats/progress section
- Save it as an image file
- Display that image in my GitHub profile or project README
- Improve the automation over time through separate editions

<img width="1380" height="260" alt="image" src="https://github.com/user-attachments/assets/cc377e39-07eb-4ff7-8072-ad818a9c0c50" />



## Why This Project Exists

TryHackMe profile badges and public embed options appear to have changed over time, and older badge workflows may no longer work reliably.

Instead of depending on an old badge API, this project explores browser-based methods to capture the public profile page and generate a reusable image.

## Project Structure

```text
.
├── README.md
├── editions/
│   ├── basic-copy-paste.md
│   └── banner-selected-area.md
├── assets/
│   └── tryhackme-stats.png
├── scripts/
│   └── tryhackme-stats.js
├── package.json
└── .github/
    └── workflows/
        └── update-tryhackme-stats.yml
```

## Editions

Each edition documents a different version of the project. This keeps the main README clean while still recording what was tested, what worked, and what needs improving.

| Edition | Description | Status |
|---|---|---|
| [Edition 1 — Basic Copy and Paste](editions/edition-01-basic-copy-paste.md) | First working version using screenshot capture and fixed cropping | Complete / tested |
| [Edition 2 — Banner Detection and Selected Area Capture](editions/edition_2_Banner_Detection_and_Selected_Area_Capture.md) | Planned improvement to close banners and capture a selected stats section | Planned |

## Current Output

The generated stats image is saved here:

```text
assets/tryhackme-stats.png
```

It can be displayed in GitHub using:

```md
![TryHackMe Stats](assets/tryhackme-stats.png)
```

## TryHackMe Profile

```md
[View my TryHackMe profile](https://tryhackme.com/p/ManicMookey)
```

## Planned Improvements

Future editions may include:

- Better banner detection
- Screenshotting only the stats section
- Running the capture locally on a schedule
- Automatically pushing the updated image to GitHub
- Improved error checking
- Avoiding broken image commits
- Support for LinkedIn or portfolio website output
- Better handling if TryHackMe changes its page layout

## Important Notes

This project is designed to use the public TryHackMe profile page only.

It should not store TryHackMe usernames, passwords, cookies, session tokens, API keys, or other secrets inside the GitHub repository.

## Project Status

The project is currently being developed in editions. Edition 1 has been tested and proved the screenshot/crop workflow, but GitHub-hosted automation can be blocked by TryHackMe browser verification. Further editions will improve reliability and explore better capture methods.
