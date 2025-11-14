# Six MCP Tools for Local Development



*My local AI-assisted development environment*

---

## Quick Start: Copy & Paste Setup

If you're setting up a new project and want to skip straight to a working configuration, here's a prompt you can paste into Claude:

```
Please set up my local MCP environment using the essential 6-tool configuration:

1. JetBrains MCP (auto-configured via IntelliJ)
2. Filesystem MCP: npx @modelcontextprotocol/server-filesystem $(pwd)
3. Git MCP: npx @cyanheads/git-mcp-server
4. Serena MCP: uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project $(pwd)
5. Memory MCP: npx @modelcontextprotocol/server-memory
6. Tree-sitter MCP: python3 -m mcp_server_tree_sitter

Requirements:
- Use JetBrains MCP for all file operations in project directories
- Configure Git MCP without repository path argument (uses current directory)
- Ensure Filesystem MCP is scoped to project directory only
- Verify all connections with: claude mcp list

Please confirm setup and verify all 6 servers show "✓ Connected"
```

---


### 1. JetBrains MCP - Your IDE Bridge

If you're using IntelliJ (or any JetBrains IDE), this is the star of the show. It connects Claude directly to your IDE's intelligence.

**Why it matters:**
Instead of Claude doing text searches with `grep`, it uses IntelliJ's cached analysis. Instead of risky find-and-replace operations, it uses the IDE's safe refactoring engine that updates all references automatically.

**Real example:**
I needed to rename a function that was used in 47 places across 12 files. Normally I'd spend time finding all usages, updating them manually, fixing imports, and inevitably missing a few. With JetBrains MCP, Claude did it in under a second with zero errors.

**How it works:**
Auto-connects when IntelliJ is running on `http://localhost:64342/sse`. No configuration needed.

---

### 2. Filesystem MCP - Controlled File Access

This one's straightforward: it lets Claude read and write files with explicit permissions.

**Why it matters:**
You want Claude to access your project files, but you don't want it accessing your entire system. Filesystem MCP lets you scope access to just your project directory.

**Setup tip:**
```bash
# Good - project only
claude mcp add filesystem npx @modelcontextprotocol/server-filesystem $(pwd)

# Bad - everything
claude mcp add filesystem npx @modelcontextprotocol/server-filesystem /
```

---

### 3. Git MCP - Automated Version Control

This tool lets Claude handle your git operations intelligently.

**Why it matters:**
Claude can analyze your changes and generate proper commit messages automatically. No more lazy "fix stuff" commits - it actually looks at what changed and writes meaningful, conventional-commit-style messages.

**Real example:**
I used to spend 2-3 minutes thinking about commit messages. Now I just ask Claude to review my changes and commit them. It analyzes the diffs and creates messages like:
```
feat(auth): implement OAuth2 integration

- Add JWT token management
- Implement refresh token rotation
- Add user session middleware
```

**Important note:**
Use `@cyanheads/git-mcp-server` - the official-sounding `@modelcontextprotocol/server-git` doesn't actually exist on npm.

---

### 4. Serena MCP - Smart Code Navigation

This is where things get interesting. Serena doesn't just read code, it understands it semantically.

**Why it matters:**
Instead of reading entire 500-line files (which burns through your context budget), Serena can navigate directly to specific functions or classes and read just what you need.

**Real example:**
I was tracking down a bug across 50+ files. Instead of reading all 50 (massive token usage), Serena helped Claude navigate directly to the 3 relevant functions using symbol relationships. Problem solved in minutes instead of hours.

---

### 5. Memory MCP - Persistent Context

Claude forgets everything between sessions. Memory MCP fixes that.

**Why it matters:**
Store architectural decisions, patterns, and context that Claude can reference later - even weeks later.

**Real example:**
When a new developer joined our team, instead of a 2-hour architecture walkthrough, I asked Claude to explain our auth flow. It pulled from Memory MCP and gave a complete, accurate overview instantly.

---

### 6. Tree-sitter MCP - Universal Code Parsing

Fast, language-agnostic code parsing for 40+ languages.

**Why it matters:**
Works across TypeScript, Python, Go, Rust, Java - whatever you're using. One tool, consistent parsing, no language-specific errors.

---

## Essential MCP Tool Configuration

### 1. JetBrains MCP: IDE-Integrated Semantic Operations

**Purpose:** Provides direct integration with IntelliJ IDEA's semantic analysis engine for code intelligence and safe refactoring operations.

**Technical Implementation:**
```typescriptd
// Traditional text-based approach:
1. grep -r "getUserData" .          // Text pattern matching
2. Manual file editing               // Manual reference tracking
3. Manual import updates             // Error-prone process
4. Manual testing                    // Post-hoc validation

// JetBrains MCP semantic approach:
mcp__jetbrains__rename_refactoring({
  symbolName: "getUserData",
  newName: "fetchUserProfile"
})
// Result: <1 second execution, 47 references updated across 12 files
```

**Core Capabilities:**
- **Performance:** 10-50x faster than text-based tools via IntelliJ's cached AST analysis
- **Semantic Intelligence:** Type system awareness, symbol resolution, inheritance tracking
- **Safe Refactoring:** Pre-validation with conflict detection
- **IDE Integration:** Direct access to terminal, test runners, and build configurations

**Use Case:** TypeScript monorepo refactoring (1000+ files). JetBrains MCP completed 200+ function reference updates in 3 seconds with zero errors. Equivalent manual operation estimated at 2-3 hours with high error probability.

**Connection:** Auto-configured via SSE (Server-Sent Events)
```
http://localhost:64342/sse
```

**Available Tools:** 20+ specialized tools including code intelligence, refactoring, project analysis, and development operations

---

### 2. Filesystem MCP: Controlled File System Access

**Purpose:** Provides explicit-permission file system operations with project-scoped access controls.

**Technical Implementation:**
```typescript
// Controlled file operations with explicit permissions
mcp__filesystem__read_text_file({ path: "package.json" })
mcp__filesystem__write_file({ 
  path: "config/settings.json",
  content: "..." 
})
```

**Core Capabilities:**
- **Security Model:** Explicit permission scoping to project directory
- **Cross-platform Compatibility:** IDE-independent operation
- **Performance:** Optimized for simple read/write operations

**Security Configuration:**
```bash
# Recommended: Project-scoped access
claude mcp add filesystem npx @modelcontextprotocol/server-filesystem $(pwd)

# Not recommended: Root-level access
claude mcp add filesystem npx @modelcontextprotocol/server-filesystem /
```

---

### 3. Git MCP: Automated Version Control Operations

**Purpose:** Enables AI-driven version control operations with context-aware commit message generation and change analysis.

**Technical Implementation:**
```typescript
// Context-aware git workflow automation
mcp__git__git_status()        // Repository status
mcp__git__git_diff()           // Change analysis
mcp__git__git_commit({         // Semantic commit generation
  message: "feat: add OAuth2 authentication\n\n- Implement JWT token flow\n- Add user session management\n- Update security middleware"
})
mcp__git__git_push()           // Remote synchronization
```

**Core Capabilities:**
- **Semantic Commit Generation:** Automated analysis-based commit messages following conventional commit standards
- **Pre-commit Review:** AI-assisted diff analysis before committing
- **History Analysis:** Code evolution tracking and decision context

**Impact:** Eliminates manual commit message composition while improving commit quality and repository history clarity. Conventional commit format adherence improves automated changelog generation and semantic versioning.

**Implementation Note:** Uses `@cyanheads/git-mcp-server` package (Note: `@modelcontextprotocol/server-git` is not available on npm)

---

### 4. Serena MCP: Semantic Code Analysis and Navigation

**Purpose:** Provides token-efficient code navigation through semantic symbol analysis rather than full-file reading.

**Technical Implementation:**
```typescript
// Traditional approach (inefficient):
Read("src/auth/UserService.ts")  // 500 lines, 10k tokens

// Serena semantic approach (optimized):
find_symbol({
  name_path: "UserService/authenticateUser",
  include_body: true
})  // Target method only, 50 lines, 1k tokens
```

**Core Capabilities:**
- **Token Optimization:** 90% reduction in context usage vs full-file reading
- **Semantic Search:** Symbol-based navigation independent of text patterns
- **Structural Understanding:** Code structure awareness for safe refactoring operations
- **Cross-reference Analysis:** Symbol relationship mapping across codebase

**Use Case:** Bug investigation across 50+ TypeScript files. Serena enabled direct navigation to 3 relevant functions via symbol relationships, avoiding full-file reads. Result: Minutes vs hours for bug isolation.

---

### 5. Memory MCP: Persistent Context Management

**Purpose:** Enables cross-session context persistence through knowledge graph storage, addressing Claude's stateless nature between sessions.

**Technical Implementation:**
```typescript
// Architecture decision storage
create_entities({
  entities: [{
    name: "Authentication Pattern",
    entityType: "Architecture Decision",
    observations: [
      "Using JWT tokens with 15-min expiry",
      "Refresh tokens stored in httpOnly cookies",
      "OAuth2 via Passport.js library"
    ]
  }]
})

// Context retrieval across sessions
search_nodes({ query: "authentication" })
// Returns stored architectural decisions and implementation details
```

**Core Capabilities:**
- **Session Persistence:** Maintains project context across multiple Claude sessions
- **Knowledge Graph Structure:** Relationship mapping between architectural concepts
- **Team Knowledge Sharing:** Centralized context accessible across team members

**Use Case:** Onboarding optimization. New team member onboarding reduced from 2-hour manual walkthrough to 30-second AI-generated architecture overview using stored Memory MCP context.

---

### 6. Tree-sitter MCP: Multi-Language AST Analysis

**Purpose:** Provides language-agnostic Abstract Syntax Tree (AST) generation for structural code analysis across multiple programming languages.

**Technical Implementation:**
```typescript
// Universal AST generation
tree_sitter.parse("TypeScript", codeContent)
// Returns: Complete AST with node types, relationships, and hierarchy

// Supported languages (40+):
// TypeScript, JavaScript, Python, Go, Rust, Java, C++, C#, Ruby, PHP, etc.
```

**Core Capabilities:**
- **Incremental Parsing:** Efficient re-parsing of only modified code sections
- **Language Agnostic:** Single interface for multi-language codebases
- **Structural Analysis:** AST-based understanding vs text pattern matching

**Use Case:** Polyglot monorepo development (TypeScript, Python, Go, Rust, SQL). Tree-sitter provides consistent parsing interface across all languages, eliminating language-specific parsing errors and enabling unified code analysis workflows.

---

## What I Removed (And Why You Should Too)

### claude-flow, ruv-swarm, and Friends

These were some of the first things I installed. They sounded powerful - swarm orchestration, advanced coordination, multi-agent workflows. The reality? They're designed for MCP *coordination*, not execution. Claude Code already has built-in tools for actually doing the work.

Think of it like this: these tools are the project manager, but Claude Code is the actual development team. You don't need two project managers.

### flow-nexus

This was the biggest context hog. It has 70+ tools, which sounds amazing until you realize you're never going to use 95% of them. It was eating up a huge chunk of my context budget for features I didn't need.

Lesson learned: Having every possible feature available doesn't mean you'll use them. Better to have the right tools than all the tools.

### lsp-mcp and claude-context

These kept failing to connect, and when they did work, they didn't do anything that JetBrains MCP and Serena weren't already handling better. Sometimes tools are just redundant.

---

## Observed Results

Here's what changed after simplifying:

**Before (15+ MCP servers):**
- Context usage at 178% of budget
- Notable response latency
- Intermittent connection failures
- Time spent managing tool configurations

**After (6 essential MCPs):**
- Context usage at 55% of budget
- Consistently fast responses
- 100% connection stability
- Tools become invisible infrastructure

The most valuable outcome wasn't just performance - it was focus. Six purposeful tools you actively use beats fifteen installed-and-forgotten tools any day.

---

## How to Set This Up

Here's the installation script. Save it as `setup-essential-mcp.sh` and run it:

```bash
#!/bin/bash
PROJECT_PATH=$(pwd)
echo "Setting up essential MCP tools for: $PROJECT_PATH"

# JetBrains MCP (auto-configured when IntelliJ is open)
echo "JetBrains MCP - will auto-configure when IntelliJ runs"

# Filesystem MCP (project-scoped)
claude mcp add filesystem npx @modelcontextprotocol/server-filesystem "$PROJECT_PATH"

# Git MCP (no path argument needed)
claude mcp add git npx @cyanheads/git-mcp-server

# Serena MCP (semantic intelligence)
claude mcp add serena uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project "$PROJECT_PATH"

# Memory MCP (persistent context)
claude mcp add memory npx @modelcontextprotocol/server-memory

# Tree-sitter MCP (code parsing)
pip3 install -q mcp-server-tree-sitter
claude mcp add tree-sitter python3 -m mcp_server_tree_sitter

echo ""
echo "Done! Checking connections..."
claude mcp list
```

**Verify it worked:**
```bash
claude mcp list
```

You should see all 6 servers with "✓ Connected" next to them.

---

## Performance Metrics: Before and After

### Baseline Configuration (Bloated)

```
Context Usage: 356k / 200k tokens (178% of budget)
MCP Tools Overhead: ~250k tokens (125% of budget)
Failed Connections: 3/10 servers (30% failure rate)
Average Response Time: 3-5 seconds
Code Operations: Text-based pattern matching
Context Overruns: Frequent
```

### Optimized Configuration (Lean)

```
Context Usage: 110k / 200k tokens (55% of budget)
MCP Tools Overhead: ~21k tokens (10.5% of budget)
Failed Connections: 0/6 servers (100% success rate)
Average Response Time: <1 second
Code Operations: Semantic analysis via AST
Context Overruns: Zero
```

**Measured Improvements:**
- Context overhead reduction: 90% (250k → 21k tokens)
- Operation speed improvement: 10-50x on semantic operations
- Connection stability: 100% (zero failures)
- Response time improvement: 3-5x faster

---

## Installation and Configuration

### System Requirements

- IntelliJ IDEA (or compatible JetBrains IDE) - running
- Claude Code CLI - installed
- Node.js and npm - available in PATH
- Python 3.x - installed and available

### Automated Setup Script

```bash
#!/bin/bash
# Filename: setup-essential-mcp.sh
# Execution: bash setup-essential-mcp.sh

PROJECT_PATH=$(pwd)
echo "Configuring essential MCP tools for: $PROJECT_PATH"

# 1. JetBrains MCP (auto-configured via IDE)
echo "JetBrains MCP - auto-configured"

# 2. Filesystem MCP (project-scoped)
claude mcp add filesystem npx @modelcontextprotocol/server-filesystem "$PROJECT_PATH"
echo "Filesystem MCP - configured"

# 3. Git MCP (current directory context)
claude mcp add git npx @cyanheads/git-mcp-server
echo "Git MCP - configured"

# 4. Serena MCP (semantic code intelligence)
claude mcp add serena uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project "$PROJECT_PATH"
echo "Serena MCP - configured"

# 5. Memory MCP (persistent context)
claude mcp add memory npx @modelcontextprotocol/server-memory
echo "Memory MCP - configured"

# 6. Tree-sitter MCP (AST parsing)
pip3 install -q mcp-server-tree-sitter
claude mcp add tree-sitter python3 -m mcp_server_tree_sitter
echo "Tree-sitter MCP - configured"

echo ""
echo "Configuration complete. Verifying connections..."
claude mcp list
```

### Connection Verification

```bash
claude mcp list
```

Expected output: All 6 servers displaying "✓ Connected" status

---

## Practical Examples

### Refactoring Across Multiple Files

Let's say you need to rename a function that's used in 47 places across 12 files.

**The old way:**
- Search with `grep` to find all usages
- Manually edit each file
- Hope you didn't miss any references
- Fix the bugs you inevitably introduced
- Time: 30+ minutes, high stress

**With JetBrains MCP:**
Just tell Claude: "Rename oldFunction to newFunction"

Claude uses the IDE's refactoring engine to update all references automatically. Done in seconds, zero errors.

---

### Understanding Legacy Code

You join a project with 1000+ TypeScript files and need to understand how authentication works.

**The inefficient way:**
Read 20+ auth-related files, burning through 120k tokens of your context budget.

**With Serena MCP:**
Claude navigates semantically - finds the entry point, maps relationships between components, and reads only the relevant methods. Same understanding, 95% less context used.

---

### Git Workflow

**Without Git MCP:**
```bash
git status
git diff
# Think for 2 minutes about commit message
git commit -m "fix stuff"  # Lazy but we've all done it
git push
```

**With Git MCP:**
Tell Claude: "Review my changes and commit"

Claude analyzes the actual code changes and generates a proper commit message:
```
feat(auth): implement OAuth2 integration

- Add JWT token management
- Implement refresh token rotation
- Add user session middleware
```

Professional commits, zero effort.

---

## What Works Well

Some advice based on what I've found effective:

**Let JetBrains MCP handle file operations** - If you're in a project directory and IntelliJ is open, JetBrains MCP is faster and safer than bash commands or the standard Write tool.

**Let Claude choose the right tool** - Claude knows which tool is best for each operation. You don't need to micromanage tool selection.

**Use Memory MCP for decisions** - Store your architectural decisions and patterns. When you come back to the project weeks later, Claude will remember the context.

**Trust semantic operations** - When Claude can use semantic tools (like JetBrains MCP or Serena), it's way more reliable than text-based find/replace.

**Keep IntelliJ running** - JetBrains MCP works best when your IDE is open and has already indexed the project.

## What to Avoid

**Don't install tools "just in case"** - If you haven't used it in two weeks, you don't need it. Remove it.

**Don't bypass the right tool** - If JetBrains MCP is available, don't use bash to edit files. If you can use semantic refactoring, don't use text find/replace.

**Resist the urge to add more** - When something doesn't work perfectly, the answer isn't usually "add another tool." It's usually "use the right tool correctly."

---

## My Configuration Philosophy

After seeing what works and what doesn't, here's what I believe:

**Essential beats comprehensive.** Six tools I actually use every day are infinitely more valuable than 50 tools I installed and forgot about.

**Performance beats features.** A fast semantic operation beats a slow text operation every time, even if the text operation has more options.

**Stability beats novelty.** Six tools with 100% uptime beats 15 tools where half are broken at any given time.

---

## If You're Using VS Code

Good news - this setup works for VS Code too, with one adjustment: you won't have JetBrains MCP (obviously), but Serena handles the semantic intelligence.

Your tool list becomes:
1. ~~JetBrains MCP~~ (N/A)
2. Filesystem MCP ✓
3. Git MCP ✓
4. Serena MCP ✓ (does the heavy lifting)
5. Memory MCP ✓
6. Tree-sitter MCP ✓

**VS Code setup:**
```bash
claude mcp add filesystem npx @modelcontextprotocol/server-filesystem $(pwd)
claude mcp add git npx @cyanheads/git-mcp-server
claude mcp add serena uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project $(pwd)
claude mcp add memory npx @modelcontextprotocol/server-memory
claude mcp add tree-sitter python3 -m mcp_server_tree_sitter
```

The experience is pretty similar - Serena provides the semantic code understanding that JetBrains MCP provides for IntelliJ users.

---

## Common Mistakes I've Seen

**Installing everything that sounds useful**
The "more is better" trap. If you're not using a tool after two weeks, remove it. Your context budget will thank you.

**Not checking context usage**
Run `/context` regularly. If you're over 100%, you need to remove some tools. Simple as that.

**Using text tools when semantic tools exist**
If you can do a semantic refactoring with JetBrains MCP, don't do a text find/replace. The semantic version is faster and safer.

---

## Quick Troubleshooting

**JetBrains MCP won't connect:**
- Make sure IntelliJ is actually running
- Check Settings → Build → Debugger → "Can accept external connections" is enabled
- Restart IntelliJ if needed

**Git MCP failing:**
Remove it and re-add it without the repository path:
```bash
claude mcp remove git
claude mcp add git npx @cyanheads/git-mcp-server
```

**Context still too high:**
Check what you have installed:
```bash
claude mcp list
```
You should see exactly 6 servers (or 5 if you're on VS Code). If there are more, remove them.

---

## The Bottom Line

Here's what I observed after simplifying from 15+ tools to 6:

**Context usage went from 178% to 55%** - plenty of room for actual code
**Connection stability went to 100%** - no more failed servers
**Response times improved noticeably** - Claude is just faster
**Mental overhead disappeared** - I stopped thinking about MCP tools

The best part? I didn't lose any functionality I actually used. Those 9+ tools I removed weren't adding value - they were just consuming resources.

---

## Try It Yourself

If you're running into context issues or slow responses:

1. Check what you have: `claude mcp list`
2. Check your usage: `/context`
3. Remove anything you haven't used in 2 weeks
4. Use the setup script above to install just the essential 6

The difference is noticeable.

---

## Resources

- **JetBrains MCP docs:** See `docs/jetbrains-mcp-integration.md` in this repo
- **Serena MCP:** https://github.com/oraios/serena
- **Claude Code:** https://docs.claude.com/claude-code
- **MCP Protocol:** https://modelcontextprotocol.io/

---

## About

I'm a Distinguished Engineer focused on AI-assisted development and developer productivity. I noticed the MCP overhead issue, measured it, and optimized it. These are my findings.

**Connect:**
- GitHub: [@samart](https://github.com/samart)
- Twitter: [@samaroo_dev]

---

*Last updated: November 2025*
