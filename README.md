# Vibe Coding Templates

Comprehensive project templates optimized for AI agents to bootstrap professional projects with modern tooling, CI/CD, and best practices.

## 🤖 AI Agent Optimized

This repository is designed for AI agents (Claude Code, GitHub Copilot, Cursor, etc.) to quickly bootstrap professional projects with:

- ✅ Modern package management
- ✅ Comprehensive testing setup
- ✅ CI/CD pipelines
- ✅ Code quality enforcement
- ✅ Best practices baked in

## 📁 Repository Structure

```
vibe-coding-templates/
├── CLAUDE.md              # Instructions for AI agents
├── llms.txt               # LLM-friendly project overview
├── python/                # Python project templates
│   ├── BOOTSTRAP.md       # Step-by-step bootstrap guide
│   ├── AI_AGENT_GUIDE.md  # Common scenarios & quick reference
│   ├── EXAMPLE_USAGE.md   # Complete example walkthrough
│   ├── docs/              # Comprehensive documentation
│   │   ├── PACKAGE_MANAGEMENT.md
│   │   ├── LLMS_TXT.md    # llms.txt documentation
│   │   ├── cicd/
│   │   │   ├── GITHUB_ACTIONS.md
│   │   │   └── PRE_COMMIT.md
│   │   └── testing/
│   │       ├── TEST_COVERAGE.md
│   │       ├── CODE_QUALITY.md
│   │       └── UNIT_TESTING.md
│   └── templates/         # Reusable templates
│       ├── Makefile       # Development tasks
│       ├── llms.txt       # AI documentation template
│       ├── CLAUDE.md      # Claude-specific template
│       ├── config/        # pyproject.toml, .pre-commit-config.yaml, .gitignore
│       └── cicd/
│           └── workflows/ # GitHub Actions: ci, release, docs, benchmark
└── README.md              # This file
```

## 🚀 Quick Start

### For AI Agents

When a user requests a new project:

1. **Identify the language** (currently Python supported)
2. **Navigate to language folder** (e.g., `python/`)
3. **Follow BOOTSTRAP.md** systematically
4. **Use templates and docs** as references
5. **Verify everything works** before completion

### For Human Developers

Browse the language-specific folders for:
- Documentation in `{language}/docs/`
- Templates in `{language}/templates/`
- Bootstrap guides in `{language}/BOOTSTRAP.md`

## 📚 Available Templates

### Python Projects

Complete Python project setup with:
- **Package Management**: `uv` (10-100x faster than pip)
- **Testing**: pytest with coverage
- **CI/CD**: GitHub Actions workflows
- **Code Quality**: ruff, mypy, pre-commit hooks
- **Documentation**: Comprehensive guides

📖 **Python Documentation:**
- [Bootstrap Guide](python/BOOTSTRAP.md) - Complete setup walkthrough
- [AI Agent Guide](python/AI_AGENT_GUIDE.md) - Quick reference for AI agents
- [Package Management](python/docs/PACKAGE_MANAGEMENT.md) - Using uv effectively
- [llms.txt Guide](python/docs/LLMS_TXT.md) - AI documentation best practices
- [GitHub Actions](python/docs/cicd/GITHUB_ACTIONS.md) - CI/CD workflows
- [Pre-commit Hooks](python/docs/cicd/PRE_COMMIT.md) - Code quality automation

🛠️ **Python Templates:**
- [Makefile](python/templates/Makefile) - Complete development tasks
- [llms.txt](python/templates/llms.txt) - AI agent documentation
- [CLAUDE.md](python/templates/CLAUDE.md) - Claude-specific instructions
- [pyproject.toml](python/templates/config/pyproject.toml) - Project, ruff, mypy, pytest and coverage config
- [Pre-commit config](python/templates/config/.pre-commit-config.yaml) - SHA-pinned hooks
- [CI/CD Workflows](python/templates/cicd/workflows/) - GitHub Actions

### Future Languages

The structure supports adding:
- `javascript/` - Node.js/TypeScript projects
- `rust/` - Rust projects
- `go/` - Go projects
- `java/` - Java projects

## 💡 Example Usage

```markdown
User: "Create a Python project called data-processor"

AI Agent: I'll bootstrap your project using vibe-coding-templates:
1. Creating project structure...
2. Setting up package management with uv...
3. Configuring testing and CI/CD...
4. Initializing git repository...

✅ Project created successfully!
```

The AI agent follows [python/BOOTSTRAP.md](python/BOOTSTRAP.md) to create a complete project.

## 🎯 Key Features

### For Python Projects

| Feature | Tool/Framework | Documentation | Template |
|---------|---------------|--------------|----------|
| Package Management | uv | [docs](python/docs/PACKAGE_MANAGEMENT.md) | [Makefile](python/templates/Makefile) |
| Testing | pytest | [docs](python/docs/testing/TEST_COVERAGE.md) | - |
| Code Quality | ruff, mypy | [docs](python/docs/testing/CODE_QUALITY.md) | [pyproject.toml](python/templates/config/pyproject.toml) |
| CI/CD | GitHub Actions | [docs](python/docs/cicd/GITHUB_ACTIONS.md) | [workflows](python/templates/cicd/workflows/) |
| Pre-commit | pre-commit | [docs](python/docs/cicd/PRE_COMMIT.md) | [config](python/templates/config/.pre-commit-config.yaml) |
| AI Documentation | llms.txt | [docs](python/docs/LLMS_TXT.md) | [llms.txt](python/templates/llms.txt) |

## 🤝 Contributing

To add templates for a new language:

1. Create a language folder (e.g., `rust/`)
2. Add `BOOTSTRAP.md` with step-by-step guide
3. Add `docs/` with comprehensive documentation
4. Add `templates/` with reusable components
5. Update this README

## 📖 Documentation Philosophy

- **No duplication** - Reference existing docs
- **Language-specific** - Each language has its own guides
- **AI-optimized** - Clear steps for agents to follow
- **Template-based** - Reusable components
- **Verification-focused** - Always test that it works

## 🔧 For AI Agent Developers

If you're building an AI agent that creates projects:

1. Point your agent to this repository
2. Have it read `CLAUDE.md` for instructions
3. Use language-specific `BOOTSTRAP.md` guides
4. Reference `docs/` for detailed information
5. Apply templates from `templates/` directories
6. Include `llms.txt` in generated projects for AI context

### Key Files for AI Agents

- **CLAUDE.md** - Repository-level AI instructions
- **llms.txt** - Standard AI documentation format ([llmstxt.org](https://llmstxt.org))
- **BOOTSTRAP.md** - Step-by-step project creation
- **templates/** - Ready-to-use configuration files

## 📬 Support

- Issues: [GitHub Issues](https://github.com/chrishayuk/vibe-coding-templates/issues)
- Discussions: [GitHub Discussions](https://github.com/chrishayuk/vibe-coding-templates/discussions)

## 📄 License

MIT License - Use these templates freely in your projects.

---

Made with ❤️ for AI-assisted development