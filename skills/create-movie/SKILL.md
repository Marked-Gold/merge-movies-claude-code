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

This skill uses the `merge-movies` MCP server. All tools are available automatically — use them directly by name. Authentication is handled by the MCP transport via the `MERGE_MOVIES_API_KEY` environment variable.

If the key is missing, tell the user to create one at https://merge.mov/settings.

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

### Step 7: Return the Studio URL

Use the `studioUrl` returned by `create_movie` and prepend the base URL. Use `$MERGE_MOVIES_URL` if set, otherwise default to `https://merge.mov`.

```
{MERGE_MOVIES_URL}{studioUrl}
```

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

The `code` field is the body of a React function component that must return JSX. It receives a `scope` object containing Remotion APIs — destructure what you need at the top.

**Available via scope:**
- `React` — the React library
- `useCurrentFrame()` — returns the current frame number (0-indexed)
- `useVideoConfig()` — returns `{ fps, durationInFrames, width, height }` (standard: 30fps, 1920x1080)
- `spring({ frame, fps, config? })` — physics-based animation (0->1). Config: damping, mass, stiffness, overshootClamping
- `interpolate(value, inputRange, outputRange, options?)` — map a value between ranges. Options: `{ extrapolateLeft, extrapolateRight }` with `'clamp'|'extend'|'wrap'`
- `Sequence` — render children only during a frame range. Props: `from`, `durationInFrames`
- `Series` — sequential Sequence blocks. Children are `<Series.Sequence durationInFrames={n}>` elements
- `AbsoluteFill` — full-screen absolutely-positioned container (1920x1080)
- `Img` — Remotion's image component

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
    "code": "const { useCurrentFrame, interpolate, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\n\nconst items = [\n  'New User model with password hashing',\n  'JWT-based auth controller',\n  'Route protection middleware',\n  'Updated API routes'\n];\n\nconst titleOpacity = interpolate(frame, [0, 20], [0, 1], { extrapolateRight: 'clamp' });\n\nreturn (\n  <AbsoluteFill style={{ padding: '96px 120px', backgroundColor: '#0d1117' }}>\n    <div style={{ opacity: titleOpacity, fontSize: 64, fontWeight: 700, color: '#ffffff', marginBottom: 48 }}>\n      What Changed\n    </div>\n    {items.map((item, i) => {\n      const delay = 20 + i * 15;\n      const opacity = interpolate(frame, [delay, delay + 15], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      const translateX = interpolate(frame, [delay, delay + 15], [30, 0], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      return (\n        <div key={i} style={{ opacity, transform: `translateX(${translateX}px)`, fontSize: 28, color: '#e6edf3', marginBottom: 20, display: 'flex', alignItems: 'center', gap: 16 }}>\n          <span style={{ color: '#58a6ff', fontSize: 24 }}>&#x2022;</span>\n          {item}\n        </div>\n      );\n    })}\n  </AbsoluteFill>\n);",
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
    "code": "const { useCurrentFrame, interpolate, spring, useVideoConfig, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\nconst { fps } = useVideoConfig();\n\nconst boxes = [\n  { label: 'Client', color: '#58a6ff', x: 160 },\n  { label: 'Auth Middleware', color: '#d2a8ff', x: 600 },\n  { label: 'API Handler', color: '#3fb950', x: 1100 }\n];\n\nreturn (\n  <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center', backgroundColor: '#0d1117' }}>\n    <div style={{ fontSize: 48, fontWeight: 700, color: '#fff', position: 'absolute', top: 96 }}>\n      Request Flow\n    </div>\n    {boxes.map((box, i) => {\n      const delay = i * 20;\n      const scale = spring({ frame: frame - delay, fps, config: { damping: 200 } });\n      const opacity = interpolate(frame, [delay, delay + 10], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      return (\n        <div key={i} style={{ position: 'absolute', left: box.x, top: 420, opacity, transform: `scale(${scale})`, border: `2px solid ${box.color}`, borderRadius: 12, padding: '24px 36px', color: box.color, fontSize: 24, fontWeight: 600 }}>\n          {box.label}\n        </div>\n      );\n    })}\n    {[0, 1].map(i => {\n      const arrowDelay = 10 + i * 20;\n      const opacity = interpolate(frame, [arrowDelay + 10, arrowDelay + 20], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      const x = i === 0 ? 380 : 870;\n      return (\n        <div key={`arrow-${i}`} style={{ position: 'absolute', left: x, top: 446, opacity, color: '#58a6ff', fontSize: 32 }}>\n          &#x2192;\n        </div>\n      );\n    })}\n  </AbsoluteFill>\n);",
    "backgroundColor": "#0d1117"
  }
}
```

**React view tips:**
- All animation should be frame-driven (`useCurrentFrame`), not time-based or stateful
- Use inline styles only — no CSS imports or className
- Stagger animations by subtracting offsets: `interpolate(frame - 20, [0, 30], [0, 1], { extrapolateLeft: 'clamp' })`
- `spring()` returns 0->1 by default — use for scale, translateY, opacity
- The canvas is 1920x1080 at 30fps. A 5-second scene = 150 frames
- Design for the full viewport — use `AbsoluteFill` as root and distribute elements across the full 1080px height
- Prefer React views over slide views for bullet lists, diagrams, architecture flows, and overviews

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
| `targetBlockId` | No | Which code block (defaults to first) |
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
5. **Prefer React views for visual content** - Use React views for bullet lists, diagrams, architecture flows, and overviews. Only use slide views for simple title cards and closing slides.
6. **Use animations for long files** - Scroll through large files instead of cramming everything
7. **Time animations to narration** - Sync scroll pauses with what you're explaining
8. **Mix scene types** - Alternate between code, react, and terminal views to keep viewers engaged
