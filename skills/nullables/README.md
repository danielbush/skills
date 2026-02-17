# Nullables Skill for Claude Code

A Claude Code skill that teaches James Shore's Nullables pattern and A-Frame architecture for building testable code without mocks.

## What's Inside

The **nullables** skill provides guidance on:

- **A-Frame Architecture**: Separation of Logic, Infrastructure, and Application layers
- **Nullables Pattern**: Production code with built-in test modes via `.create()` and `.createNull()` factory methods
- **State-Based Testing**: Writing narrow, sociable tests that verify outputs instead of interactions
- **Infrastructure Wrappers**: Building testable wrappers around third-party APIs with embedded stubs
- **Output Tracking**: Verifying what infrastructure sends externally without mocks

## Prerequisites

This project uses [Task](https://taskfile.dev/) to manage skill installation.

```bash
# macOS
brew install go-task

# Linux
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin

# Windows
choco install go-task

# Or via npm
npm install -g @go-task/cli
```

## Installing the Skill

Skills are installed as directories in `~/.claude/skills/`. The skill directory must contain a `SKILL.md` file with YAML frontmatter defining `name` and `description`.

### Using Task

```bash
# Install the skill (copies to ~/.claude/skills/)
task install

# Or symlink for development (changes reflect immediately)
task install:dev

# Remove the skill
task uninstall

# Validate skill structure
task validate

# List all files in the skill
task list
```

### Manual Installation

```bash
# Create the skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy the skill directory
cp -r skills/nullables ~/.claude/skills/

# Or create a symlink for development
ln -s "$(pwd)/skills/nullables" ~/.claude/skills/nullables
```

## Using the Skill

### How Skills Work

Claude Code skills are **automatically invoked** based on task context. Claude reads all skill descriptions and determines when a skill is relevant to the current conversation. You don't need to explicitly invoke the skill - just ask questions or request tasks related to the pattern:

```
"How do I implement the nullables pattern for an HTTP client?"
"Show me how to write tests using createNull"
"Help me refactor this class to use A-Frame architecture"
"Add output tracking to this infrastructure wrapper"
```

Claude will automatically load the skill's guidance when it recognizes these topics.

### In Claude Code CLI

After installing the skill, just run Claude Code normally:

```bash
claude
```

Then ask questions about nullables, testing without mocks, or A-Frame architecture.

### In VS Code

The official [Claude Code VS Code extension](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code) runs Claude Code within VS Code. Skills installed in `~/.claude/skills/` are available automatically.

1. Install the Claude Code extension from the VS Code marketplace
2. Ensure the skill is installed in `~/.claude/skills/nullables/`
3. Open the Claude Code sidebar and ask questions - skills are loaded automatically based on context

### In Zed

Zed integrates with Claude Code via the [Agent Client Protocol (ACP)](https://zed.dev/blog/claude-code-via-acp). When running Claude Code through Zed, skills installed in `~/.claude/skills/` are available.

1. Ensure you're running the latest version of Zed
2. Install the skill to `~/.claude/skills/nullables/`
3. Open the Agent Panel and select Claude Code
4. Ask questions - skills are loaded automatically based on context

## Skill Structure

```
skills/nullables/
├── SKILL.md                        # Main skill with metadata and quick-start guide
└── references/
    └── nullables-guide.md          # Comprehensive implementation reference
```

Skills use progressive disclosure:
1. **Metadata** (name + description) - Always loaded, used to determine relevance
2. **SKILL.md body** - Loaded when skill triggers
3. **References** - Loaded as needed for detailed information

## Why This Approach?

Traditional testing with mocks leads to:
- Brittle tests that break on refactoring
- Tests that pass but production code fails
- Complex mock setup and verification

The nullables pattern enables:
- **Fast tests**: No external dependencies in unit tests
- **Reliable tests**: Tests verify real behavior, not mocked interactions
- **Simple tests**: No mock frameworks, just call methods and verify results
- **Maintainable tests**: Tests survive refactoring because they test behavior

## Learn More

- [Testing Without Mocks - James Shore](https://www.jamesshore.com/v2/projects/testing-without-mocks)

## Contributing

To improve this skill:

1. Edit files in `skills/nullables/`
2. Test with `task install:dev` to symlink to `~/.claude/skills/`
3. Submit improvements
