---
description: Research and plan implementation for a feature or task
argument-hint: "[feature description]"
---

# Implementation Planning Command

Help me research and create a comprehensive plan for implementing this feature or task:

**Feature/Task Description:**
$ARGUMENTS

## Planning Process

Follow this systematic approach to create a well-researched implementation plan:

### Phase 1: Parallel Research (Run Simultaneously)

Launch these three agents in parallel to gather comprehensive context:

1. **repo-research-analyst**
   - Analyze the repository structure and existing patterns
   - Review documentation for project conventions
   - Search codebase for similar implementations
   - Identify relevant files and modules

2. **best-practices-researcher**
   - Research industry best practices for the technology
   - Find official documentation and guides
   - Identify common pitfalls and anti-patterns
   - Gather examples from well-regarded projects

3. **framework-docs-researcher**
   - Fetch framework/library documentation relevant to the task
   - Identify version-specific considerations
   - Find implementation examples and patterns
   - Locate configuration requirements

**Key Point:** Run all three agents concurrently using the Task tool with multiple invocations in a single message.

### Phase 2: Code Review (If Modifying Existing Code)

If this task involves modifying existing code in a SvelteKit project, invoke:

4. **kieran-sveltekit-reviewer**
   - Review any existing code that will be modified
   - Ensure changes align with Svelte 5 runes patterns
   - Verify SvelteKit conventions are followed
   - Check for potential breaking changes

### Phase 3: Synthesize and Plan

Based on the research from all agents, create a structured implementation plan:

#### 1. Summary
- Brief description of what needs to be built
- Key technical decisions made based on research
- Estimated complexity (Small/Medium/Large)

#### 2. Technical Approach
- Chosen architecture/pattern (with justification from research)
- Key technologies/libraries to use
- Files that need to be created or modified

#### 3. Implementation Steps

Break down the work into clear, sequential steps:

**Step 1: [Name]**
- What: Specific task description
- Why: Rationale based on research
- How: Brief implementation notes
- Files: Specific file paths

**Step 2: [Name]**
- [Continue pattern...]

#### 4. Testing Strategy
- What needs to be tested
- How to test it
- Edge cases to consider

#### 5. Considerations & Risks
- Potential challenges identified during research
- Performance implications
- Security considerations
- Breaking changes or migration needs

#### 6. Reference Links
- Relevant documentation links from research
- Similar implementations found
- Best practice guides consulted

## Output Format

Present the plan in a clear, actionable format that can be followed step-by-step. Use markdown formatting with:

- Clear section headers
- Numbered steps for sequential tasks
- Bullet points for details
- Code blocks for examples
- File path references (e.g., `src/routes/+page.svelte:42`)

## Success Criteria

A good plan should:
- ✅ Be informed by actual project patterns (from repo-research-analyst)
- ✅ Follow industry best practices (from best-practices-researcher)
- ✅ Use correct framework patterns (from framework-docs-researcher)
- ✅ Follow SvelteKit/Svelte 5 conventions (from kieran-sveltekit-reviewer)
- ✅ Be actionable with specific file paths and steps
- ✅ Include rationale for technical decisions
- ✅ Identify potential risks and edge cases

Remember: The goal is to create a plan that accelerates implementation by doing the research upfront, not to create a perfect plan. Prioritize practical, working solutions over theoretical perfection.
