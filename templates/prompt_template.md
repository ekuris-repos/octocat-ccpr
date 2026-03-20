# Prompt Template (Tier 2)

Use this template when creating new prompts for the OctoCat CCPR.

---

## YAML Front Matter

```yaml
---
title: "Descriptive Prompt Title"
category: "classification|extraction|generation|summarization|transformation"
subcategory: "specific_domain_or_use_case"
author: "Author Name"
created_date: "YYYY-MM-DD"
last_modified: "YYYY-MM-DD"
version: "X.Y.Z"
status: "draft|review|approved|deprecated"
compliance_frameworks: []
tags: ["keyword1", "keyword2", "keyword3"]
difficulty_level: "beginner|intermediate|advanced|expert"
estimated_time: "< 1 minute|1-5 minutes|5-15 minutes|> 15 minutes"
confidence_score: 0.85
dependencies: []
related_prompts: []
---
```

## Required Sections

### Metadata (Tier 1 - Required)
- **Purpose**: Brief description of what this prompt does
- **Category**: generation/classification/transformation/summarization/extraction
- **Last Updated**: YYYY-MM-DD

### Enhanced Metadata (Tier 2)
- **Domain**: Specific area (e.g., supply-chain, frontend, API)
- **Complexity**: simple/moderate/complex

### Prompt (Required)
```
[Your prompt goes here. Use clear, specific instructions.]
```

### Example Usage (Required)
**Input:**
```
[Example input]
```

**Expected Output:**
```
[Example output]
```

### Variables (Tier 2)

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `{variable_name}` | What this variable represents | Yes/No | Default value |

### Guidelines (Tier 2)
- Specific usage guidelines
- Performance considerations
- Common pitfalls to avoid

### Related Prompts (Tier 2)
- Links to related prompts in this library

### Compliance Notes (If applicable)
- Compliance considerations
- Data handling requirements
- Security notes
