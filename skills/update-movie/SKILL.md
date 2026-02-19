# Update Movie Skill

Modify an existing merge.mov movie — add, edit, reorder, or remove scenes.

$ARGUMENTS

## Overview

This skill helps you update existing movies by:
1. Finding the target movie by ID or title search
2. Fetching its current state (scenes, metadata)
3. Making surgical updates via the REST API
4. Returning the studio URL for review

## Usage

- `/merge-movies:update-movie <movie-id>` - Update a specific movie by ID
- `/merge-movies:update-movie <search term>` - Find a movie by title and update it
- `/merge-movies:update-movie` - List recent movies and pick one

## API Authentication

All API calls require the `MERGE_MOVIES_API_KEY` environment variable. Include it as a header on every request:

```bash
curl -s -X GET "https://merge.mov/api/movies" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

If the key is missing, tell the user to create one at https://studio.merge.mov/settings.

## Workflow

### Step 1: Find the Target Movie

**If arg looks like a UUID** (contains dashes, 32+ hex chars):
```bash
curl -s -X GET "https://merge.mov/api/movies/$MOVIE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

**If arg is text** — list movies and match by title:
```bash
MOVIES=$(curl -s -X GET "https://merge.mov/api/movies" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY")
echo "$MOVIES" | jq '.[] | { id, title: .metadata.title }'
```

**If no arg** — list movies and ask the user to pick:
```bash
curl -s -X GET "https://merge.mov/api/movies" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" | jq '.[] | { id, title: .metadata.title }'
```

### Step 2: Fetch Current State

Get the full movie with all scenes to understand what exists:

```bash
MOVIE=$(curl -s -X GET "https://merge.mov/api/movies/$MOVIE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY")
echo "$MOVIE" | jq '.scenes[] | { id, title, narration: .narration[:60], viewType: .view.type }'
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

Use the appropriate endpoint for each change:

#### Update Movie Metadata

```bash
curl -s -X PUT "https://merge.mov/api/movies/$MOVIE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ full movie object with updated metadata }'
```

#### Add a Scene

```bash
curl -s -X POST "https://merge.mov/api/movies/$MOVIE_ID/scenes" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{
    "narration": "New scene narration.",
    "view": { "type": "slide", "elements": [...] }
  }'
```

After adding, use reorder to place it in the correct position.

#### Update a Scene (Partial)

```bash
curl -s -X PATCH "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{
    "narration": "Updated narration text."
  }'
```

PATCH only updates the fields you include — everything else stays unchanged.

#### Replace a Scene (Full)

```bash
curl -s -X PUT "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{
    "narration": "Completely new scene.",
    "view": { "type": "code", "layout": "single", "codeBlocks": [...] }
  }'
```

#### Delete a Scene

```bash
curl -s -X DELETE "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

#### Reorder Scenes

```bash
curl -s -X POST "https://merge.mov/api/movies/$MOVIE_ID/scenes/reorder" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "sceneIds": ["scene-3", "scene-1", "scene-2"] }'
```

#### Manage Code Blocks

```bash
# Add a code block to a scene
curl -s -X POST "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID/codeblocks" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{
    "filePath": "src/new-file.ts",
    "lineRanges": [{ "start": 1, "end": 20 }],
    "changeType": "add",
    "content": "// file content..."
  }'

# Update a code block
curl -s -X PUT "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID/codeblocks/$BLOCK_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{
    "filePath": "src/file.ts",
    "lineRanges": [{ "start": 10, "end": 30 }],
    "changeType": "modify",
    "content": "// updated content..."
  }'

# Delete a code block
curl -s -X DELETE "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID/codeblocks/$BLOCK_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

### Step 5: Return the Studio URL

```
https://studio.merge.mov/movie/{MOVIE_ID}
```

## Common Update Patterns

### Rewrite narration for all scenes

Fetch all scenes, iterate through them, PATCH each with updated narration.

### Insert a scene at a specific position

1. POST the new scene (it gets appended)
2. GET all scene IDs in current order
3. Splice the new scene ID into the desired position
4. POST reorder with the new order

### Swap a code scene for a slide

PUT the scene with a completely new view object — the old view is fully replaced.

### Add highlights to an existing code scene

PATCH the scene, including the full `view.animations` object with the new highlights added.

### Update code to reflect new file content

1. Read the updated source file
2. PUT the code block with new `content` and `lineRanges`
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
  "entries": [
    { "id": "e1", "command": "npm test", "output": "Tests passed", "exitCode": 0 }
  ]
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

## REST API Reference

**Base URL:** `https://merge.mov`
**Auth:** `X-API-Key: $MERGE_MOVIES_API_KEY` header on all requests
**Content-Type:** `application/json`
**Studio URL:** `https://studio.merge.mov/movie/{movieId}` (for viewing)

### Movies

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/movies` | List all movies |
| POST | `/api/movies` | Create movie — body: `{ id?, movie: { metadata, scenes } }` |
| GET | `/api/movies/:id` | Get movie with all scenes |
| PUT | `/api/movies/:id` | Update full movie — body: full movie object |
| DELETE | `/api/movies/:id` | Delete movie |

**Movie metadata fields:** `title` (required), `description` (required), `repository`, `branch`, `commitRange: { from?, to? }`

### Scenes

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/movies/:movieId/scenes` | List scenes |
| POST | `/api/movies/:movieId/scenes` | Create scene — body: `{ narration, view, ... }` |
| GET | `/api/movies/:movieId/scenes/:sceneId` | Get scene |
| PATCH | `/api/movies/:movieId/scenes/:sceneId` | Partial update — body: any scene fields |
| PUT | `/api/movies/:movieId/scenes/:sceneId` | Full replace — body: complete scene |
| DELETE | `/api/movies/:movieId/scenes/:sceneId` | Delete scene |
| POST | `/api/movies/:movieId/scenes/reorder` | Reorder — body: `{ sceneIds: [...] }` |

**Create scene fields:** `narration` (required), `view` (required), `id`, `title`, `timestamp`, `duration`, `startTransition`, `endTransition`

### Code Blocks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/movies/:movieId/scenes/:sceneId/codeblocks` | List code blocks |
| POST | `/api/movies/:movieId/scenes/:sceneId/codeblocks` | Create — body: `{ filePath, lineRanges, changeType, ... }` |
| GET | `/api/movies/:movieId/scenes/:sceneId/codeblocks/:blockId` | Get code block |
| PUT | `/api/movies/:movieId/scenes/:sceneId/codeblocks/:blockId` | Update code block |
| DELETE | `/api/movies/:movieId/scenes/:sceneId/codeblocks/:blockId` | Delete code block |

**Create code block fields:** `filePath` (required), `lineRanges` (required), `changeType` (required), `id`, `name`, `parentId`, `content`, `lineOrder`
