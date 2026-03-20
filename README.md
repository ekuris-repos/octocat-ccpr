# OctoCat CCPR

Central Code Prompt Repository (CCPR) for the [OctoCAT Supply Chain Management](https://github.com/msft-common-demos/octocat_supply-ubiquitous-octo-bassoon) application.

## Overview

This repository is a **Tier 2 CCPR implementation** that provides structured, versioned AI prompt management for the OctoCAT Supply project. It follows the [CCPR framework](https://github.com/ekuris-repos/ccpr_root) standards for prompt governance, compliance, and reuse.

## Repository Structure

```
octocat-ccpr/
  README.md                              # This file
  prompts/
    generation/
      octocat_supply_development.md      # Full-stack development prompt (Tier 2)
    classification/
    extraction/
    summarization/
    transformation/
  templates/
    prompt_template.md                   # Tier 2 prompt template
  docs/
    architecture-reference.md            # OctoCAT Supply architecture summary
```

## CCPR Tier 2 Features

- **Enhanced Metadata**: YAML front matter with category, tags, confidence scores, and dependency tracking
- **Variables**: Parameterized prompts with documented variable tables
- **Guidelines**: Best practices, quality criteria, common pitfalls, and performance tips
- **Related Prompts**: Cross-references between prompts in the library
- **Compliance Notes**: Security and data handling considerations

## Getting Started

1. Browse the `prompts/` directory for available prompts organized by category
2. Each prompt includes metadata, usage examples, and guidelines
3. Use the `templates/prompt_template.md` when creating new prompts
4. Follow the Tier 2 template structure for all contributions

## Prompt Lifecycle

| Status | Description |
|--------|-------------|
| `draft` | Initial creation, not yet reviewed |
| `review` | Submitted for peer review |
| `approved` | Reviewed and approved for use |
| `deprecated` | No longer recommended; replacement noted |

## Related Repositories

- [OctoCAT Supply Application](https://github.com/msft-common-demos/octocat_supply-ubiquitous-octo-bassoon) - The target application
- [CCPR Root](https://github.com/ekuris-repos/ccpr_root) - The CCPR framework and standards

## License

MIT
