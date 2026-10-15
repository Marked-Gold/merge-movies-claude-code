# Create Movie Skill

Create code walkthrough movies using merge.mov — from git diffs, feature walkthroughs, architecture overviews, setup guides, or free-form narratives.

$ARGUMENTS

## Overview

This skill creates engaging code walkthrough videos by:
1. Determining the creation path from arguments
2. Gathering source material (diffs, source files, docs)
3. Planning logical scenes that tell a story
4. Writing meaningful narration for each scene
5. Using the merge.mov MCP tools to create the movie

## Usage

- `/merge-movies:create-movie HEAD~3..HEAD` - From a commit range
- `/merge-movies:create-movie uncommitted` - From uncommitted changes
- `/merge-movies:create-movie <branch>` - From branch changes vs main
- `/merge-movies:create-movie walkthrough <feature>` - Feature walkthrough
- `/merge-movies:create-movie architecture` - Architecture overview
- `/merge-movies:create-movie setup` - Setup / getting started guide
- `/merge-movies:create-movie` - Interactive mode (free-form or guided)

## MCP Tools

This skill uses the `merge-movies` MCP server. All tools are available automatically — use them directly by name. Authentication is handled automatically via OAuth — on first use, a browser window opens for you to log in, and tokens are managed by Claude Code.

**Available tools:**

| Tool | Description |
|------|-------------|
| `list_movies` | List all movies |
| `get_movie` | Get a movie by ID with all scenes |
| `create_movie` | Create a new movie |
| `update_movie` | Update an existing movie (full replacement) |
| `delete_movie` | Delete a movie |
| `list_scenes` | List all scenes in a movie |
| `get_scene` | Get a single scene |
| `create_scene` | Create a new scene in a movie |
| `update_scene` | Full replacement of a scene |
| `patch_scene` | Partial update of a scene |
| `delete_scene` | Delete a scene |
| `reorder_scenes` | Reorder scenes within a movie |
| `list_codeblocks` | List code blocks in a scene |
| `get_codeblock` | Get a single code block |
| `create_codeblock` | Create a code block in a scene |
| `update_codeblock` | Update a code block |
| `delete_codeblock` | Delete a code block |
| `list_reference_assets` | List the movie's cast (characters/objects/locations/styles) for consistent AI video |
| `create_reference_asset` | Add a cast member, optionally AI-generating its reference image |
| `list_video_models` | AI video models the user can use, with sizes, durations and **USD per second** |
| `estimate_video_cost` | Price planned AI clips — nothing is generated or billed |
| `generate_scene_video` | **Spends money.** Generate an AI clip for a video scene, capped at `maxCostUsd` |
| `get_scene_video_status` | Poll a clip until it lands in the scene (`waitSeconds` blocks up to 50s) |

## Workflow

### Step 1: Determine Creation Path

Parse `$ARGUMENTS` to detect the creation mode:

| Argument Pattern | Mode | Description |
|-----------------|------|-------------|
| `HEAD~N..HEAD`, `abc123..def456` | Git diff (commit range) | Diff between two commits |
| `uncommitted` | Git diff (working tree) | Uncommitted changes vs HEAD |
| `<branch-name>` (matches a git branch) | Git diff (branch) | Branch changes vs main |
| `walkthrough <feature>` | Feature walkthrough | Explain how a feature works |
| `architecture` | Architecture overview | Explore and explain the system |
| `setup` | Setup guide | Document how to set up the project |
| (no args or free text) | Free-form / interactive | Ask user what story to tell |

If the mode is ambiguous, ask the user to clarify.

### Step 2: Gather Source Material

#### Git Diff Modes

```bash
# For commit range
git diff --name-status HEAD~3..HEAD   # scope first
git diff HEAD~3..HEAD                 # full diff

# For uncommitted changes
git diff --name-status HEAD
git diff HEAD

# For branch comparison
git diff --name-status main..feature-branch
git diff main..feature-branch
```

#### Feature Walkthrough Mode

1. Ask the user which feature to walk through (or infer from args)
2. Use Glob/Grep to find relevant source files — entry points, key modules, tests
3. Read each file to understand the implementation
4. Map out the data/control flow through the feature
5. Plan scenes that walk the viewer through how it works, not what changed

#### Architecture Overview Mode

1. Explore the directory structure (`ls`, Glob for key patterns)
2. Read entry points: `package.json`, `main.ts`, `app.ts`, config files
3. Identify layers: API routes, services, data models, utilities
4. Read representative files from each layer
5. Plan scenes that build up from foundations to the full picture

#### Setup Guide Mode

1. Read README, CONTRIBUTING, package.json, Dockerfile, docker-compose, config files
2. Identify prerequisites, install steps, environment variables, build commands
3. Test commands if possible to capture realistic terminal output
4. Plan scenes that walk through setup step-by-step

#### Free-form / Interactive Mode

1. Ask the user what story they want to tell
2. Gather supporting code, files, or context based on their description
3. Plan scenes collaboratively

### Step 3: Analyze and Plan Scenes

Create a scene outline before building. Group material into logical scenes:

**For git diff modes:**
- Group by feature, file type, or architectural layer
- Consider the narrative: What problem? What solution? What result?

**For walkthrough modes:**
- Follow the execution flow: entry point -> core logic -> output
- Build understanding progressively — introduce concepts before using them

**For architecture modes:**
- Start high-level (system diagram), drill into layers
- Show how components connect to each other

**For setup modes:**
- Follow the chronological setup order
- Show terminal commands alongside config files

**Scene Duration Guidelines:**
- Simple changes: 3-5 seconds per scene
- Complex logic: 8-12 seconds per scene
- Architecture overviews: 10-15 seconds
- AI video clips: 4-8 seconds each — they are billed per second

**Does the movie need AI video?** Marketing, launch and promo videos often want cinematic AI footage (a hero shot of someone using the product, an atmospheric opener, b-roll behind a tagline). Code walkthroughs, setup guides and architecture overviews almost never do — prefer code, terminal, React and screen-recording scenes, which are free. If AI video would help, follow Step 3b before writing anything.

### Step 3b: AI Video Budget Check (only if the plan has AI clips)

AI video is billed to the user's own provider account (fal.ai, Runway, …). **Never generate a clip the user hasn't approved the cost of.** Do this once, for the whole movie, before creating it:

1. **Pick models** with `list_video_models`. Only `configured: true` providers can generate; the user's default is `isDefault: true`. Rules of thumb:
   - Drafts / cheap b-roll: `gemini-omni-flash-1.1` at `360p`/`720p`, or `wan-3.0` at `480p`/`720p`
   - Best quality per dollar: `wan-3.0` at `1080p`
   - Hero shots: `kling-3.0-pro` or `seedance-2.5` (most expensive — use sparingly)
   - Animating a screenshot or still (first frame): `minimax-h3` or `gemini-omni-flash-1.1`
   - Match the movie: `16:9:*` sizes for landscape, `9:16:*` for `orientation: "portrait"`
   - Leave `audio` off unless the clip needs its own sound — the movie has narration and music, and Kling charges 50% more for audio
2. **Price the plan** with `estimate_video_cost`, one entry per AI clip, labelled with its scene:

```
estimate_video_cost({
  clips: [
    { label: "Scene 1 — opener", provider: "fal", model: "wan-3.0", size: "16:9:1080p", duration: 5 },
    { label: "Scene 4 — hero shot", provider: "fal", model: "kling-3.0-pro", size: "16:9:1080p", duration: 8 }
  ]
})
→ { clips: [{ label, usd, duration, size, usdPerSecond, warnings }, …], totalUsd, blockers, approximate }
```

   Durations snap **up** to what the model supports — use the returned `duration`/`size`, not your request. Fix any `blockers` (no API key → ask the user to add one in Settings → Video Generation; retired model → pick another) and re-estimate.
3. **Propose the scene layout with costs** — every scene in order, so the user sees where the money goes:

```
| # | Scene | Type | AI clip (model · size · length) | Cost |
|---|-------|------|----------------------------------|------|
| 1 | Opener | AI video | Wan 3.0 · 16:9 1080p · 5s | $1.00 |
| 2 | The problem | React | — | — |
| 3 | Live demo | Screen recording | — | — |
| 4 | Hero shot | AI video | Kling 3.0 Pro · 16:9 1080p · 8s | $0.90 |
|   | **Total AI video** | | 13s | **$1.90** |
```

   Say if the total is approximate (`approximate: true` — token-billed models like Seedance) and mention a cheaper alternative when one exists (e.g. "$0.65 at 720p").
4. **Ask for approval** with `AskUserQuestion` — options like "Approve $1.90", "Use the cheaper plan ($0.65)", "Skip AI video". Only an explicit approval of a dollar amount counts. If the user changes the plan, re-estimate and ask again.
5. Remember each clip's approved `usd` — you'll pass it as `maxCostUsd` in Step 6.

### Step 4: Write Narration

Effective narration:
- Explains the "why" not just the "what"
- Uses active voice ("We add..." not "A function is added...")
- Connects to the bigger picture
- Avoids jargon when possible

**Examples:**

Bad: "Here we see the addition of a new function called handleSubmit."

Good: "The handleSubmit function validates user input before sending it to the API, preventing invalid data from reaching the server."

### Step 5: Read Source Files for Code Scenes

**Always read the actual source file** before creating code view scenes. The diff tells you which lines changed; the file gives you the content with proper surrounding context.

For non-diff modes, read files to get the exact content for the lines you want to show.

### Step 6: Create the Movie

Use the MCP tools in order:

1. **Create the movie** using `create_movie`:

```
create_movie({
  movie: {
    metadata: {
      title: "Add User Authentication",
      description: "JWT-based auth with route protection",
      repository: "acme/web-app",
      branch: "feature/auth",
      commitRange: { from: "abc123", to: "def456" }
    },
    scenes: []
  }
})
→ Returns { id, studioUrl }
```

For non-diff modes, omit `branch` and `commitRange` from metadata — just use `title`, `description`, and optionally `repository`.

**Portrait/Mobile videos:** To create a mobile-optimized video (1080x1920, 9:16), add `orientation: "portrait"` to the movie object. When creating portrait videos:
- Do NOT use `side-by-side` code layout (use `stacked` or `single` instead)
- React view code must target 1080x1920 canvas (use `useVideoConfig()` for dimensions)
- Slide elements use percentages and adapt automatically
- Terminal views work well in portrait without changes

2. **Add scenes** using `create_scene` (see Scene Types below for view structures).

Always include a `title` — a short label (2-5 words) for the scene that appears in the timeline and scene list:

```
create_scene({
  movieId: "<movie-id>",
  scene: {
    title: "Email Validation",
    narration: "We update the user validation logic to check email format.",
    view: {
      type: "code",
      layout: "single",
      codeBlocks: [{
        filePath: "src/validators/user.ts",
        lineRanges: [{ start: 15, end: 28 }],
        changeType: "modify",
        content: "// The actual code content..."
      }]
    }
  }
})
```

Use a mix of scene types for engaging movies: code views for implementation, terminal views for CLI demos, and **React views for most visual/explanatory scenes** (bullet lists, diagrams, overviews, architecture flows). Reserve slide views only for simple title cards and closing slides — for anything with animation, progressive reveal, or visual complexity, use React views instead.

3. **Generate approved AI clips** (only after Step 3b approval). For each AI scene, create it as an empty video scene whose `duration` matches the clip, then generate into it:

```
create_scene({ movieId, scene: {
  title: "Opener",
  narration: "…",
  duration: 5,
  view: { type: "video", source: "", videoFit: "cover" }
}})
→ { id: "<scene-id>" }

generate_scene_video({
  movieId, sceneId: "<scene-id>",
  prompt: "Slow dolly-in on a developer's desk at dawn, phone glowing with a minimal launcher UI, cinematic, shallow depth of field",
  provider: "fal", model: "wan-3.0", size: "16:9:1080p", duration: 5,
  maxCostUsd: 1.00          // the approved per-clip estimate
})
→ { jobId, status: "pending", estimate: { usd: 1.00, … } }
```

   - Pass exactly the provider/model/size/duration/audio you estimated. If the server's estimate exceeds `maxCostUsd` it refuses — **don't raise the cap yourself**; re-estimate and re-ask the user.
   - Start all clips first (they run in parallel), then poll each with `get_scene_video_status({ movieId, sceneId, waitSeconds: 50 })` until `completed` (the clip is now the scene's video) or `failed`. Clips usually take 1–5 minutes.
   - On `failed`, report the error and ask before retrying — a retry is billed again.
   - Cast members (`create_reference_asset`) keep characters consistent: pass their ids as `referenceAssetIds`, and count their images in `referenceImageCount` when estimating.

### Step 7: Return the Studio URL

Use the `studioUrl` returned by `create_movie` and prepend the base URL. Use `$MERGE_MOVIES_URL` if set, otherwise default to `https://merge.mov`.

```
{MERGE_MOVIES_URL}{studioUrl}
```

If the movie has AI clips, also report what was generated and the total estimated spend (sum of the `estimate.usd` values from `generate_scene_video`).

## Scene Types

### Code View Scenes

Display code with syntax highlighting. Layouts:
- `single` - One code block, full width
- `side-by-side` - Compare two files horizontally
- `stacked` - Multiple blocks vertically
- `inline-diff` - Unified diff view

```json
{
  "narration": "We update the user validation logic to check email format.",
  "view": {
    "type": "code",
    "layout": "single",
    "codeBlocks": [{
      "filePath": "src/validators/user.ts",
      "lineRanges": [{ "start": 15, "end": 28 }],
      "changeType": "modify",
      "content": "// The actual code content..."
    }]
  }
}
```

### Slide View Scenes

**Use slide views only for simple title cards and closing/summary slides.** For bullet lists, diagrams, architecture flows, or anything that benefits from animation, use React views instead.

Create simple title cards with positioned elements on a dark canvas.

**Canvas details:**
- Composition: 1920x1080 with 5% inset on all sides (effective area: 90% x 90%)
- All positions use percentages (0-100) relative to the inset area
- Default background: `#0d1117` (customizable via `backgroundColor`)

**Text elements** (`type: "text"`):

| Style | Default size | Weight | Color | Notes |
|-------|-------------|--------|-------|-------|
| `title` | 64px | Bold (700) | `#ffffff` | Main headings |
| `body` | 32px | Normal (400) | `#e6edf3` | Paragraphs |
| `bullet` | 28px | Normal (400) | `#e6edf3` | Blue `•` prefix, split on `\n` |

Use `fontSize` to override the default size. Use `size.width` (percentage) to constrain the text box width.

**Shape elements:**

| Type | Default size | Default stroke | Default fill |
|------|-------------|----------------|-------------|
| `rect` | 30% x 20% | `#58a6ff` | `transparent` |
| `circle` | 15% diameter | `#58a6ff` | `transparent` |
| `line` | start to endPosition | `#58a6ff` | n/a |

Shapes support `text`, `textColor`, and `textFontSize` for centered labels. Rects also support `borderRadius` (0-50).

**Image elements** (`type: "image"`):
- Requires `src` with an uploaded image URL (no direct file upload via API)
- Default width: 40%, height auto-scales to aspect ratio
- Can set explicit `size.width` and `size.height`

**Example: Title card**

```json
{
  "narration": "Let's walk through the authentication changes in this PR.",
  "view": {
    "type": "slide",
    "elements": [
      {
        "id": "title",
        "type": "text",
        "style": "title",
        "content": "Authentication Overhaul",
        "position": { "x": 10, "y": 30 }
      },
      {
        "id": "subtitle",
        "type": "text",
        "style": "body",
        "content": "Adding JWT tokens, refresh flow, and route protection",
        "position": { "x": 10, "y": 50 }
      }
    ]
  }
}
```

**Aesthetic tips:**
- Stick to the dark theme palette: background `#0d1117`, white titles, `#e6edf3` body text, `#58a6ff` accents
- Leave generous white space — don't pack elements to edges
- Keep slides focused — one idea per slide

### React View Scenes

Custom animated scenes using React/JSX with Remotion APIs.

**Before writing React view code from scratch**, search the community template library for a matching starting point:
1. Call `list_react_templates` with a relevant query or tags (e.g. `query: "diagram"`, `tags: ["architecture"]`)
2. If a template fits, call `get_react_template` to get the full code
3. Customize the template code for your scene's specific content (labels, colors, data)

This saves time and produces higher-quality animations. Only write from scratch if no template is a close match.

The `code` field is the body of a React function component that must return JSX. It receives a `scope` object containing Remotion APIs — destructure what you need at the top.

**Available via scope:**

Core: `React`, `useCurrentFrame()`, `useVideoConfig()`, `spring({ frame, fps, config? })`, `interpolate(value, inputRange, outputRange, options?)`, `Easing` (easing curves), `interpolateColors(frame, inputRange, colorRange)`, `random(seed)` (deterministic), `Sequence`, `Series`, `AbsoluteFill`, `Loop`, `Freeze`, `Img`, `Video`, `AuthenticatedVideo`, `useAuthenticatedImage(url)`

Shapes (`@remotion/shapes`): `Circle`, `Rect`, `Star`, `Triangle`, `Ellipse`, `Pie`, `Heart`, `Polygon` — SVG shapes with fill, stroke, and stroke-draw animation support

Paths (`@remotion/paths`): `evolvePath(progress, path)` (animate path drawing 0->1), `getPointAtLength(path, length)`, `getLength(path)`, `interpolatePath(progress, path1, path2)` (morph between paths), `parsePath(d)`, `resetPath`, `scalePath`, `translatePath`, `getBoundingBox`, `getSubpaths`

Noise (`@remotion/noise`): `noise2D(seed, x, y)`, `noise3D(seed, x, y, z)`, `noise4D(seed, x, y, z, w)` — Perlin noise (-1 to 1) for organic wave/particle effects

Animation Utils (`@remotion/animation-utils`): `makeTransform([...transforms])` (compose CSS transforms safely), `interpolateStyles(frame, inputRange, styleRanges)` (interpolate entire style objects)

Transitions (`@remotion/transitions`): `TransitionSeries` (like Series with transitions between children), `springTiming({ config? })`, `linearTiming({ durationInFrames })`, `transitionFade()`, `transitionSlide({ direction? })`, `transitionWipe({ direction? })`, `transitionFlip({ direction? })`, `transitionClockWipe()`

Layout Utils (`@remotion/layout-utils`): `measureText({ text, fontFamily, fontSize })`, `fitText({ text, withinWidth, fontFamily })`

Effects (`@remotion/effects`): `Solid` (a solid-color canvas: `<Solid width height color effects />`) plus effect factories `blur`, `vignette`, `lightLeak`, `noise`, `scanlines`, `shine`, `starburst`, `glow`, `chromaticAberration`, `zoomBlur`, `duotone`, `pixelate`, `colorCorrection`, `saturation`, `hue`, `tint`, `linearGradientTint`, `barrelDistortion`, `wave`. Effects only work through the `effects` prop of `Img`, `Video` or `Solid` (not on plain `div`s), e.g. `<Img src={url} effects={[glow({ radius: 24 }), vignette({ amount: 0.5 })]} />`. `lightLeak` and `shine` take a `progress` (0-1) you drive from the frame. For whole-scene effects, prefer the scene-level `effects` array (see [Effects](#effects)).

3D (`@remotion/three`): `ThreeCanvas` (`<ThreeCanvas width={width} height={height} camera={{ position: [0, 0, 6] }}>`), `THREE` (the three.js namespace) and `useThree`. Inside the canvas use R3F elements (`<mesh>`, `<boxGeometry />`, `<meshStandardMaterial />`, `<ambientLight />`, `<directionalLight />`). Animate only from `useCurrentFrame()` (`useFrame` isn't available), load nothing over the network (no GLTF, textures or 3D fonts; overlay text as DOM), and keep scenes modest because renders use software WebGL. See the create-react-template skill for a full example.

```json
{
  "narration": "Welcome to the code walkthrough.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, useVideoConfig, spring, interpolate, AbsoluteFill } = scope;\n\nconst frame = useCurrentFrame();\nconst { fps } = useVideoConfig();\n\nconst opacity = interpolate(frame, [0, 30], [0, 1], { extrapolateRight: 'clamp' });\nconst scale = spring({ frame, fps, config: { damping: 200 } });\n\nreturn (\n  <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}>\n    <div style={{ opacity, transform: `scale(${scale})`, fontSize: 80, color: '#58a6ff' }}>\n      Hello World\n    </div>\n  </AbsoluteFill>\n);",
    "backgroundColor": "#0d1117"
  }
}
```

**React views are the preferred scene type for all visual/explanatory content** — bullet lists, diagrams, architecture flows, feature overviews, and anything that isn't pure code or terminal output. They produce more engaging, animated scenes than static slides.

**Example: Animated bullet list**

```json
{
  "narration": "Here's what we'll cover in this walkthrough.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, interpolate, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\n\nconst items = [\n  'New User model with password hashing',\n  'JWT-based auth controller',\n  'Route protection middleware',\n  'Updated API routes'\n];\n\nconst titleOpacity = interpolate(frame, [0, 20], [0, 1], { extrapolateRight: 'clamp' });\n\nreturn (\n  <AbsoluteFill style={{ display: 'flex', flexDirection: 'column', justifyContent: 'center', padding: '0 120px', backgroundColor: '#0d1117' }}>\n    <div style={{ opacity: titleOpacity, fontSize: 64, fontWeight: 700, color: '#ffffff', marginBottom: 48 }}>\n      What Changed\n    </div>\n    {items.map((item, i) => {\n      const delay = 20 + i * 15;\n      const opacity = interpolate(frame, [delay, delay + 15], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      const translateX = interpolate(frame, [delay, delay + 15], [30, 0], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      return (\n        <div key={i} style={{ opacity, transform: `translateX(${translateX}px)`, fontSize: 36, color: '#e6edf3', marginBottom: 32, display: 'flex', alignItems: 'center', gap: 16 }}>\n          <span style={{ color: '#58a6ff', fontSize: 28 }}>•</span>\n          {item}\n        </div>\n      );\n    })}\n  </AbsoluteFill>\n);",
    "backgroundColor": "#0d1117"
  }
}
```

**Example: Architecture diagram with animated flow**

```json
{
  "narration": "The request flows from client through middleware to the API handler.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, interpolate, spring, useVideoConfig, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\nconst { fps, height } = useVideoConfig();\n\nconst boxes = [\n  { label: 'Client', color: '#58a6ff', x: 160 },\n  { label: 'Auth Middleware', color: '#d2a8ff', x: 600 },\n  { label: 'API Handler', color: '#3fb950', x: 1100 }\n];\n\nreturn (\n  <AbsoluteFill style={{ display: 'flex', flexDirection: 'column', justifyContent: 'center', alignItems: 'center', backgroundColor: '#0d1117' }}>\n    <div style={{ fontSize: 48, fontWeight: 700, color: '#fff', marginBottom: 80 }}>\n      Request Flow\n    </div>\n    <div style={{ display: 'flex', alignItems: 'center', gap: 48 }}>\n      {boxes.map((box, i) => {\n        const delay = i * 20;\n        const scale = spring({ frame: frame - delay, fps, config: { damping: 200 } });\n        const opacity = interpolate(frame, [delay, delay + 10], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n        const arrowOpacity = i < boxes.length - 1 ? interpolate(frame, [delay + 10, delay + 20], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }) : 0;\n        return (\n          <div key={i} style={{ display: 'flex', alignItems: 'center', gap: 48 }}>\n            <div style={{ opacity, transform: `scale(${scale})`, border: `2px solid ${box.color}`, borderRadius: 12, padding: '24px 36px', color: box.color, fontSize: 24, fontWeight: 600 }}>\n              {box.label}\n            </div>\n            {i < boxes.length - 1 && <div style={{ opacity: arrowOpacity, color: '#58a6ff', fontSize: 32 }}>→</div>}\n          </div>\n        );\n      })}\n    </div>\n  </AbsoluteFill>\n);",
    "backgroundColor": "#0d1117"
  }
}
```

**React view tips:**
- All animation should be frame-driven (`useCurrentFrame`), not time-based or stateful
- Use inline styles only — no CSS imports or className
- Stagger animations by subtracting offsets: `interpolate(frame - 20, [0, 30], [0, 1], { extrapolateLeft: 'clamp' })`
- `spring()` returns 0->1 by default — use for scale, translateY, opacity
- The canvas is 1920x1080 (landscape) or 1080x1920 (portrait) at 30fps. A 5-second scene = 150 frames
- **Use the full viewport — both axes** — This is critical. Content must fill the entire canvas (1920x1080 landscape, 1080x1920 portrait), not cluster in one corner or the top half. Vertically: use `display: 'flex', flexDirection: 'column', justifyContent: 'center'` on the root `AbsoluteFill` to vertically center content, or `justifyContent: 'space-between'` with generous padding to distribute elements across the full height. Never leave the bottom half of the screen empty. Horizontally: spread elements across the full width — cards in a row should span the available space with even gaps, not huddle on one side. Use the full 1920px (landscape) or 1080px (portrait) width.
- Prefer React views over slide views for bullet lists, diagrams, architecture flows, and overviews
- **Use literal Unicode characters, not escape sequences** — The `code` field is a string, so `\u2713` renders as the literal text `\u2713`, not a checkmark. Always paste the actual character (e.g. `✓`, `→`, `•`, `ℹ️`) directly into the code string

### Terminal View Scenes

Display terminal sessions with animated command input and output.

```json
{
  "narration": "First, install the dependencies and run the tests.",
  "view": {
    "type": "terminal",
    "title": "~/project",
    "theme": "generic",
    "entries": [
      {
        "id": "install",
        "command": "npm install",
        "output": "added 127 packages in 4s"
      },
      {
        "id": "test",
        "command": "npm test",
        "output": "Tests: 12 passed, 12 total\nTime: 2.4s",
        "exitCode": 0
      }
    ],
    "inputAnimation": "type",
    "outputAnimation": "fade"
  }
}
```

**Terminal options:**
- `theme`: `"generic"`, `"claude-code"`, or `"codex"` — styles the prompt
- `entryType` (per entry): Override the theme for individual entries
- `inputAnimation`: `"type"` (typewriter), `"fade"`, or `"cut"` (instant)
- `outputAnimation`: `"type"`, `"fade"`, or `"cut"`
- `prompt`: Custom prompt text override per entry

## Change Type Guidelines

- `modify` - Existing file being changed. This is the most common type — use it whenever showing changes to a file that already exists, even if the scene shows added lines within that file.
- `add` - Brand-new files or entirely new functions/classes being introduced
- `delete` - File or code being removed
- `context` - Unchanged code shown purely for reference, no changes being discussed

**Important:** If a commit modifies an existing file (adds lines, changes lines, or removes lines within it), use `modify` — not `add`. Most scenes in a typical movie should use `modify`.

**For non-diff modes:** Use `context` for walkthrough/architecture scenes where you're showing existing code without changes. Use `add` only when showing newly created files.

## Code Context Guidelines

When creating code view scenes, provide proper context so viewers can understand where changes occur in the source file. Without context, viewers see isolated changed lines and can't understand the surrounding code structure.

### Rules

1. **Always read the source file first** — Never paste raw diff hunks as content. Use the Read tool to get the actual file content. The diff tells you *which* lines changed; the file gives you the *content* including surrounding context.
2. **Include 3-5 context lines before and after each change** — Viewers need to see what surrounds the changed code.
3. **Show the enclosing structure** — When a change is inside a function, class, or block, include the function/class signature and closing brace. Use a separate lineRange for the header if it's far from the change, which creates a `...` gap separator.
4. **Content must contain ALL lines covered by lineRanges** — The number of lines in `content` must exactly equal the total number of lines spanned by all lineRanges.
5. **Use `lineRanges` with actual source line numbers** — The renderer uses these to display correct line numbers and visual gap separators (`...`) between non-contiguous sections.
6. **Use multiple lineRanges for non-contiguous sections** — When showing a function header + a change deeper inside, use separate ranges.
7. **Highlights reference actual line numbers** — The `lines` array in highlights must use real source file line numbers.

### Example: Good vs Bad

**Bad** (no context, just the changed lines):
```json
{
  "narration": "We add email validation.",
  "view": {
    "type": "code",
    "layout": "single",
    "codeBlocks": [{
      "filePath": "src/validators/user.ts",
      "lineRanges": [{ "start": 47, "end": 50 }],
      "changeType": "modify",
      "content": "  const email = input.email.trim();\n  if (!isValidEmail(email)) {\n    throw new ValidationError('Invalid email format');\n  }"
    }]
  }
}
```

**Good** (enclosing function, 3+ context lines before/after change):
```json
{
  "narration": "We add email validation inside validateUser.",
  "view": {
    "type": "code",
    "layout": "single",
    "codeBlocks": [{
      "filePath": "src/validators/user.ts",
      "lineRanges": [
        { "start": 38, "end": 40 },
        { "start": 44, "end": 55 }
      ],
      "changeType": "modify",
      "content": "export function validateUser(input: UserInput): ValidationResult {\n  const errors: string[] = [];\n\n  const name = input.name?.trim();\n  if (!name || name.length < 2) {\n    errors.push('Name must be at least 2 characters');\n  }\n  const email = input.email.trim();\n  if (!isValidEmail(email)) {\n    throw new ValidationError('Invalid email format');\n  }\n  const age = Number(input.age);\n  if (isNaN(age) || age < 0 || age > 150) {\n    errors.push('Invalid age');\n  }\n  return { valid: errors.length === 0, errors };"
    }],
    "animations": {
      "highlights": [
        { "id": "h1", "lines": [50, 51, 52, 53], "color": "rgba(255, 213, 79, 0.3)" }
      ]
    }
  }
}
```

## Code View Animations

Animations bring code to life by scrolling through long files.

### Scrolling Animation

For files too long to fit on screen, use scrolling to navigate through the code during playback.

**Note:** A 2-second pause at the start of scrolling blocks is automatically added by the backend — you don't need to include it.

```json
{
  "narration": "Let's walk through this implementation step by step.",
  "view": {
    "type": "code",
    "layout": "single",
    "codeBlocks": [{
      "filePath": "src/services/auth.ts",
      "lineRanges": [{ "start": 1, "end": 60 }],
      "changeType": "add",
      "content": "// Long file content..."
    }],
    "animations": {
      "scroll": {
        "id": "scroll1",
        "linesPerSecond": 3,
        "pauses": [
          { "lineNumber": 15, "durationMs": 2000 },
          { "lineNumber": 42, "durationMs": 3000 }
        ]
      }
    }
  }
}
```

**Scroll parameters:**
- `linesPerSecond` - Scroll speed (0.1 to 50, recommend 2-5 for readability). Enables scroll animation.
- `pauses` - Stop at specific lines to let viewers read important code
- `targetBlockId` - Which code block to scroll (optional, defaults to first block)

**Tips:**
- Scrolling only activates when content exceeds viewport height
- If you omit the scroll animation, long content will still auto-scroll at 1.5 lines/sec with a start pause
- Use pauses to draw attention to key implementation details
- Match scroll speed to narration pace

### Line Highlighting

Highlight important lines in code view scenes with a colored background.

Place highlights inside `view.animations.highlights`:

```json
{
  "view": {
    "type": "code",
    "codeBlocks": [{ "..." : "..." }],
    "animations": {
      "highlights": [
        {
          "id": "h1",
          "lines": [5, 6, 7, 8],
          "color": "rgba(255, 213, 79, 0.3)"
        }
      ]
    }
  }
}
```

**Each highlight object:**

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Highlight ID |
| `lines` | Yes | Source line numbers to highlight (must be within code block's lineRanges) |
| `targetBlockId` | No | Which code block to highlight (defaults to first). **Must match one of the code block IDs in the same request** — see warning below |
| `color` | No | RGBA color string (default: yellow `rgba(255, 213, 79, 0.3)`) |
| `startTimeMs` | No | Start time in ms from scene start (omit for whole scene) |
| `endTimeMs` | No | End time in ms from scene start (omit for whole scene) |
| `glow` | No | Enable glow effect (pulsing box-shadow) |
| `size` | No | Enable size effect (scale up highlighted lines) |
| `focus` | No | Enable focus effect (zoom in and blur non-highlighted lines) |

> **Common Pitfall: Non-contiguous lineRanges**
>
> When using multiple `lineRanges` (e.g., `[{start: 3, end: 13}, {start: 54, end: 58}]`),
> highlight `lines` must use **source file line numbers** (e.g., `[54, 55, 56]`), NOT content string
> positions. The API validates this and will reject highlights referencing lines outside the declared ranges.

> **Common Pitfall: targetBlockId must match a code block ID in the same request**
>
> When using `targetBlockId` on highlights (or scroll animations), the value **must exactly match**
> one of the `id` fields you provide on the `codeBlocks` array in the same API call. The server
> remaps all code block IDs to UUIDs internally — if a `targetBlockId` doesn't match any code block
> ID in the request, the highlight will be **silently removed** and a warning returned.
>
> For example, if your code blocks use `"id": "left-block"` and `"id": "right-block"`, then
> highlights must use `"targetBlockId": "left-block"` or `"targetBlockId": "right-block"`.
> Using any other value (like `"cb-left"`) will cause the highlight to be dropped.

**Preset colors:**
- Yellow: `rgba(255, 213, 79, 0.3)` (default)
- Blue: `rgba(96, 165, 250, 0.3)`
- Purple: `rgba(192, 132, 252, 0.3)`
- Orange: `rgba(251, 146, 60, 0.3)`
- Cyan: `rgba(34, 211, 238, 0.3)`
- Pink: `rgba(244, 114, 182, 0.3)`

**Timed highlights (sequential):**

```json
{
  "view": {
    "type": "code",
    "codeBlocks": [{
      "filePath": "src/services/parser.ts",
      "lineRanges": [{ "start": 1, "end": 30 }],
      "changeType": "add",
      "content": "// code..."
    }],
    "animations": {
      "highlights": [
        { "id": "h1", "lines": [3, 4, 5], "color": "rgba(96, 165, 250, 0.3)", "startTimeMs": 0, "endTimeMs": 3000 },
        { "id": "h2", "lines": [12, 13, 14], "color": "rgba(244, 114, 182, 0.3)", "startTimeMs": 3000, "endTimeMs": 6000 }
      ]
    }
  }
}
```

**Highlights combined with scroll:**

```json
{
  "view": {
    "type": "code",
    "codeBlocks": [{
      "filePath": "src/services/auth.ts",
      "lineRanges": [{ "start": 1, "end": 60 }],
      "changeType": "add",
      "content": "// long file..."
    }],
    "animations": {
      "scroll": {
        "id": "scroll1",
        "linesPerSecond": 3,
        "pauses": [
          { "lineNumber": 15, "durationMs": 3000 },
          { "lineNumber": 40, "durationMs": 3000 }
        ]
      },
      "highlights": [
        { "id": "h1", "lines": [15, 16, 17], "color": "rgba(34, 211, 238, 0.3)" },
        { "id": "h2", "lines": [40, 41, 42, 43], "color": "rgba(192, 132, 252, 0.3)" }
      ]
    }
  }
}
```

## Transitions

Control how scenes transition in and out using `startTransition` and `endTransition` fields on the scene:

```json
{
  "narration": "Now let's look at the implementation.",
  "startTransition": { "type": "fade", "duration": 0.5 },
  "endTransition": { "type": "fade", "duration": 0.3 },
  "view": { "..." : "..." }
}
```

| Type | When to use |
|------|-------------|
| `cut` | Between closely related scenes (same topic, quick progression) |
| `fade` | Between distinct topics or for dramatic effect |
| `slide` | For sequential steps or progression |
| `zoom` | For drilling into details |

## Effects

Scenes can layer cinematic visual effects on top of the view with an optional `effects` array (max 8, applied in order): light leaks, vignette, film grain, scanlines, shine sweeps, starbursts, gradient tints, color filters, and video-only pixel effects. Effects preview live in Studio and render identically in the final video.

```json
{
  "title": "Tests Pass",
  "narration": "With the retry logic in place, the full suite goes green — including the flaky upload test.",
  "view": { "type": "terminal", "..." : "..." },
  "effects": [
    { "type": "scanlines", "params": { "amount": 0.3 } },
    { "type": "vignette", "params": { "amount": 0.5 } },
    { "type": "shine", "params": { "intensity": 0.55 }, "startMs": 3000, "endMs": 5000 }
  ]
}
```

**Each effect object:**

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Effect type from the catalog below |
| `params` | No | Effect parameters. Omitted params use defaults; out-of-range values are clamped |
| `startMs` | No | Start time in ms from scene start (default: 0) |
| `endMs` | No | End time in ms from scene start (default: end of scene) |
| `fadeInMs` | No | Ease the effect in over this many ms |
| `fadeOutMs` | No | Ease the effect out over this many ms |
| `enabled` | No | `false` keeps the effect but hides it (default: `true`) |
| `id` | No | Stable ID for editing (generated if omitted) |

**Catalog** — overlays and filters work on every scene type; video-only effects need a canvas source and **only render on `video` scenes**:

| Type | What it looks like | Key params (range, default) | Scope |
|------|--------------------|-----------------------------|-------|
| `lightLeak` | Warm film light leak that blooms across the frame and retracts over ~2s. Great on scene openers, reveals and celebratory moments; set startMs to time it to a beat. | `intensity` 0..1 (0.55), `duration` 0.5..10 s (2), `hueShift` 0..360 (0), `seed` 0..100 (3) | All scenes (overlay) |
| `vignette` | Darkens the edges to pull focus toward the center. Subtle and cinematic; safe on any scene. | `amount` 0..1 (0.55), `radius` 0..1 (0.6), `feather` 0..1 (0.45), `color` color (#000000) | All scenes (overlay) |
| `filmGrain` | Animated analog film grain. Adds texture and a cinematic, organic feel. | `amount` 0..1 (0.35), `animated` boolean (true) | All scenes (overlay) |
| `scanlines` | CRT-style horizontal scanlines that slowly scroll. Retro, terminal and hacker vibes. | `amount` 0..1 (0.35), `spacing` 2..16 (4), `thickness` 1..8 (1), `speed` 0..200 (30) | All scenes (overlay) |
| `shine` | A glossy light streak that sweeps across the frame once (~1.2s). Perfect for highlighting a result or a hero moment; set startMs to time it. | `intensity` 0..1 (0.6), `duration` 0.3..6 s (1.2), `angle` 0..180 (30), `width` 20..400 (160) | All scenes (overlay) |
| `starburst` | Slowly rotating sunburst rays behind the content. Energetic and bold; use for titles, launches and wins. | `intensity` 0..1 (0.25), `rays` 4..48 (16), `color` color (#f59e0b), `rotationSpeed` -90..90 (8), `blend` screen/soft-light/overlay/color/multiply/normal (screen) | All scenes (overlay) |
| `gradientTint` | Washes the scene in a two-color gradient. Use to give a movie a consistent brand color mood. | `amount` 0..1 (0.45), `startColor` color (#7c3aed), `endColor` color (#06b6d4), `angle` 0..360 (135), `blend` screen/soft-light/overlay/color/multiply/normal (soft-light) | All scenes (overlay) |
| `blur` | Gaussian blur of the whole scene. Fade it in/out for a dreamy focus pull, or keep it subtle behind narration. | `radius` 0..60 (12) | All scenes (filter) |
| `brightness` | Brighten (positive) or darken (negative) the scene. | `amount` -1..1 (0.15) | All scenes (filter) |
| `contrast` | Increase (>1) or flatten (<1) contrast. | `amount` 0..3 (1.25) | All scenes (filter) |
| `saturation` | Boost (>1) or mute (<1) colors. 0 is fully desaturated. | `amount` 0..3 (1.4) | All scenes (filter) |
| `grayscale` | Removes color. Use for flashbacks, "before" states or a documentary look. | `amount` 0..1 (1) | All scenes (filter) |
| `hue` | Rotates every color around the color wheel. Playful and surreal. | `degrees` -180..180 (90) | All scenes (filter) |
| `invert` | Inverts colors (turns dark mode into light mode). Striking as a quick flash. | `amount` 0..1 (1) | All scenes (filter) |
| `colorGrade` | Professional color correction: exposure, contrast, temperature, tint, saturation and vibrance in one pass. | `exposure` -2..2 (0), `contrast` 0.5..2 (1.1), `temperature` -1..1 (0.15), `tint` -1..1 (0), `saturation` 0..2 (1.1), `vibrance` -1..1 (0.2) | Video scenes only |
| `tint` | Tints the footage toward a single color. | `color` color (#22d3ee), `amount` 0..1 (0.35) | Video scenes only |
| `duotone` | Maps shadows and highlights to two colors. Bold, graphic, poster-like. | `darkColor` color (#1e1b4b), `lightColor` color (#f472b6), `threshold` 0..1 (0.35) | Video scenes only |
| `chromaticAberration` | Splits RGB channels for a glitchy lens fringe. Use briefly for impact or a glitch aesthetic. | `amount` 0..40 (8), `angle` 0..360 (0) | Video scenes only |
| `zoomBlur` | Radial motion blur rushing toward a point. Adds speed and punch. | `amount` 0..120 (40), `center` [x,y] 0..1 ([0.5, 0.5]) | Video scenes only |
| `glow` | Blooms bright areas with a soft glow. Dreamy, neon, polished. | `intensity` 0..3 (1), `radius` 1..80 (24), `threshold` 0..1 (0.5), `color` color (#ffffff) | Video scenes only |
| `pixelate` | Chunky pixel mosaic. Retro 8-bit look, or censor-style obscuring. | `blockSize` 1..64 (12) | Video scenes only |
| `crt` | Bulges the image like an old CRT monitor. Pair with Scanlines for a full retro look. | `amount` 0..1 (0.25) | Video scenes only |

Colors are hex strings (`"#f59e0b"`); `center` is `[x, y]` from 0 to 1. Animated effects (`lightLeak`, `shine`, `starburst`, `filmGrain`, `scanlines`) move on their own over their time window — `lightLeak` and `shine` play once, starting at `startMs` and lasting their `duration` param (2s / 1.2s by default), so use `startMs` to land them on the narration beat.

**Presets** — copy-pasteable looks (the same ones offered in Studio):

**Cinematic** — Vignette, fine grain and a touch of contrast.

```json
"effects": [{"type":"contrast","params":{"amount":1.12}},{"type":"vignette","params":{"amount":0.5,"radius":0.62,"feather":0.5}},{"type":"filmGrain","params":{"amount":0.22}}]
```

**Golden Hour** — Warm light leak with a soft amber wash.

```json
"effects": [{"type":"lightLeak","params":{"intensity":0.65,"hueShift":0}},{"type":"gradientTint","params":{"amount":0.3,"startColor":"#f97316","endColor":"#fde047","angle":120,"blend":"soft-light"}}]
```

**Retro CRT** — Rolling scanlines, grain and a heavy vignette.

```json
"effects": [{"type":"scanlines","params":{"amount":0.4,"spacing":4,"thickness":1,"speed":24}},{"type":"filmGrain","params":{"amount":0.25}},{"type":"vignette","params":{"amount":0.6,"radius":0.55,"feather":0.4}}]
```

**Neon Night** — Purple-cyan wash with punchy saturation.

```json
"effects": [{"type":"saturation","params":{"amount":1.35}},{"type":"gradientTint","params":{"amount":0.5,"startColor":"#7c3aed","endColor":"#06b6d4","angle":135,"blend":"soft-light"}},{"type":"vignette","params":{"amount":0.45}}]
```

**Flashback** — Black & white, dreamy blur-in and heavy grain.

```json
"effects": [{"type":"grayscale","params":{"amount":1}},{"type":"filmGrain","params":{"amount":0.4}},{"type":"vignette","params":{"amount":0.7}},{"type":"blur","params":{"radius":10},"fadeOutMs":800}]
```

**Hero Moment** — Rotating starburst plus a shine sweep. For launches and big wins.

```json
"effects": [{"type":"starburst","params":{"intensity":0.22,"rays":18,"color":"#f59e0b"}},{"type":"shine","params":{"intensity":0.55}},{"type":"vignette","params":{"amount":0.4}}]
```

**Taste guidance:**

- **Less is more** — 1-3 effects per scene. Most code scenes need at most a subtle baseline; effects should support the story, not decorate every frame.
- **Keep a consistent look** — pick one baseline (e.g. the Cinematic preset, or `vignette` + `filmGrain`) and reuse it across the movie, then add accents on specific moments.
- **Motion effects mark moments** — use `lightLeak`, `shine` or `starburst` on openers, reveals and wins (title card, "tests pass", the final result), not on every scene.
- **Cinematic baseline** — `filmGrain` (~0.2) + `vignette` (~0.5) is safe on anything.
- **Terminal scenes** — `scanlines` (optionally with `vignette`) fits the CLI aesthetic.
- **Video-only effects only on video scenes** — `colorGrade`, `tint`, `duotone`, `chromaticAberration`, `zoomBlur`, `glow`, `pixelate` and `crt` are ignored on code/slide/terminal/react scenes.
- **Time effects to the narration beat** — use `startMs`/`endMs` to fire a `shine` when the narration reaches the result, and `fadeInMs`/`fadeOutMs` so filters ease in and out instead of popping.
- **Keep code readable** — don't put heavy `blur`, `invert`, `hue` or strong tints on code the viewer must read. A `blur` that fades out in the first second (like Flashback) is fine.

## Scene Ordering Tips

1. **Start with context** - Title card, overview, or what exists before changes
2. **Introduce the problem** - Why is this change/feature needed?
3. **Show the solution** - Walk through the implementation
4. **Demonstrate the result** - Tests, usage, terminal output, or UI changes

## Tips for Great Movies

1. **Keep it focused** - One topic per movie, don't try to cover too much
2. **Tell a story** - Have a beginning, middle, and end
3. **Show, don't just tell** - Let the code speak when possible
4. **Use transitions** - Fade between unrelated scenes, cut between related ones
5. **Use effects sparingly** - A consistent subtle baseline plus a motion accent (light leak, shine) on the big moments
6. **Prefer React views for visual content** - Use React views for bullet lists, diagrams, architecture flows, and overviews. Only use slide views for simple title cards and closing slides.
7. **Use animations for long files** - Scroll through large files instead of cramming everything
8. **Time animations to narration** - Sync scroll pauses with what you're explaining
9. **Mix scene types** - Alternate between code, react, and terminal views to keep viewers engaged
