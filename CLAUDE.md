# Claude Code Project Instructions

## Project Overview

This is the **Infrastructure and Platform Management Handbook** - a comprehensive ITIL/ITSM guide published on GitHub Pages.

- **URL:** https://zettai-seigi.github.io/InfrastructureAndPlatformManagementHandbook/
- **Repository:** https://github.com/zettai-seigi/InfrastructureAndPlatformManagementHandbook
- **Theme:** Jekyll with Just the Docs
- **License:** MIT

## Project Structure

```
InfrastructureAndPlatformManagementHandbook/
├── docs/                    # GitHub Pages content (MAIN CONTENT)
│   ├── _config.yml         # Jekyll configuration
│   ├── index.md            # Home page
│   ├── chapters/           # All 19 chapters
│   └── assets/images/      # Published images
├── README.md               # Repository documentation
├── LICENSE                 # MIT License
├── .gitignore              # Git ignore rules
├── CLAUDE.md               # This file
└── CLAUDE_REPLICATION_GUIDE.md  # Full replication documentation
```

## Key Framework Elements

### 8 Critical Success Factors (CSFs)
1. Executive Sponsorship and Strategic Alignment (Leadership)
2. Comprehensive Architecture and Design (Foundation)
3. Automation and Infrastructure as Code (Automation)
4. Security and Compliance Integration (Security)
5. Observability and Monitoring (Visibility)
6. Operational Excellence and Reliability (Operations)
7. Cost Optimization and FinOps (Efficiency)
8. Continuous Improvement and Learning (Sustainability)

### 6 Key Performance Indicators (KPIs)
1. System Availability (target: ≥99.9%)
2. Mean Time to Recovery (target: <30 min)
3. Change Success Rate (target: ≥95%)
4. IaC Coverage (target: ≥95%)
5. Cost Efficiency (decreasing cost per transaction)
6. Security Compliance Score (target: ≥95%)

### 5 Maturity Levels
1. Ad-hoc (Initial) - Manual operations
2. Developing (Basic) - Documented, basic automation
3. Defined (Standardized) - IaC, CI/CD, observability
4. Managed (Measured) - Data-driven decisions
5. Optimizing (Continuous) - Self-healing, continuous improvement

## Content Guidelines

### Chapter Structure
Each chapter should include:
1. YAML frontmatter with layout, title, parent, nav_order, permalink
2. Learning Objectives section
3. Main content with ## and ### headings
4. Tables for structured data
5. ASCII diagrams for technical concepts
6. Code examples where relevant
7. Review Questions
8. Key Takeaways (bullet points)
9. Summary paragraph
10. Chapter Navigation links

### Writing Style
- Professional, instructional tone
- Consistent ITIL/infrastructure terminology
- Cross-reference other chapters where relevant
- Use real-world examples and code samples
- Target 400-600+ lines per chapter (2000-4000 words)

### Code Examples
Use language-specific code blocks:
```hcl
# Terraform examples
resource "aws_instance" "example" {...}
```

```yaml
# Kubernetes/Ansible YAML
apiVersion: v1
kind: Service
```

## Common Tasks

### Editing Chapters
- Chapter files are in `docs/chapters/`
- Use Edit tool for precise changes
- Maintain consistent formatting

### Adding Images
- Place in `docs/assets/images/`
- Reference with `../assets/images/filename.png`

### Git Operations
```bash
git add .
git commit -m "Description"
git push origin main
```

## Important Notes

- Always maintain framework consistency across chapters
- The book is for GitHub Pages - don't create files that break Jekyll
- When editing, verify cross-references remain accurate
- CSF numbering should be consistent (1-8)
- KPI definitions must match across chapters
- Use ASCII art diagrams for technical illustrations
