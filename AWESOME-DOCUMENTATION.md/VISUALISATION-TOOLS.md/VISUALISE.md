---

# 3. VISUALIZATION.md

markdown
# Visualizing Your Infrastructure as Code (IaC)

Below are some recommended tools and methods to visualize your AWS CloudFormation–based architecture.

## 1. Mermaid Diagrams

You’ve seen Mermaid examples in our docs. Mermaid is a powerful, text-based way to create diagrams. You can:
- Embed Mermaid code in your `.md` files (as we did).
- Use the [Mermaid Live Editor](https://mermaid.live) to prototype diagrams quickly.
- Integrate Mermaid in many static site generators (e.g., MkDocs, Docusaurus) for automated diagram rendering.

## 2. cfn-diagram

[cfn-diagram](https://github.com/awslabs/cfn-diagram) is an **open‐source** tool from AWS Labs. It:
- Parses your CloudFormation templates.
- Generates architecture diagrams automatically.
- Can output to formats like `png`, `svg`, or `draw.io` diagrams.

### Usage Example

1. Install:
   bash
   pip install cfn-diagram