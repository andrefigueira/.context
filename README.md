# Substrate Methodology Template

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Stars](https://img.shields.io/github/stars/yourusername/substrate-template?style=social)](https://github.com/yourusername/substrate-template/stargazers)
[![Documentation](https://img.shields.io/badge/docs-substrate-blue.svg)](./substrate/)

A complete "Documentation as Code as Context" template implementing the Substrate Methodology. Transform any software project into a self-documenting, AI-optimized codebase with modular, Git-native documentation that serves as a living knowledge base.

## What is the Substrate Methodology?

The Substrate Methodology solves the critical problem of outdated documentation and AI hallucinations by creating a structured, domain-organized documentation system in a `.context/` directory. This approach:

- **Eliminates documentation drift** through Git-native versioning
- **Reduces AI hallucinations by 50%+** with structured context
- **Accelerates onboarding** with comprehensive domain knowledge
- **Captures decision history** for future reference
- **Scales development teams** with consistent patterns

## Quick Start

1. **Clone and customize:**
   ```bash
   git clone https://github.com/yourusername/substrate-template.git your-project
   cd your-project
   rm -rf .git && git init
   ```

2. **Customize the template:**
   - Replace `[Your Project Name]` placeholders
   - Update code examples for your stack
   - Modify domains in `.context/` to match your architecture

3. **Start documenting:**
   ```bash
   # Edit the entry point
   vim .context/substrate.md
   
   # Add your first domain documentation
   vim .context/architecture/overview.md
   ```

4. **Use with AI:**
   ```bash
   # Example: Generate auth middleware
   cat .context/auth/overview.md .context/auth/security.md | your-ai-tool
   
   # Review AI agent guidelines
   cat agents.md .context/substrate.md
   ```

## Why This Matters in 2025

Modern software development faces a documentation crisis:
- 73% of developers work with outdated docs
- AI tools hallucinate when missing context
- Knowledge silos slow team velocity
- Onboarding takes weeks instead of days

The Substrate Methodology transforms documentation from a burden into a force multiplier, creating a comprehensive knowledge base that grows with your codebase.

## Structure Overview

```
README.md                     # Project introduction and quick start
agents.md                     # AI agent usage patterns and guidelines
.context/
├── substrate.md              # Entry point and methodology guide
├── architecture/             # System design and patterns
│   ├── overview.md          # System architecture with diagrams
│   ├── dependencies.md      # Dependency injection patterns
│   └── patterns.md          # Code organization and error handling
├── auth/                     # Authentication and security
│   ├── overview.md          # JWT authentication flows
│   ├── integration.md       # HTTP middleware integration
│   └── security.md          # Security model and threat mitigation
├── api/                      # API reference and examples
│   ├── endpoints.md         # REST API documentation
│   ├── headers.md           # HTTP headers and CORS
│   └── examples.md          # Client implementation examples
├── database/                 # Data models and migrations
│   ├── schema.md            # PostgreSQL schema design
│   ├── models.md            # Data models and validation
│   └── migrations.md        # Migration strategy and tooling
└── guidelines.md             # Development workflows and standards
```

Each domain contains modular Markdown files optimized for:
- **Human readability** with clear structure
- **AI consumption** with parseable formats
- **Version control** with Git integration
- **Extensibility** for project-specific needs

## Features

- ✅ **Modular Documentation**: Domain-organized for precise context
- ✅ **AI-Optimized Format**: Structured for LLM consumption
- ✅ **Decision History**: Captures rationale and trade-offs
- ✅ **Code Examples**: Real patterns for immediate use
- ✅ **Mermaid Diagrams**: Visual architecture documentation
- ✅ **Generic Template**: Adaptable to any tech stack
- ✅ **MIT Licensed**: Free for commercial use

## Usage Patterns

### For Development Teams
```bash
# Before implementing auth
cat .context/auth/*.md > context.txt
# Use context.txt with your preferred AI tool for implementation guidance
```

### For Onboarding
```bash
# New developer orientation
cat .context/substrate.md .context/architecture/overview.md
```

### For AI-Assisted Development
```bash
# Context-aware code generation
echo "Based on the following documentation, implement user registration:" && \
cat .context/auth/integration.md .context/database/models.md
```

## Contributing

We welcome contributions to improve the Substrate Methodology template:

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-domain`)
3. **Document** your changes in the relevant `.context/` files
4. **Submit** a pull request

### Contribution Guidelines

- Follow the established documentation structure
- Include decision rationale for changes
- Add code examples for new patterns
- Update the main README if adding new domains
- Ensure Markdown follows consistent formatting

## Community

- 🐛 **Report issues**: [GitHub Issues](https://github.com/yourusername/substrate-template/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/yourusername/substrate-template/discussions)
- 📖 **Documentation**: [Substrate Methodology Guide](./.context/substrate.md)

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Adapting to Your Tech Stack

The template uses Go code examples but is designed to be adapted to any technology:

### Popular Adaptations
- **JavaScript/TypeScript**: Replace Go structs with TypeScript interfaces
- **Python**: Convert Go models to Pydantic models or dataclasses
- **Java**: Transform to Spring Boot patterns with JPA entities
- **C#**: Adapt to ASP.NET Core with Entity Framework
- **Rust**: Convert to Rust structs with Serde serialization
- **PHP**: Adapt to Laravel models and Eloquent patterns

### Framework Examples
The authentication patterns can be adapted to:
- **Express.js**: JWT middleware and Passport.js integration
- **Django**: Django REST framework with token authentication
- **FastAPI**: OAuth2 with JWT tokens and dependency injection
- **Ruby on Rails**: Devise with JWT tokens
- **Spring Security**: JWT authentication filters

## Inspiration

Based on the ["Documentation as Code as Context" methodology](https://buildingbetter.tech/p/documentation-as-code-as-context) by Building Better. This template transforms that philosophy into a practical, ready-to-use system for any software project.

### Related Projects
- 📖 [Original methodology article](https://buildingbetter.tech/p/documentation-as-code-as-context) - The foundational concept behind this template
- 🤖 [Your AI sucks because you suck at prompting](https://buildingbetter.tech/p/your-ai-sucks-because-you-suck-at) - Why context matters for AI effectiveness
- 🧠 [Information Substrate Convergence](https://github.com/andrefigueira/information-substrate-convergence) - Advanced implementation and research

---

**Ready to transform your documentation?** Start with `.context/substrate.md` and experience the difference of having your codebase document itself.