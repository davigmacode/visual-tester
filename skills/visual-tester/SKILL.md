---
name: visual-tester
description: Automated headless browser testing, visual verification, and UI snapshot inspection for Google Antigravity. Drives Chromium via Playwright or lightweight Node scripts, verifies live DOM elements, captures viewport/element screenshots, inspects Three.js/WebGL canvas rendering, and checks for visual regressions. Triggers on "/visual-test", "visual test", "verify in browser", "take a screenshot of localhost", "test ui rendering", "playwright snapshot".
---

# Visual Tester for Google Antigravity

Inspect live web applications, take viewport screenshots, verify Three.js/WebGL canvases, and eliminate visual regressions through automated browser testing.

---

## Core Mission

> **NEVER ASSUME UI OR 3D WORKS WITHOUT VISUAL PROOF.**
> A web app may build with zero compiler errors, but have overlapping text, broken flexbox alignments, blank Three.js canvas contexts, or unhandled client-side JavaScript console errors. Visual Tester gives the agent "eyes" to inspect actual rendered state.

---

## 4 Core Capabilities

### 1. Viewport & Multi-Device Snapshots
* Captures high-fidelity PNG screenshots at standard viewports:
  * **Desktop**: 1440x900 or 1920x1080
  * **Tablet**: 768x1024
  * **Mobile**: 375x812 (iPhone standard)
* Enables inspection of responsive layout breaks, hamburger menu states, and font scaling.

### 2. Three.js / WebGL & Canvas Verification
* Specifically checks if a `<canvas>` element exists and is active.
* Probes for WebGL context creation failures (`webglcontextlost`, invalid shader compilations).
* Captures animation frames at specified time offsets (e.g. `t=0s`, `t=2s`, `t=5s`) to ensure procedural motion or GSAP animations are actually progressing.

### 3. Console & Network Error Trapping
* Intercepts unhandled browser exceptions (`window.onerror`, unhandled promise rejections).
* Detects failed HTTP 404/500 asset requests (missing textures, failed font loads, broken API endpoints).

### 4. Interactive Flow Verification (E2E)
* Simulates real user interactions: clicking buttons, filling inputs, waiting for asynchronous state transitions, and asserting element visibility.

---

## Standard Execution Pattern (Zero Extra Config)

When asked to visually test or inspect a local URL (e.g. `http://localhost:3000` or a static `index.html` file):

### Step 1: Check Environment & Serve
If testing a static file or build directory, launch a lightweight background server using Python:
```powershell
python -m http.server 8080 --directory ./dist
```

### Step 2: Run Headless Snapshot Script
Use a lightweight runner with `npx playwright` or a standalone snapshot script in the Antigravity `scratch/` directory:

```javascript
// scratch/snapshot.mjs
import { chromium } from 'playwright';

const browser = await chromium.launch({ headless: true });
const context = await browser.newContext({ viewport: { width: 1440, height: 900 } });
const page = await context.newPage();

const consoleErrors = [];
page.on('console', msg => {
  if (msg.type() === 'error') consoleErrors.push(msg.text());
});

await page.goto(process.argv[2] || 'http://localhost:8080', { waitUntil: 'networkidle' });
await page.screenshot({ path: process.argv[3] || 'screenshot.png', fullPage: false });

console.log(JSON.stringify({ consoleErrors, title: await page.title() }));
await browser.close();
```

Run via Antigravity `run_command`:
```powershell
node scratch/snapshot.mjs http://localhost:8080 scratch/view-desktop.png
```

### Step 3: Embed Screenshots in Antigravity Artifacts
Copy the captured screenshot into the conversation artifact directory (`<appDataDir>/brain/<conversation-id>/`) and embed it into a review artifact:
```markdown
# Visual Verification Report

![Desktop Viewport](view-desktop.png)

### Diagnostics
- Page Title: ...
- Console Errors: 0
- WebGL Status: Operational
```

---

## Synergy with Gauntlet Loop & Three.js

* **Gauntlet Loop Integration**: When running the **Classic A/B** mode of `gauntlet-loop`, `visual-tester` acts as the fetcher that screenshots the real benchmark site and our local implementation at identical viewports for the `Harsh Critic` subagent.
* **Three.js Validation**: Validates that models loaded via `threejs-loaders` or custom shaders from `threejs-shaders` render proper pixels rather than a blank canvas.

---

## Trigger Commands

* `/visual-test [URL or path]`
* `verify in browser: [URL]`
* `take a screenshot of localhost: [port]`
* `test ui rendering for [feature]`
