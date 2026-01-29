# Cursor Commands Guide

This guide explains how to use the custom commands available in Cursor IDE after installing CursorKit.

## How Commands Work

Commands in Cursor are triggered using the `/` prefix in the chat interface. When you type `/` followed by a command name, Cursor will use the corresponding command file from `~/.cursor/commands/` to guide the AI's behavior.

## Available Commands

### `/docs` - Generate Documentation

**Purpose:** Create or update technical documentation for code or features.

**When to use:**
- Documenting a new feature
- Updating outdated documentation
- Creating API documentation
- Writing user guides

**Usage:**
```
/docs [what to document]
```

**Examples:**
```
/docs the authentication flow in auth.ts
/docs how to use the UserService class
/docs the API endpoints in the users module
```

**What it does:**
- Analyzes the code/feature to understand what needs documentation
- Identifies the appropriate documentation type (API docs, guides, etc.)
- Creates clear, accurate documentation that matches the codebase
- Includes examples and type information

---

### `/implement` - Implement Features

**Purpose:** Convert ideas into actionable implementation plans and production-ready code.

**When to use:**
- Starting a new feature
- Adding functionality to existing code
- Implementing user stories
- Building new components or modules

**Usage:**
```
/implement [feature description]
```

**Examples:**
```
/implement a user profile page with edit functionality
/implement OAuth 2.0 authentication
/implement a dark mode toggle component
/implement a REST API endpoint for user management
```

**What it does:**
- Analyzes the feature request to understand requirements
- Creates a clear implementation plan with steps
- Identifies quick wins vs complexities
- Generates production-ready code following your project's patterns

**Output includes:**
1. Feature Summary (2-3 lines)
2. Core Requirements (3-5 bullet points)
3. Implementation Steps (numbered, specific)
4. Quick Wins vs Complexities

---

### `/fix` - Fix Bugs

**Purpose:** Diagnose and fix bugs with a systematic approach.

**When to use:**
- Debugging errors
- Fixing broken functionality
- Resolving unexpected behavior
- Addressing production issues

**Usage:**
```
/fix [bug description or error message]
```

**Examples:**
```
/fix the login button doesn't work on mobile
/fix TypeError: Cannot read property 'name' of undefined
/fix the payment form validation is not working
/fix memory leak in the data fetching hook
```

**What it does:**
- Understands the problem (expected vs actual behavior)
- Diagnoses root cause (not just symptoms)
- Applies safe fixes that don't break existing functionality
- Explains what broke and why
- Suggests tests to prevent regression

**Output format:**
- **DIAGNOSIS:** symptom, root cause, location
- **FIX:** explanation and code changes
- **VERIFICATION:** what was tested
- **PREVENTION:** how to avoid similar bugs

---

### `/refactor` - Refactor Code

**Purpose:** Improve code quality while preserving functionality.

**When to use:**
- Improving code readability
- Reducing duplication
- Aligning with codebase patterns
- Enhancing type safety
- Optimizing structure

**Usage:**
```
/refactor [code or file to refactor]
```

**Examples:**
```
/refactor the UserService class to follow our patterns
/refactor this component to use composables
/refactor the authentication logic in auth.ts
```

**What it does:**
- Explores codebase to understand patterns and conventions
- Analyzes code for smells (duplication, complexity, unclear naming)
- Creates a refactoring plan with safety markers
- Preserves existing functionality and patterns
- Aligns code with codebase conventions

**Safety markers:**
- `[SAFE]` - Low risk changes
- `[NEEDS TESTING]` - Should be tested
- `[BREAKING]` - May break existing code

**Refactoring priorities:**
1. Pattern alignment
2. Readability
3. DRY (Don't Repeat Yourself)
4. Type safety
5. Performance (only if obviously bad)

---

### `/review` - Code Review

**Purpose:** Perform thorough code review with structured feedback.

**When to use:**
- Reviewing pull requests
- Checking code quality before commit
- Auditing code for issues
- Ensuring code standards

**Usage:**
```
/review [code to review]
```

**Examples:**
```
/review the changes in auth.ts
/review this component implementation
/review the API service I just wrote
```

**What it does:**
- Checks correctness, architecture, readability
- Identifies performance and security issues
- Evaluates testability
- Provides constructive, prioritized feedback

**Review checklist:**
- ✅ Correctness (does it work?)
- ✅ Architecture (well-structured?)
- ✅ Readability (easy to understand?)
- ✅ Performance (any issues?)
- ✅ Security (vulnerabilities?)
- ✅ Testability (can it be tested?)

**Output format:**
For each issue:
- **Severity:** 🔴 Critical | 🟡 Warning | 🔵 Suggestion
- **Location:** File and line reference
- **Issue:** What's wrong
- **Fix:** How to resolve it

---

### `/test` - Generate Tests

**Purpose:** Create comprehensive test suites for code.

**When to use:**
- Writing tests for new features
- Adding tests for existing code
- Creating test coverage
- Testing edge cases

**Usage:**
```
/test [code to test]
```

**Examples:**
```
/test the UserService class
/test the authentication flow
/test this React component
/test the API endpoint handler
```

**What it does:**
- Analyzes public interfaces and inputs/outputs
- Identifies edge cases and error scenarios
- Generates tests following your project's test framework
- Uses AAA pattern (Arrange, Act, Assert)
- Includes happy path, edge cases, and error cases

**Test categories:**
1. **Happy Path Tests** - Normal expected usage
2. **Edge Cases** - Empty inputs, boundary values, null handling
3. **Error Cases** - Invalid inputs, missing dependencies, failures
4. **Integration Points** - External APIs, database, file system

---

### `/explain` - Explain Code

**Purpose:** Get clear explanations of code or concepts.

**When to use:**
- Understanding unfamiliar code
- Learning new concepts
- Onboarding to a codebase
- Clarifying complex logic

**Usage:**
```
/explain [code or concept]
```

**Examples:**
```
/explain how this authentication middleware works
/explain what React Suspense does
/explain the data flow in this component
/explain the difference between useMemo and useCallback
```

**What it does:**
- Provides clear, step-by-step explanations
- Breaks down complex concepts
- Uses examples and analogies
- Adapts complexity to your skill level
- Highlights common mistakes

**Explanation approach:**
- **Overview:** High-level understanding
- **Breakdown:** Section-by-section walkthrough
- **Key Concepts:** Patterns and techniques used
- **Flow:** Execution flow description
- **Why:** Design decisions explanation

---

## How to Use Commands in Cursor

### Step 1: Open Cursor Chat

Press `Cmd+L` (Mac) or `Ctrl+L` (Windows/Linux) to open the Cursor chat interface.

### Step 2: Type the Command

Type `/` followed by the command name:

```
/docs
/implement
/fix
/refactor
/review
/test
/explain
```

### Step 3: Add Your Request

After typing the command, add your specific request:

```
/implement a user dashboard with charts
/fix the login error on Safari
/review the changes in user-service.ts
```

### Step 4: Review the Response

The AI will use the command's instructions to provide a structured, focused response tailored to that specific task type.

## Command Best Practices

### Be Specific

❌ **Bad:**
```
/implement feature
/fix bug
```

✅ **Good:**
```
/implement a user profile page with edit and delete functionality
/fix the TypeError when submitting the form with empty fields
```

### Provide Context

Include relevant information:
- File names or paths
- Error messages
- Expected vs actual behavior
- Related code snippets

### Use the Right Command

- **`/implement`** - For new features or functionality
- **`/fix`** - For bugs and errors
- **`/refactor`** - For improving existing code
- **`/review`** - For code quality checks
- **`/test`** - For writing tests
- **`/docs`** - For documentation
- **`/explain`** - For understanding code

### Combine with Skills

Commands automatically use the relevant Skills and Rules from your `.cursor/` configuration:
- They identify which skills apply to your request
- They follow your team's coding standards
- They use your project's patterns and conventions

## Tips for Effective Command Usage

1. **Start with the command** - Type `/` and the command name first
2. **Be descriptive** - Include details about what you want
3. **Provide examples** - Show what you're working with
4. **Specify constraints** - Mention any requirements or limitations
5. **Iterate** - Refine your request based on the response

## Example Workflows

### Feature Development

```
1. /implement a shopping cart component
2. /review the shopping cart implementation
3. /test the shopping cart component
4. /docs the shopping cart API
```

### Bug Fixing

```
1. /fix the cart total calculation is wrong
2. /review the fix I just applied
3. /test the cart calculation edge cases
```

### Code Improvement

```
1. /refactor the user authentication logic
2. /review the refactored code
3. /test the refactored authentication
```

## Troubleshooting

### Command Not Recognized

If a command doesn't work:
1. Make sure CursorKit is installed: `cursorkit install`
2. Check that `~/.cursor/commands/` contains the command files
3. Restart Cursor IDE

### Command Not Following Instructions

If the AI doesn't follow the command's instructions:
1. Be more specific in your request
2. Reference the command explicitly: "Use the /implement command format"
3. Check that the command file exists in `~/.cursor/commands/`

### Getting Generic Responses

If responses are too generic:
1. Provide more context about your codebase
2. Include relevant file paths or code snippets
3. Specify which patterns or conventions to follow

---

**Need help?** Check the main [README.md](./README.md) for installation instructions.

