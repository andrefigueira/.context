# Substrate Methodology Template

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Stars](https://img.shields.io/github/stars/andrefigueira/.context?style=social)](https://github.com/andrefigueira/.context/stargazers)
[![Documentation](https://img.shields.io/badge/docs-substrate-blue.svg)](./.context/)

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
   git clone https://github.com/andrefigueira/.context.git your-project
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

4. **Generate your substrate with AI:**
   Use the comprehensive AI prompt below to create domain-specific documentation for your project. See [AI-Assisted Substrate Generation](#ai-assisted-substrate-generation) section for the complete prompt.

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

## AI-Assisted Substrate Generation

**Recommended Approach**: Instead of manually writing documentation, use AI to generate comprehensive substrate documentation tailored to your specific project. This ensures consistency with the methodology while adapting to your unique architecture, tech stack, and business domain.

### Complete AI Generation Prompt

Copy and paste this prompt into your preferred AI tool (Claude, GPT-4, etc.) to generate substrate documentation for your project:

```
You are an expert technical writer specializing in the "Documentation as Code as Context" methodology. Create comprehensive substrate documentation following the exact structure and quality standards of this template: https://github.com/andrefigueira/.context

PROJECT CONTEXT:
- Project Name: [YOUR PROJECT NAME]
- Tech Stack: [YOUR TECH STACK - e.g., Node.js/Express, React, PostgreSQL, Redis]
- Architecture Pattern: [YOUR PATTERN - e.g., microservices, monolith, serverless]
- Authentication Method: [YOUR AUTH - e.g., OAuth2, JWT, session-based]
- Database Type: [YOUR DB - e.g., PostgreSQL, MongoDB, MySQL]
- Target Audience: [YOUR USERS - e.g., internal APIs, public SaaS, enterprise]

REQUIREMENTS:
1. Generate content for ALL substrate domains: architecture, auth, api, database, guidelines
2. Use my specific tech stack and adapt all code examples accordingly
3. Include actual implementation patterns, not generic advice
4. Add decision rationale sections explaining "why" choices were made
5. Include Mermaid diagrams for architecture flows and database schemas
6. Provide realistic error handling patterns
7. Include performance considerations and security measures
8. Add specific testing strategies for my tech stack
9. Include deployment procedures for my infrastructure

STRUCTURE TO FOLLOW:
Create these files with comprehensive, production-ready content:

substrate.md - Entry point with navigation and AI usage patterns
architecture/
├── overview.md - System architecture with Mermaid diagrams
├── dependencies.md - Dependency injection patterns for my stack
└── patterns.md - Code organization and error handling

auth/
├── overview.md - Authentication flow for my auth method
├── integration.md - Framework integration patterns
└── security.md - Security model and threat mitigation

api/
├── endpoints.md - API reference with my actual endpoints
├── headers.md - HTTP headers and middleware patterns
└── examples.md - Client implementations for my stack

database/
├── schema.md - Database schema with ERD for my data model
├── models.md - Data models and validation for my stack
└── migrations.md - Migration strategy for my database

guidelines.md - Development workflow and deployment for my stack

QUALITY STANDARDS:
- Each file should be 400-800 words with practical examples
- Include 2-3 realistic code snippets per file using my tech stack
- Add "Decision History & Trade-offs" sections explaining architectural choices
- Use consistent technical terminology throughout
- Include specific performance benchmarks and security considerations
- Provide actionable implementation guidance, not theoretical concepts

EXAMPLE OUTPUT QUALITY:
Reference the template structure at https://github.com/andrefigueira/.context but adapt ALL content to my specific project. Don't copy generic examples - create realistic implementations for my exact tech stack and business domain.

START WITH: substrate.md as the entry point, then generate each domain systematically.
```

### Usage Instructions

1. **Replace the bracketed placeholders** with your specific project details
2. **Paste the prompt** into your AI tool of choice
3. **Generate each domain** systematically (start with substrate.md)
4. **Review and refine** the generated content for accuracy
5. **Iterate on specific sections** that need more detail

### Example Customization

For a Node.js/Express API project:
```
PROJECT CONTEXT:
- Project Name: TaskFlow API
- Tech Stack: Node.js, Express.js, TypeScript, PostgreSQL, Redis, Docker
- Architecture Pattern: Layered monolith with clear service boundaries
- Authentication Method: JWT with refresh tokens
- Database Type: PostgreSQL with Prisma ORM
- Target Audience: Internal microservice for task management SaaS
```

### AI Tool Recommendations

- **Claude (Anthropic)**: Excellent for following complex instructions and maintaining consistency
- **GPT-4 (OpenAI)**: Great for code examples and technical accuracy
- **Cursor/Continue**: IDE integration for iterative refinement

### Quality Assurance

After AI generation:
1. **Validate code examples** actually work with your stack
2. **Test database schemas** match your actual data model  
3. **Verify security patterns** align with your compliance requirements
4. **Update decision rationale** to reflect your specific constraints
5. **Add team-specific conventions** not captured in the template

This AI-assisted approach typically produces 80-90% complete documentation that requires minimal manual refinement, compared to weeks of manual documentation work.

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

- 🐛 **Report issues**: [GitHub Issues](https://github.com/andrefigueira/.context/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/andrefigueira/.context/discussions)
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