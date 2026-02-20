# Update Movie Skill

Modify an existing merge.mov movie — add, edit, reorder, or remove scenes.

$ARGUMENTS

## Overview

This skill helps you update existing movies by:
1. Finding the target movie by ID or title search
2. Fetching its current state (scenes, metadata)
3. Making surgical updates via MCP tools
4. Returning the studio URL for review

## Usage

- `/merge-movies:update-movie <movie-id>` - Update a specific movie by ID
- `/merge-movies:update-movie <search term>` - Find a movie by title and update it
- `/merge-movies:update-movie` - List recent movies and pick one

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

### Step 1: Find the Target Movie

**If arg looks like a UUID** (contains dashes, 32+ hex chars):

```
get_movie({ movieId: "<movie-id>" })
```

**If arg is text** — list movies and match by title:

```
list_movies({})
→ Returns array of { id, title, updatedAt }
```

**If no arg** — list movies and ask the user to pick.

### Step 2: Fetch Current State

Get the full movie with all scenes to understand what exists:

```
get_movie({ movieId: "<movie-id>" })
→ Returns full movie with metadata, scenes, and code blocks
```

Present a summary of existing scenes to the user: scene order, types, and narration snippets.

### Step 3: Understand the Request

Ask the user what they want changed, or infer from conversation context. Common operations:

- **Rewrite narration** for a scene
- **Add new scenes** at a specific position
- **Remove scenes** that aren't needed
- **Reorder scenes** to improve flow
- **Update code blocks** (change file content, line ranges, highlights)
- **Change view type** (e.g., swap a code scene for a slide)
- **Update metadata** (title, description)

### Step 4: Make Surgical Updates

#### Update Movie Metadata

Use `update_movie` with the full movie object including updated metadata:

```
update_movie({ movieId: "<movie-id>", movie: { ...fullMovieWithUpdatedMetadata } })
```

#### Add a Scene

Always include a `title` — a short label (2-5 words) for the scene:

```
create_scene({
  movieId: "<movie-id>",
  scene: {
    title: "New Feature Overview",
    narration: "New scene narration.",
    view: { type: "slide", elements: [...] }
  }
})
```

After adding, use reorder to place it in the correct position.

#### Update a Scene (Partial)

```
patch_scene({
  movieId: "<movie-id>",
  sceneId: "<scene-id>",
  scene: { narration: "Updated narration text." }
})
```

`patch_scene` only updates the fields you include — everything else stays unchanged.

#### Replace a Scene (Full)

```
update_scene({
  movieId: "<movie-id>",
  sceneId: "<scene-id>",
  scene: {
    narration: "Completely new scene.",
    view: { type: "code", layout: "single", codeBlocks: [...] },
    timestamp: 0
  }
})
```

#### Delete a Scene

```
delete_scene({ movieId: "<movie-id>", sceneId: "<scene-id>" })
```

#### Reorder Scenes

```
reorder_scenes({
  movieId: "<movie-id>",
  sceneIds: ["scene-3", "scene-1", "scene-2"]
})
```

#### Manage Code Blocks

```
// Add a code block
create_codeblock({
  movieId: "<movie-id>",
  sceneId: "<scene-id>",
  block: {
    filePath: "src/new-file.ts",
    lineRanges: [{ start: 1, end: 20 }],
    changeType: "add",
    content: "// file content..."
  }
})

// Update a code block
update_codeblock({
  movieId: "<movie-id>",
  sceneId: "<scene-id>",
  blockId: "<block-id>",
  block: {
    filePath: "src/file.ts",
    lineRanges: [{ start: 10, end: 30 }],
    changeType: "modify",
    content: "// updated content...",
    parentId: null
  }
})

// Delete a code block
delete_codeblock({
  movieId: "<movie-id>",
  sceneId: "<scene-id>",
  blockId: "<block-id>"
})
```

### Step 5: Return the Studio URL

Use the `studioUrl` returned by the API and prepend the base URL. Use `$MERGE_MOVIES_URL` if set, otherwise default to `https://merge.mov`.

```
{MERGE_MOVIES_URL}{studioUrl}
```

## Common Update Patterns

### Rewrite narration for all scenes

Fetch all scenes with `list_scenes`, iterate through them, `patch_scene` each with updated narration.

### Insert a scene at a specific position

1. `create_scene` to add the new scene (it gets appended)
2. `list_scenes` to get all scene IDs in current order
3. Splice the new scene ID into the desired position
4. `reorder_scenes` with the new order

### Swap a code scene for a slide

Use `update_scene` with a completely new view object — the old view is fully replaced.

### Add highlights to an existing code scene

Use `patch_scene`, including the full `view.animations` object with the new highlights added.

### Update code to reflect new file content

1. Read the updated source file
2. `update_codeblock` with new `content` and `lineRanges`
3. Update any `highlights` that reference changed line numbers

## Scene Type Reference

### Code View
```json
{
  "type": "code",
  "layout": "single | side-by-side | stacked | inline-diff",
  "codeBlocks": [{
    "filePath": "src/file.ts",
    "lineRanges": [{ "start": 1, "end": 30 }],
    "changeType": "modify | add | delete | context",
    "content": "// actual file content"
  }],
  "animations": {
    "scroll": { "id": "s1", "linesPerSecond": 3, "pauses": [{ "lineNumber": 15, "durationMs": 2000 }] },
    "highlights": [{ "id": "h1", "lines": [5, 6, 7], "color": "rgba(255, 213, 79, 0.3)" }]
  }
}
```

### Slide View
```json
{
  "type": "slide",
  "backgroundColor": "#0d1117",
  "elements": [
    { "id": "t", "type": "text", "style": "title | body | bullet", "content": "...", "position": { "x": 10, "y": 30 } },
    { "id": "r", "type": "rect", "position": { "x": 5, "y": 35 }, "size": { "width": 20, "height": 15 }, "stroke": "#58a6ff", "text": "Label" },
    { "id": "c", "type": "circle", "position": { "x": 50, "y": 50 }, "size": { "width": 15 }, "fill": "#58a6ff" },
    { "id": "l", "type": "line", "position": { "x": 25, "y": 42 }, "endPosition": { "x": 35, "y": 42 }, "stroke": "#58a6ff" },
    { "id": "i", "type": "image", "src": "https://...", "position": { "x": 10, "y": 10 }, "size": { "width": 40 } }
  ]
}
```

### Terminal View
```json
{
  "type": "terminal",
  "title": "~/project",
  "theme": "generic | claude-code | codex",
  "inputAnimation": "type | fade | cut",
  "outputAnimation": "type | fade | cut",
  "entries": [{ "id": "e1", "command": "npm test", "output": "Tests passed", "exitCode": 0 }]
}
```

### React View
```json
{
  "type": "react",
  "backgroundColor": "#0d1117",
  "code": "const { useCurrentFrame, spring, interpolate, AbsoluteFill } = scope; ..."
}
```

Scope provides: `React`, `useCurrentFrame`, `useVideoConfig`, `spring`, `interpolate`, `Sequence`, `Series`, `AbsoluteFill`, `Img`

## Transitions

```json
{
  "startTransition": { "type": "fade", "duration": 0.5 },
  "endTransition": { "type": "fade", "duration": 0.3 }
}
```

Types: `cut`, `fade`, `slide`, `zoom`
