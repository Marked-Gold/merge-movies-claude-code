# Merge Movies — Claude Code Plugin

Create and update code walkthrough movies using [merge.mov](https://merge.mov).

## Quick Start

### Install the Plugin

```bash
# From marketplace
claude plugin marketplace add Marked-Gold/merge-movies-claude-code
claude plugin install merge-movies@merge-movies-marketplace

# Or local installation
claude --plugin-dir ./merge-movies-plugin
```

### Authenticate

1. Sign up at [merge.mov](https://merge.mov)
2. Run `/mcp` in Claude Code — a browser window will open for you to log in
3. Authentication is handled automatically via OAuth — no API keys or environment variables needed

### Create a Movie

```
/merge-movies:create-movie HEAD~3..HEAD
```

### Update an Existing Movie

```
/merge-movies:update-movie my-movie-title
```

## Skills

### `create-movie` — Create a new movie

| Command | Description |
|---------|-------------|
| `/merge-movies:create-movie HEAD~5..HEAD` | From a commit range |
| `/merge-movies:create-movie uncommitted` | From uncommitted changes |
| `/merge-movies:create-movie feature-branch` | From branch changes vs main |
| `/merge-movies:create-movie walkthrough auth` | Feature walkthrough |
| `/merge-movies:create-movie architecture` | Architecture overview |
| `/merge-movies:create-movie setup` | Setup / getting started guide |
| `/merge-movies:create-movie` | Interactive / free-form |

### `update-movie` — Modify an existing movie

| Command | Description |
|---------|-------------|
| `/merge-movies:update-movie <movie-id>` | Update by movie ID |
| `/merge-movies:update-movie <search term>` | Find by title and update |
| `/merge-movies:update-movie` | List movies and pick one |

The update skill supports adding, editing, reordering, and removing scenes, as well as updating metadata and code blocks.

## How It Works

This plugin connects to [merge.mov](https://merge.mov) via MCP (Model Context Protocol). Authentication is handled automatically via OAuth 2.1 — on first use, a browser window opens for you to log in, and tokens are managed by Claude Code.

## License

MIT
