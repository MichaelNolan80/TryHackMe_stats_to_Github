
# Edition 2 — Banner Detection and Selected Area Capture

## Overview

Edition 2 is planned as the next improvement to the TryHackMe stats automation project.

The aim is to make the capture process more reliable by detecting banners, closing or hiding them, and then capturing a selected section of the page rather than relying only on a fixed crop.

## Goal

The goal of Edition 2 is to:

- Detect if a banner or pop-up appears
- Close or hide the banner before the screenshot is taken
- Locate the TryHackMe stats area
- Capture only the selected stats section
- Save the result as `assets/tryhackme-stats.png`
- Avoid committing broken or incorrect images

## Planned Workflow

```text
Open TryHackMe profile
↓
Wait for page to load
↓
Check for browser verification failure
↓
Detect banner or pop-up
↓
Close or hide banner
↓
Locate stats section
↓
Screenshot selected section
↓
Save image
```

## Why This Edition Is Needed

Sometimes the TryHackMe website may display a banner at the top of the page.

If a fixed crop is used, the banner can push the profile stats lower down the page. This means the crop may capture the wrong section.

Edition 2 aims to avoid this by capturing a selected area or page element instead of using only fixed coordinates.

## Planned Banner Handling

```js
async function closePossibleBanners(page) {
  const possibleCloseButtons = [
    'button:has-text("Accept")',
    'button:has-text("I agree")',
    'button:has-text("Got it")',
    'button:has-text("Close")',
    'button:has-text("Dismiss")',
    '[aria-label="Close"]',
    '[aria-label="Dismiss"]'
  ];

  for (const selector of possibleCloseButtons) {
    const button = page.locator(selector).first();

    try {
      if (await button.isVisible({ timeout: 1500 })) {
        await button.click();
        await page.waitForTimeout(1000);
        console.log(`Closed banner using selector: ${selector}`);
      }
    } catch {
      // Ignore if not found
    }
  }
}
```

## Planned Banner Hiding

If a banner cannot be closed, the script may hide common banner elements before taking the screenshot.

```js
async function hidePossibleBanners(page) {
  await page.evaluate(() => {
    const possibleBanners = [
      '[role="banner"]',
      '.banner',
      '.cookie-banner',
      '.notification',
      '.announcement',
      '.toast',
      '.alert'
    ];

    for (const selector of possibleBanners) {
      document.querySelectorAll(selector).forEach(el => {
        el.style.display = "none";
      });
    }
  });
}
```

## Planned Selected Area Screenshot

Instead of cropping blindly, Playwright can screenshot a specific page element.

```js
const statsSection = page.locator("PASTE_STATS_SELECTOR_HERE").first();

await statsSection.screenshot({
  path: "assets/tryhackme-stats.png"
});
```

The exact selector needs to be found by inspecting the TryHackMe profile page in the browser.

## How to Find the Stats Selector

1. Open the TryHackMe profile page in Chrome or Edge.
2. Right-click the stats area.
3. Click **Inspect**.
4. Find the HTML element that contains the full stats section.
5. Right-click the element.
6. Choose **Copy → Copy selector**.
7. Paste that selector into the Playwright script.

## Expected Improvement Over Edition 1

Edition 2 should be more reliable because it will not depend only on fixed crop coordinates.

It should handle:

- Banners appearing at the top of the page
- Stats moving slightly on the page
- Different screen heights
- Safer failures when the stats section cannot be found

## Risks

This version may still break if:

- TryHackMe changes the page structure
- The selector changes
- TryHackMe blocks automated browsers
- The stats section is not visible without login
- Browser verification appears before the profile loads

## Status

Planned.
