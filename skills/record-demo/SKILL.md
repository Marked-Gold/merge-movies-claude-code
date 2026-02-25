# Record Demo Skill

Record a browser demo video using Playwright and add it as a VideoView scene in a merge.mov movie.

$ARGUMENTS

## Overview

This skill records a browser viewport video by:
1. Ensuring Playwright + Chromium are available
2. Writing a Playwright script to `.merge-movies/demos/`
3. Running the script (video output saved alongside it)
4. Uploading the resulting WebM via a presigned URL
5. Creating a VideoView scene in the movie

Scripts and videos are persisted in `.merge-movies/demos/` so they can be reviewed, re-run, or debugged.

## MCP Tools

This skill uses the `merge-movies` MCP server. All tools are available automatically. Authentication is handled by the MCP transport via the `MERGE_MOVIES_API_KEY` environment variable.

If the key is missing, tell the user to create one at https://merge.mov/settings.

**Key tools for this skill:**

| Tool | Description |
|------|-------------|
| `create_video_upload` | Get a presigned upload URL for a video file |
| `create_scene` | Create a VideoView scene with the uploaded video |
| `list_movies` | List movies to find the target movie |
| `get_movie` | Get movie details |

## Workflow

### Step 1: Determine Target Movie and Demo Scenario

Parse `$ARGUMENTS` to determine:
- **Movie ID** — Which movie to add the demo to. If not specified, list movies and ask.
- **Demo scenario** — What to record. This can be a URL + description, or the agent infers from context (e.g., the feature just implemented).

### Step 2: Ensure Playwright is Available

```bash
npm install playwright 2>/dev/null && npx playwright install chromium --with-deps 2>/dev/null || npx playwright install chromium
```

This installs the `playwright` npm package and downloads Chromium (~150MB) on first run. If already installed, it exits quickly.

### Step 3: Write the Playwright Script

Choose a descriptive kebab-case name for the demo (e.g., `login-flow`, `search-feature`, `theme-picker`). Use the **Write** tool to create the script at `.merge-movies/demos/<name>.mjs`.

The script should follow this template:

```javascript
// .merge-movies/demos/<name>.mjs
import { chromium } from 'playwright';
import { dirname } from 'path';
import { fileURLToPath } from 'url';

const __dirname = dirname(fileURLToPath(import.meta.url));

const browser = await chromium.launch();
const context = await browser.newContext({
  recordVideo: { dir: __dirname, size: { width: 1280, height: 720 } },
  viewport: { width: 1280, height: 720 },
});
const page = await context.newPage();

// === Customize this section for the specific demo ===
await page.goto('http://localhost:5173');
// Perform the demo actions: clicks, fills, navigations, waits...
// Example:
// await page.fill('#email', 'user@example.com');
// await page.click('button[type="submit"]');
// await page.waitForSelector('.dashboard');
// === End custom section ===

// Final pause so the viewer can see the end state
await page.waitForTimeout(2000);

// Close context to finalize the video
await context.close();
const videoPath = await page.video().path();
console.log('VIDEO_PATH:' + videoPath);
await browser.close();
```

**Tips for the script:**
- Use `page.waitForTimeout(ms)` between actions for pacing (500-1500ms between steps)
- Use `page.waitForSelector(selector)` instead of fixed waits when possible
- Add a 2s pause at the end so the final state is visible
- Keep demos short: 10-30 seconds is ideal
- Use headless mode (default) — it works everywhere and only captures the viewport
- **Avoid repeated logins** — If the app requires authentication, use Playwright's `storageState` to save the browser session after the first login and reuse it for subsequent recordings:
  ```javascript
  // After logging in:
  await context.storageState({ path: '.merge-movies/demos/auth-state.json' });
  // For future recordings:
  const context = await browser.newContext({ storageState: '.merge-movies/demos/auth-state.json', ... });
  ```

### Step 4: Run the Script

```bash
node .merge-movies/demos/<name>.mjs
```

Parse the video file path from the `VIDEO_PATH:` line in stdout. The `.webm` file will be saved in `.merge-movies/demos/` alongside the script.

If the script fails, read the error output, fix the script file, and re-run. The script persists on disk so you can iterate without rewriting from scratch.

### Step 5: Upload the Video

1. Call `create_video_upload` with the movie ID:

```
create_video_upload({ movieId: "<movie-id>" })
-> { presignedUploadUrl, videoSource, filename, uploadInstructions }
```

2. Upload the file using the presigned URL:

```bash
curl -X PUT "<presignedUploadUrl>" \
  -H "Content-Type: video/webm" \
  --data-binary @<video-path>
```

### Step 6: Create the VideoView Scene

```
create_scene({
  movieId: "<movie-id>",
  scene: {
    title: "Demo: <feature name>",
    narration: "<description of what the video shows>",
    view: {
      type: "video",
      source: "<videoSource from step 5>",
      aspectRatio: 1.778
    }
  }
})
```

**Note:** `aspectRatio` should be `1.778` (16:9) to match the 1280x720 recording.

### Step 7: Confirm

Tell the user the demo video has been recorded and added to the movie. Provide the studio URL so they can preview it.

Note: The script and video remain in `.merge-movies/demos/` for future reference. This folder is gitignored.

## Examples

### Record a login flow demo

```
/merge-movies:record-demo movie:abc123 Demo the login flow at localhost:5173
```

### Record a feature demo after implementing it

```
/merge-movies:record-demo movie:abc123 Demo the new search feature we just built
```

### Record with a specific URL

```
/merge-movies:record-demo movie:abc123 Navigate to localhost:5173/settings and show the new theme picker
```

## Troubleshooting

- **Playwright not installed**: Run `npm install playwright && npx playwright install chromium` manually
- **Video file empty**: Ensure `context.close()` is called before accessing the video path
- **Upload fails**: Check that the presigned URL hasn't expired (15 min window)
- **Page not loading**: Verify the dev server is running and accessible at the specified URL
- **Script errors**: The script is at `.merge-movies/demos/<name>.mjs` — read the error, fix the file, and re-run
