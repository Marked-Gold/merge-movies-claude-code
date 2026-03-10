# Create Mobile Video Skill

Convert an existing landscape merge.mov movie into a portrait (9:16) mobile-optimized version.

$ARGUMENTS

## Overview

This skill takes an existing movie and creates a new portrait-oriented copy optimized for mobile platforms (TikTok, Reels, Shorts). It:
1. Fetches the source movie
2. Creates a new movie with `orientation: "portrait"` (1080x1920)
3. Re-creates each scene adapted for portrait layout
4. Returns the studio URL for the new mobile version

## Usage

- `/merge-movies:create-mobile-video movieId: <movie-id>` - Convert a specific movie to mobile

## MCP Tools

This skill uses the `merge-movies` MCP server. All tools are available automatically. Authentication is handled by the MCP transport.

If the key is missing, tell the user to create one at https://merge.mov/settings.

## Making Effective Mobile Videos

Mobile videos are consumed differently from landscape videos. Viewers scroll through feeds quickly, watch on small screens, and expect fast-paced, visually engaging content. A straight 1:1 conversion of a landscape video won't produce a great mobile video — you need to rethink the content for the format.

### Key Principles

1. **Shorter is better.** Mobile viewers have less patience. If the landscape movie is 60s, aim for 30-45s in portrait. Cut filler scenes, merge related scenes, and tighten narration.

2. **Front-load the hook.** The first 2-3 seconds determine whether someone keeps watching. Start with the most interesting scene — the key insight, the dramatic before/after, or a compelling question. Don't open with a title card.

3. **One idea per scene.** Mobile screens are small. Each scene should communicate one clear thing. If a landscape scene showed two files side-by-side, split it into two sequential scenes or stack them.

4. **Bigger text, fewer words.** Landscape videos can fit paragraphs on screen. Portrait videos should use large, scannable text. For React/slide scenes, cut body text to the essentials and increase font sizes aggressively.

5. **Vertical flow is natural.** Portrait videos have a natural top-to-bottom reading flow. Use this — animate elements appearing from top to bottom, stack content vertically, let the eye travel downward.

6. **Keep narration tight.** Rewrite narration to be punchier. Cut filler phrases ("Let's take a look at", "As you can see"). Use active, direct language. Each word should earn its place.

### What to Cut

When converting a landscape movie, don't blindly convert every scene. Consider:

- **Title/intro slides**: Replace with a single punchy opening or skip entirely. Mobile viewers don't need ceremony.
- **Context-heavy code scenes**: If a scene shows 60 lines of unchanged code for context, trim to just the relevant section. Show 10-15 lines max per code scene on mobile.
- **Redundant scenes**: Two scenes making the same point? Pick the stronger one.
- **Long terminal output**: Trim verbose command output. Show the command and the key result line, not the full log.

### What to Emphasize

- **The "wow" moment**: Every good video has a payoff — make sure it lands clearly in portrait.
- **Visual scenes**: React views with animations, diagrams, and reveals work great on mobile. Invest time making these look polished at portrait dimensions.
- **Before/after contrasts**: Quick cuts between problem and solution are effective on mobile.
- **Code highlights**: Use focus highlights (`focus: true`) liberally — they zoom in and blur the rest, which works perfectly on small screens.

### Narration Adaptation

Don't just copy narration verbatim. Adapt it:

- **Shorten**: "We're going to add JWT-based authentication with refresh token support to protect our API routes" → "Adding JWT auth with refresh tokens to protect our API."
- **Be direct**: "Let's take a look at what we need to change" → Cut entirely, just show the code.
- **Front-load the point**: "After looking at the codebase, we decided to add validation here because..." → "This validation prevents invalid emails from reaching the database."

## Workflow

### Step 1: Fetch the Source Movie

```
get_movie({ movieId: "<movie-id>" })
```

Parse `$ARGUMENTS` to extract the movieId. If not provided, use `list_movies` and ask the user which movie to convert.

### Step 2: Plan the Portrait Version

Before creating anything, review the source movie's scenes and plan the conversion:

1. **Identify scenes to keep, cut, or merge.** Not every landscape scene needs a portrait equivalent.
2. **Identify the hook scene** — which scene is most visually interesting or communicates the key idea? Move it to the front.
3. **Flag React views** that need full rewrites vs scenes that can transfer directly.
4. **Estimate total duration** — aim for 60-70% of the landscape duration as a starting point.

### Step 3: Create the Portrait Movie

Create a new movie with the same metadata but `orientation: "portrait"`:

```
create_movie({
  movie: {
    metadata: {
      title: "Mobile: <original title>",
      description: "<original description>",
      repository: "<original if present>",
      branch: "<original if present>"
    },
    orientation: "portrait",
    scenes: []
  }
})
```

### Step 4: Convert Each Scene

Process each scene from the source movie and create portrait-adapted versions. The key adaptations:

#### Code View Scenes

- **Side-by-side layout**: Convert to `stacked` layout (the renderer does this automatically, but set it explicitly for clarity)
- **Single layout**: Keep as-is, the renderer adapts to portrait width automatically
- **Inline-diff**: Keep as-is
- **Stacked**: Keep as-is
- Code blocks, animations, highlights all transfer directly
- Font sizes and scroll speeds work well at portrait width
- **Trim long code blocks** — show 10-15 lines max. Use lineRanges to zoom into the important section rather than showing the full file.
- **Use focus highlights** — `focus: true` on highlights zooms in and blurs the rest, ideal for mobile viewers
- **Slow down scroll speed** — portrait width means more line-wrapping, so reduce `linesPerSecond` by ~30% from the landscape version

```
create_scene({
  movieId: "<new-movie-id>",
  scene: {
    title: "<original title>",
    narration: "<shortened narration>",
    view: {
      type: "code",
      layout: "stacked",  // was "side-by-side"
      codeBlocks: [...],  // copy from source, trim lineRanges
      animations: {...}   // copy from source, consider adding focus highlights
    }
  }
})
```

#### Terminal View Scenes

- Terminal views work well in portrait — the narrower width is still comfortable for command display
- Copy all properties directly (entries, theme, animations)
- **Trim long output** — cut verbose output to just the key result lines
- **Reduce typing speed** slightly if commands are long — they wrap more in portrait width

#### Slide View Scenes

- Slide positions use percentages (0-100), so they adapt automatically
- **Adjust positions** for portrait layout:
  - Move title elements higher (y: 15-25 instead of 30-40)
  - Increase font sizes for mobile readability (title: 72-80px, body: 36-40px)
  - Stack elements more vertically since there's more vertical space
  - Use full width — don't leave half the screen empty
- **Consider replacing with React views** — slides are static and can feel flat on mobile. If the slide has bullet points or a diagram, a React view with staggered animations will be more engaging.

#### React View Scenes — REWRITE REQUIRED

**Before writing React view code from scratch**, search the community template library for a matching starting point:
1. Call `list_react_templates` with a relevant query or tags (e.g. `query: "diagram"`, `tags: ["portrait"]`)
2. If a template fits, call `get_react_template` to get the full code
3. Customize the template code for your scene's specific content

**Use literal Unicode characters, not escape sequences** — The `code` field is a string, so `\u2713` renders as the literal text `\u2713`, not a checkmark. Always paste the actual character (e.g. `✓`, `→`, `↓`, `•`) directly into the code string.

React views contain hardcoded layout code that assumes 1920x1080. **You must rewrite the React code** for portrait (1080x1920):

- The `useVideoConfig()` hook will return `{ width: 1080, height: 1920, fps: 30 }` — use this for responsive layouts
- Swap horizontal layouts to vertical
- Adjust absolute pixel positions (e.g., boxes at x: 600 should be repositioned for 1080px width)
- Increase font sizes for mobile readability (titles: 72-80px, body: 36-40px, bullets: 32-36px)
- Use more vertical spacing since there's more height available
- Keep animation logic (springs, interpolations) the same — just adjust positions and sizes
- **Speed up animations** — mobile viewers expect faster pacing. Reduce delays between staggered items by ~30%.
- **Fill the screen** — Portrait has 1920px of vertical space. Spread content across the full height instead of clustering everything in the center. Use larger gaps (40-60px between sections), bigger padding on cards/containers, and wider elements (use 80-90% of the 1080px width). Avoid leaving large empty bands at top and bottom.

**Example: Landscape bullet list → Portrait**

Landscape (1920x1080):
```javascript
// padding: '96px 120px', fontSize: 64 title, 28 bullets, stagger delay 15 frames
```

Portrait (1080x1920):
```javascript
// padding: '120px 60px', fontSize: 72 title, 34 bullets, stagger delay 10 frames
// more vertical spacing between items (marginBottom: 32 vs 20)
```

**Example: Landscape architecture diagram → Portrait**

Landscape had boxes at x: 160, 600, 1100 with horizontal arrows between them.
Portrait should stack boxes vertically at y: 400, 800, 1200 with downward arrows (↓ instead of →).
Use the full 1080px width for each box — make them prominent, not cramped.

#### Video View Scenes

- Set `videoFit: "contain"` to prevent cropping in portrait
- Copy all other properties directly
- **Consider whether the video scene adds value on mobile** — a small landscape video letterboxed inside a portrait frame may not be worth keeping. If the video content isn't essential, cut it.

### Step 5: Transfer Transitions

Copy startTransition and endTransition from each source scene. Consider:
- **Faster transitions** — use 0.3s instead of 0.5s for fades. Mobile pacing should feel snappy.
- **More cuts, fewer fades** — cut transitions (`"type": "cut"`) feel more energetic and are common in short-form video.

### Step 6: Return the Studio URL

```
{MERGE_MOVIES_URL}{studioUrl}
```

## Portrait Design Guidelines

### Canvas: 1080x1920 at 30fps

- **More vertical space**: Use the full height for stacking content
- **Narrower width**: 1080px vs 1920px — avoid cramming too much horizontally
- **Mobile-first text sizes**: Larger than landscape since screens are physically smaller
- **Padding**: Use 60-80px horizontal padding (vs 96-120px in landscape)
- **Safe zones**: Keep important content away from the top 120px and bottom 200px — mobile UIs overlay controls there

### Color Palette (same as landscape)

- Background: `#0d1117`
- Title text: `#ffffff`
- Body text: `#e6edf3`
- Accent: `#58a6ff`

### Typography for Mobile

| Element | Landscape | Portrait | Notes |
|---------|-----------|----------|-------|
| Title | 64px | 72-80px | Bold, white, one line if possible |
| Body text | 32px | 36-40px | Keep to 2-3 lines max |
| Bullet items | 28px | 32-36px | Bigger spacing between items |
| Code (in React views) | 18-20px | 22-24px | Monospace, generous line height |
| Labels/annotations | 20-24px | 28-32px | Must be readable at phone size |

### Tips

- Think "vertical" — stack elements instead of placing them side by side
- **Fill the vertical space** — Don't just center a small cluster of content. Use the full 1920px height with generous gaps, larger elements, and wider containers. Portrait screens should feel full, not empty.
- **Use the full width** — Elements should span 80-90% of the 1080px width. Cards, buttons, and containers should feel wide and prominent, not narrow and floating.
- **Size up everything** — Titles: 72-120px. Body: 36-42px. Icons/emoji: 48-56px. Gaps: 40-60px. Padding on cards: 20-32px. These are physically smaller screens — be generous.
- Code views automatically adapt their font rendering for the narrower width
- Terminal views work great in portrait with no changes needed
- React views are the main effort — they need manual layout adaptation
- Keep narration text shorter than landscape — tighten every sentence
- Scene durations can often be shorter than landscape — mobile pacing is faster
- Use `focus` highlights on code to draw the eye to what matters
- Prefer bold visual moments over dense information
- Test the feel: if a scene doesn't earn its screen time on mobile, cut it
- **Use literal Unicode** — paste actual characters (`→`, `↓`, `✓`, `•`), never escape sequences like `\u2192`
