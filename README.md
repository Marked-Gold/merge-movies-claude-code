# Merge Movies — Claude Code Plugin

Generate code walkthrough movies from git diffs using [merge.mov](https://merge.mov).

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
2. Go to [Settings](https://studio.merge.mov/settings) and create an API key
3. Set the environment variable:

```bash
export MERGE_MOVIES_API_KEY="mm_your_key_here"
```

### Generate a Movie

```
/merge-movies:movie HEAD~3..HEAD
```

## Usage

| Command | Description |
|---------|-------------|
| `/merge-movies:movie` | Interactive — prompts for what to include |
| `/merge-movies:movie HEAD~5..HEAD` | From a commit range |
| `/merge-movies:movie uncommitted` | From uncommitted changes |
| `/merge-movies:movie feature-branch` | From branch changes vs main |

The skill will:
1. Parse the git diff for the specified range
2. Plan logical scenes from the changes
3. Generate narration for each scene
4. Create the movie via the merge.mov API

View your movie at `https://studio.merge.mov/movie/{id}`.

## How It Works

This plugin is a pure skill — no MCP server, no dependencies to install. The skill teaches Claude to call the [merge.mov REST API](https://merge.mov) directly using `curl`, creating movies with code views, slide views, terminal demos, and custom React animations.

## License

MIT
