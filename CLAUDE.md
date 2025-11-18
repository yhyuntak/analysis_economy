# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a research and documentation project analyzing the 11 GICS (Global Industry Classification Standard) sectors. The goal is to build a comprehensive knowledge base covering:

- Historical evolution and development of each sector
- Current market status, trends, and key players
- Relationships between macroeconomic factors (interest rates, inflation, GDP, exchange rates) and sector performance
- Economic cycle characteristics (expansion, peak, contraction, trough)
- Inter-sector correlations and value chain connections
- Investment insights and risk factors

**Language**: All documentation is written in Korean (한국어).

## Repository Structure

```
docs/
├── sectors/          # 11 sector deep-dive analyses
├── macroeconomics/   # Macro factor analysis (interest rates, inflation, cycles, etc.)
├── themes/           # Cross-sector themes (AI, cloud, fintech, ESG, etc.)
└── investment/       # Investment strategies (sector rotation, portfolio construction)
```

## Custom Agent: sector-investment-analyst

**Location**: `.claude/agents/sector-investment-analyst.md`

This specialized agent acts as a veteran investment professional with deep sector knowledge. It provides conversational, insight-driven analysis rather than mechanical checklists or scoring.

**When to invoke this agent:**
- User asks about specific sector performance or outlook
- Questions about macro factors (interest rates, inflation) impact on industries
- Historical industry patterns or cycles
- Supply chain dynamics and inter-industry relationships
- Policy/geopolitical impacts on sectors
- Comparative sector analysis
- **IMPORTANT**: When the user says "트레이더 형님" (analyst-hyung), they are requesting the sector-investment-analyst agent. Invoke it immediately.

**How it differs from general assistance:**
- Provides nuanced, experience-based insights ("in theory X, but in practice Y")
- Conversational tone with analogies and real-world examples
- Balanced perspectives with multiple scenarios
- Focus on what to watch rather than definitive predictions
- Always responds in Korean to match user preference

## Custom Agent: fullstack-dev-master

**Location**: `.claude/agents/fullstack-dev-master.md`

This specialized agent acts as a seasoned fullstack development master with expertise in both traditional web development and cutting-edge AI systems. It provides patient, practical guidance for developers at all skill levels.

**When to invoke this agent:**
- Frontend development questions (React, Vue, TypeScript, CSS, state management)
- Backend development (Node.js, Python, APIs, databases, authentication)
- Fullstack architecture decisions and design patterns
- AI/ML integration (LLM APIs, RAG systems, vector databases, AI agents)
- Code reviews, debugging, and security audits
- Technology stack selection and trade-offs
- DevOps, deployment, and CI/CD questions
- **IMPORTANT**: When the user says "개발자 형님" (developer-hyung), they are requesting the fullstack-dev-master agent. Invoke it immediately.

**How it differs from general assistance:**
- Provides complete, production-ready code examples
- Explains not just "what" but "why" behind decisions
- Progressive learning approach (simple → advanced)
- Real-world trade-off analysis and best practices
- Covers both traditional web development and AI system integration
- Patient mentoring style that encourages questions

## Custom Agent: architecture-advisor

**Location**: `.claude/agents/architecture-advisor.md`

This specialized agent acts as a senior architect who balances product, design, and technical perspectives. It provides comprehensive architectural guidance for building scalable, maintainable systems.

**When to invoke this agent:**
- Architecture or technology stack decisions
- Feature prioritization and roadmap planning
- Trade-off analysis (performance vs. complexity, features vs. timeline)
- Refactoring or system redesign discussions
- MVP scope definition and phased implementation
- UX/technical conflict resolution
- Technical debt assessment
- **IMPORTANT**: When the user says "아키형님" (architect-hyung), they are requesting the architecture-advisor agent. Invoke it immediately.

**How it differs from general assistance:**
- Multi-perspective analysis (PM, Designer, Engineer viewpoints)
- Strategic architectural guidance beyond just code
- Long-term thinking about system evolution
- Practical trade-off evaluation for real-world constraints
- Focus on maintainability and team productivity
- Helps align technical decisions with business goals

## Document Template Structure

Each sector analysis follows this framework (see README.md for full template):

1. Overview and Definition
2. Historical Development (key turning points, tech/regulatory changes)
3. Current Status (market size, key players, trends, valuation)
4. Sub-Industries Breakdown
5. Macroeconomic Sensitivity
   - Interest rate sensitivity
   - Economic cycle characteristics
   - Inflation impact
   - Currency exposure
6. Key Risk Factors
7. Investment Considerations
8. References

## Working with This Repository

### Research and Writing Workflow

1. **Starting a new sector document**: Use the template structure outlined in README.md. Ensure comprehensive coverage of all 8 sections.

2. **Cross-referencing**: When writing about one sector, consider connections to:
   - Related sectors in the value chain (upstream/downstream)
   - Macroeconomic factors that affect multiple sectors
   - Cross-sector themes (e.g., AI impacts IT + Healthcare + Industrials)

3. **Korean language conventions**:
   - Use formal/professional tone (존댓말)
   - Technical terms: Include English in parentheses on first use (e.g., "주가수익비율 (P/E Ratio)")
   - Maintain consistency with existing documents

### Leveraging the Sector Analyst Agent

When user asks sector-related questions:
1. Invoke the sector-investment-analyst agent for deep analysis
2. The agent provides conversational insights based on its specialized knowledge
3. Use agent responses to inform documentation or answer follow-up questions
4. Agent responses can be refined into formal documentation with proper structure

### Document Status Tracking

Check `docs/sectors/GICS_Classification.md` for the checklist of which sectors have been analyzed. Update checkboxes as documents are completed.

## Key Concepts

**GICS 11 Sectors**: Information Technology, Communication Services, Consumer Discretionary, Consumer Staples, Energy, Financials, Health Care, Industrials, Materials, Real Estate, Utilities

**Sector Rotation**: The investment strategy of shifting capital between sectors based on economic cycle phases

**Value Chain Analysis**: Understanding upstream (suppliers) and downstream (customers) relationships between industries

**Cyclical vs Defensive**:
- Cyclical sectors perform well during economic expansion (IT, Consumer Discretionary, Industrials)
- Defensive sectors provide stability during downturns (Consumer Staples, Healthcare, Utilities)

## Research Sources

When adding data or claims to documents, consider citing:
- Central bank reports and monetary policy statements
- Industry association data
- Financial disclosures and earnings reports
- Economic indicators (GDP, CPI, unemployment, etc.)
- Research from investment banks and asset managers

Document sources in the "References" section of each analysis.

## Project Status

**Phase 1** (Current): Basic research and structure
- GICS classification completed
- Sector-investment-analyst agent configured
- Individual sector documents: In progress

**Phase 2**: Deep analysis of all 11 sectors + macro factor documents

**Phase 3**: Integration, cross-sector themes, and investment frameworks
