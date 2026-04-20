---
agent: agent
description: 'Review completed code changes against task requirements to verify scope alignment, identify gaps, and provide quality assessment with actionable recommendations.'
tools: ['execute/getTerminalOutput', 'execute/createAndRunTask', 'execute/runInTerminal', 'read/problems', 'read/readFile', 'edit/createDirectory', 'edit/createFile', 'edit/editFiles', 'search', 'web', 'azure-mcp/search', 'context7/*', 'sequential-thinking/*', 'todo']
---

# Implementation analysis command
As a Senior Software Engineer expert in Full Stack development, review the implementation against the task. This unified command handles verification of task requirements, scope alignment with consistent quality to identify the gaps and suggest the actionable recommendations.

## Input Parameter (Task File)s: $ARGUMENTS (Mandatory)
**Accepts:** Task file path

**Required Parameters:**
- `task_file_path`: Path to the task file that defines requirements, acceptance criteria, and validation gates (e.g., `.propel/context/tasks/task_001_signin.md`)

**Optional Parameters:**
- `analysis_depth`: "quick" | "standard" | "comprehensive" (default: "standard") - comprehensive mode triggers full analysis of all implications
- `focus_areas`: Specific areas to emphasize with deep analysis (e.g., "security,testing,performance") - each area will be probed extensively for hidden issues

### Parameter Validation
- Validate task file path and accessibility
- Parse basic task file structure to ensure it's processable
- Verify repository context and related code accessibility

## Output
- Artifact generation:
  - `.propel/context/tasks/us_<ID>/reviews/task-review-<task-id>.md`

**Note:** Extract US ID and task ID from task_file_path parameter
- Example: `.propel/context/tasks/us_001/task_001_login.md`
  - US ID: `us_001`
  - Task ID: `task_001`
  - Output: `.propel/context/tasks/us_001/reviews/task-review-task_001.md`
- Print the following:
  - List of rules used by the workflow in bulleted format
  - Evaluation Scores per Quality Evaluation section below (scale: 0-100).
  - Evaluation summary (less than 100 words).
  **Do not save as file.** (console output only)

**Note:**
- **File Handling**: IF file exists → UPDATE changed sections only (delta mode). IF file does not exist → CREATE complete file.
- Always create the output file in manageable smaller chunks to manage memory and processing constraints.
- Always generate a single unified document using the
  - `.propel/templates/task-analysis-template.md` template

### US ID and Task ID Extraction
**Parse from task_file_path:**
1. Extract US ID using pattern: `us_(\d{3})/`
   - Example: `.propel/context/tasks/us_001/task_001.md` → `us_001`
2. Extract Task ID using pattern: `(task_\d{3}[^/]*?)\.md`
   - Example: `task_001_login.md` → `task_001`
3. Construct output path: `.propel/context/tasks/{US_ID}/reviews/task-review-{TASK_ID}.md`
4. Ensure reviews directory exists before writing

## Execution Flow

### Core Principles
- Analyze tasks for requirements verification, test coverage analysis, security compliance, and integration validation
- Gather framework documentation and best practices simultaneously using Context7 MCP
- Map task requirements to actual implementation files and functions
- Perform stepwise reasoning (use Sequential-Thinking MCP) throug complex requirement verfication
- Trace task specifications against actual implementation methodically
- Build layered understanding of business logic correctness and completeness
- Develop causal chains linking requirements to code quality and test coverage

### Implementation Workflow
- **Task File Analysis**: Parse requirements, acceptance criteria, validation gates, todos
- **Code Discovery**: Grep for features/endpoints/components referenced by task
- **Framework Documentation**: Fetch targeted API/guide excerpts for versions in use
- **Pattern Analysis**: Map controller -> service -> repository (backend) and view -> state -> API client (frontend)
- **Infrastructure Changes**: Identify migrations/SQL and configuration changes

#### Requirements Alignment Analysis
- Derive detailed checklist from acceptance criteria and non-functional constraints
- Map each requirement to concrete files, functions, and code lines
- Identify missing implementations and scope gaps
- Assess business logic correctness and completeness

#### Repository Scanning
- Locate implementation files based on task references
- Identify related test files and coverage areas
- Map system integration points and dependencies
- Extract validation commands from task file

#### Gap Analysis
- **Missing Features**: Identify unimplemented requirements
- **Incomplete Logic**: Highlight logical or business-logic errors
- **Test Gaps**: Missing unit tests, integration tests, edge cases
- **Documentation Gaps**: Missing or outdated documentation
- **Security Gaps**: Missing error handling, input validation, security headers
- **AI Gaps** [CONDITIONAL: If task maps to AIR-XXX]:
  - Missing prompt template versioning
  - Guardrails not implemented (input/output validation)
  - Fallback logic incomplete or missing
  - Token budget not enforced
  - Audit logging not configured
  - RAG pipeline incomplete (chunking, retrieval, citation)

#### Risk Analysis
- Identify high-risk code areas and potential failure points
- Evaluate security vulnerabilities and performance impacts
- Analyze transaction boundaries and data consistency
- Review authentication, authorization, and role-based access
- **AI Risk Analysis** [CONDITIONAL: If AIR-XXX in task scope]:
  - Prompt injection attack surface
  - Hallucination risk in critical flows
  - Model provider outage impact
  - Cost runaway scenarios (unbounded tokens)
  - Data leakage through model prompts

### Summary Presentation
Produce comprehensive task analysis reports including:
    - Executive summary with pass/fail verdict and critical issues
    - Requirements traceability matrix mapping specifications to implementation
    - Quality assessment scorecard across all evaluation dimensions
    - Gap analysis identifying missing features, incomplete logic, and test coverage
    - Risk register with high-priority risks and mitigation strategies
    - Prioritized action plan with effort estimates and file-specific recommendations

**Note:** Do not create any file to store the summary presentation.

### Quality Assurance Framework
- Evaluate code quality, maintainability, and pattern adherence
- Analyze error handling, logging, and security implementation
- Review testing coverage and test quality
- Assess integration impact and backwards compatibility

## Guardrails
- `instructions/ai-assistant-usage-policy.instructions.md`: Explicit commands; minimal output
- `instructions/code-anti-patterns.instructions.md`: Avoid god objects, circular deps, magic constants
- `instructions/dry-principle-guidelines.instructions.md`: Single source of truth; delta updates
- `instructions/iterative-development-guide.instructions.md`: Strict phased workflow
- `instructions/language-agnostic-standards.instructions.md`: KISS, YAGNI, size limits, clear naming
- `instructions/markdown-styleguide.instructions.md`: Front matter, heading hierarchy, code fences
- `instructions/performance-best-practices.instructions.md`: Optimize after measurement
- `instructions/security-standards-owasp.instructions.md`: OWASP Top 10 alignment
- `instructions/software-architecture-patterns.instructions.md`: Pattern selection, boundaries

### Extended Standards (apply based on task domain)
- `instructions/code-documentation-standards.instructions.md`: Comment WHY, not WHAT
- `instructions/react-development-standards.instructions.md`: React component patterns
- `instructions/typescript-styleguide.instructions.md`: TypeScript typing & consistency
- `instructions/web-accessibility-standards.instructions.md`: WCAG 2.2 AA validation
- `instructions/backend-development-standards.instructions.md`: Service/controller patterns
- `instructions/frontend-development-standards.instructions.md`: Frontend patterns
- `instructions/ui-ux-design-standards.instructions.md`: Layout/interaction standards
- `instructions/database-standards.instructions.md`: Schema/migration standards
- `instructions/stored-procedure-standards.instructions.md`: Stored procedure patterns
- `instructions/dotnet-architecture-standards.instructions.md`: .NET architecture patterns

**Selection**: Apply only standards matching task domain. Most specific overrides general.

**ALWAYS: Execute Quality Evaluation per 4-tier framework below. Scale: 0-100. Use workflow-specific criteria.**

## Quality Evaluation

**DO NOT SKIP.** Execute `.github/prompts/evaluate-output.prompt.md`:
- **$OUTPUT_FILE**: `.propel/context/tasks/us_XXX/reviews/task-review-*.md`
- **$SCOPE_FILES**: `.propel/context/tasks/us_XXX/task_*.md`
- **--workflow-type**: `task-review`

**Print 4-tier scores to console before completion.**

---