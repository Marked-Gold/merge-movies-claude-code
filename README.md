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

### Set Your API Key

1. Sign up at [merge.mov](https://merge.mov)
2. Go to [Settings](https://merge.mov/settings) and create an API key
3. Set the environment variable:

```bash
export MERGE_MOVIES_API_KEY="mm_your_key_here"
```

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

This plugin is a pure skill — no MCP server, no dependencies to install. The skills teach Claude to call the [merge.mov REST API](https://merge.mov) directly using `curl`, creating movies with code views, slide views, terminal demos, and custom React animations.

## License

MIT
