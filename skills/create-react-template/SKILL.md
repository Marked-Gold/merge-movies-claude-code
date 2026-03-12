# Create React Template Skill

Create reusable React animation templates (snippets) for the merge.mov template library.

$ARGUMENTS

## Overview

This skill creates standalone React animation templates that can be reused across movies. Templates are saved to the user's personal snippet library and can be shared with all Merge Movies users.

1. Understand what the user wants to visualize
2. Search existing templates for a starting point
3. Write or customize the React animation code
4. Save it as a template with good name, description, and tags

## Usage

- `/merge-movies:create-react-template animated progress bar` - Create from a description
- `/merge-movies:create-react-template` - Interactive mode

## MCP Tools

This skill uses the `merge-movies` MCP server. Authentication is handled automatically via OAuth.

| Tool | Description |
|------|-------------|
| `list_react_templates` | Search existing templates by keyword or tags |
| `get_react_template` | Get full code of an existing template |
| `create_react_template` | Save a new template to the user's library |

## Workflow

### Step 1: Understand the Request

Parse `$ARGUMENTS` to understand what kind of animation template the user wants. If unclear, ask. Common categories:
- **Diagrams**: flow charts, architecture, hub-and-spoke, layer stacks
- **Lists**: bullet points, numbered steps, checklists
- **Data viz**: progress bars, charts, counters, timelines
- **Text**: title cards, quote displays, feature highlights
- **Decorative**: backgrounds, transitions, loaders

### Step 2: Search Existing Templates

Call `list_react_templates` to check if a similar template already exists:
- Use `query` for keyword search (e.g. "diagram", "progress")
- Use `tags` for category filtering (e.g. `["diagram", "architecture"]`)

If a close match exists, call `get_react_template` to get the full code and use it as a starting point. Customize rather than rewrite.

### Step 3: Write the Template Code

The `code` field is the body of a React function component that receives a `scope` object with Remotion APIs.

**Available via scope:**
- `React` — the React library
- `useCurrentFrame()` — current frame number (0-indexed)
- `useVideoConfig()` — returns `{ fps, durationInFrames, width, height }`
- `spring({ frame, fps, config? })` — physics-based animation (0->1)
- `interpolate(value, inputRange, outputRange, options?)` — map values between ranges
- `Sequence` — render children during a frame range
- `Series` — sequential Sequence blocks
- `AbsoluteFill` — full-screen container
- `Img` — Remotion image component

**Code rules:**
- All animation must be frame-driven (`useCurrentFrame`), not time-based or stateful
- Use inline styles only — no CSS imports or className
- Use `useVideoConfig()` for width/height — do not hardcode 1920/1080
- Use `extrapolateLeft: 'clamp'` and `extrapolateRight: 'clamp'` on interpolations
- Use literal Unicode characters, not escape sequences (paste `→` not `\u2192`)
- Design for 1920x1080 landscape at 30fps (150 frames = 5 seconds)
- Use the dark theme: background `#0d1117`, text `#e6edf3`, accent `#58a6ff`

**Quality guidelines:**
- Stagger animations for visual interest (don't reveal everything at once)
- Use `spring()` for organic motion, `interpolate()` for precise control
- Leave generous whitespace — don't pack elements to edges
- **Use the full viewport — both axes** — Content must fill the entire canvas, not cluster in the top half or one corner. Vertically: use `display: 'flex', flexDirection: 'column', justifyContent: 'center'` on the root `AbsoluteFill` to center content, or `justifyContent: 'space-between'` with padding to distribute across the full height. Horizontally: spread elements across the full width — cards in a row should span the available space with even gaps.
- Make data items (labels, colors, counts) easy to customize by putting them in arrays/objects at the top

### Step 4: Create the Template

Call `create_react_template` with:
- `name`: Clear, descriptive (e.g. "Animated Timeline", "Hub-and-Spoke Diagram")
- `description`: What it visualizes and when to use it (1-2 sentences)
- `code`: The React component body
- `tags`: 2-4 tags for discoverability. Use existing tag conventions:
  - Categories: `diagram`, `list`, `data`, `text`, `progress`, `chart`
  - Styles: `animation`, `stagger`, `spring`, `gradient`
  - Use cases: `architecture`, `flow`, `overview`, `comparison`

### Step 5: Return Result

Tell the user the template was created and they can view/preview it at `/snippets` in the app. If they want a portrait version too, offer to create a second template with the `portrait` tag and layout adjusted for 1080x1920.

## Tips

- Templates should be **generic and reusable** — use placeholder data that's easy to swap out
- A good template has 3-5 data items to demonstrate the pattern (not too few, not too many)
- If the user provides specific content, still structure the code so data is separated from layout
- Consider creating both landscape and portrait versions for maximum reuse
