# Review code

## Usage
/review <description of flow/feature>
Example: /review auth flow
Example: /review payment processing

## Process

### Step 1: Discovery (Haiku - cheap & fast)
Use Claude Haiku to:
- Parse the user's description (e.g., "auth flow")
- Search codebase for all relevant files
- Output: List of file paths involved

### Step 2: Flow Mapping (Haiku - cheap)
Use Claude Haiku to:
- Analyze file dependencies
- Create execution flow diagram (entry → middleware → controller → service → model)
- Output: Code flow path as Mermaid diagram or text

### Step 3: Parallel Review (Sonnet - quality)
Launch 4 Sonnet agents in parallel to review:
1. **Security Agent**: Finds all possible security issues
2. **Logic Agent**: Finds all possible bugs, unhandled cases, errors and so on to make the code bullet proof
3. **Architecture Agent**: Check if the code is has clear separation of concerns and no unnecessary logic or code
4. **Performance Agent**: Finds all possible performance issues

Each agent:
- Reviews ALL discovered files
- References the flow path from Step 2
- Scores issues CRITICAL / HIGH / MEDIUM

### Step 4: Consolidation (Haiku - cheap)
Use Haiku to:
- Deduplicate issues across agents
- Sort by severity