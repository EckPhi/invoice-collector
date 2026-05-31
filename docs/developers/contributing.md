# Contributing to Invoice Collector

Thank you for your interest in contributing! This guide will help you get started.

## Ways to Contribute

### 1. Report Bugs

Found a bug? [Open an issue](https://github.com/invoice-collector/invoice-collector/issues/new?template=bug-report.yml)

**Include:**

- Description of the bug
- Steps to reproduce
- Expected behavior
- Actual behavior
- Environment (OS, browser, versions)
- Screenshots if applicable

### 2. Request Features

Have an idea? [Open a feature request](https://github.com/invoice-collector/invoice-collector/issues/new?template=feature-request.yml)

**Include:**

- Description of the feature
- Use case
- Benefits
- Possible implementation approach

### 3. Add New Collectors

Want to add support for a new supplier? [Open a collector request](https://github.com/invoice-collector/invoice-collector/issues/new?template=new-collector.yml)

Then follow the [Creating Collectors](creating-collectors.md) guide.

### 4. Improve Documentation

Documentation improvements are always welcome:

- Fix typos
- Add examples
- Clarify explanations
- Add translations

### 5. Write Code

- Fix bugs
- Implement features
- Improve performance
- Refactor code

## Getting Started

### 1. Fork the Repository

Click the "Fork" button on GitHub.

### 2. Clone Your Fork

```bash
git clone https://github.com/YOUR_USERNAME/invoice-collector.git
cd invoice-collector
```

### 3. Add Upstream Remote

```bash
git remote add upstream https://github.com/invoice-collector/invoice-collector.git
```

### 4. Create a Branch

```bash
git checkout -b feature/your-feature-name
```

### 5. Make Changes

Follow the [Development Setup](setup.md) guide.

### 6. Test Your Changes

```bash
npm run test.manual <collector_id>
npm test
```

### 7. Commit Your Changes

```bash
git add .
git commit -m "feat: add support for new supplier"
```

**Commit Message Format:**

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `refactor:` Code refactoring
- `test:` Adding tests
- `chore:` Maintenance tasks

### 8. Push to Your Fork

```bash
git push origin feature/your-feature-name
```

### 9. Create Pull Request

Go to GitHub and create a Pull Request from your branch to `main`.

## Code Style

### TypeScript

- Use TypeScript strict mode
- Define types for all parameters
- Avoid `any` when possible
- Use interfaces for complex types

**Example:**

```typescript
interface InvoiceData {
  id: string;
  amount: string;
  date: Date;
}

async function processInvoice(data: InvoiceData): Promise<void> {
  // Implementation
}
```

### Naming Conventions

- **Classes:** PascalCase (`AmazonCollector`)
- **Functions:** camelCase (`collectInvoices`)
- **Constants:** UPPER_SNAKE_CASE (`MAX_RETRIES`)
- **Private methods:** prefix with `_` (`_parseAmount`)

### Formatting

- 2 spaces for indentation
- Single quotes for strings
- Semicolons required
- Trailing commas in multiline

**Use Prettier:**

```bash
npm run format
```

### Comments

Add comments for:

- Complex logic
- Non-obvious decisions
- TODO items
- Workarounds

**Example:**

```typescript
// Wait for dynamic content to load
// The supplier's site uses lazy loading
await page.waitForTimeout(2000);

// TODO: Replace with more robust detection
if (await page.$('.loading')) {
  await page.waitForSelector('.loaded');
}
```

## Testing Requirements

All contributions must include tests:

### For New Collectors

```typescript
describe('NewCollector', () => {
  it('should collect invoices', async () => {
    // Test implementation
  });
  
  it('should handle authentication errors', async () => {
    // Test implementation
  });
});
```

### For Bug Fixes

Add a test that reproduces the bug:

```typescript
it('should handle edge case X', async () => {
  // Test that would have failed before the fix
});
```

### For Features

Add tests for the new functionality:

```typescript
describe('New Feature', () => {
  it('should work as expected', async () => {
    // Test implementation
  });
});
```

## Pull Request Guidelines

### PR Title

Use descriptive titles:

- ✅ "feat: add Carrefour collector"
- ✅ "fix: handle timeout in Amazon collector"
- ❌ "Update code"
- ❌ "Fix bug"

### PR Description

Include:

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Manual testing completed
- [ ] Automated tests added
- [ ] All tests passing

## Checklist
- [ ] Code follows style guidelines
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
```

### Review Process

1. Automated checks run (tests, linting)
2. Maintainers review code
3. Feedback addressed
4. Approved and merged

## Development Workflow

### Keep Your Fork Updated

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Rebase Your Branch

```bash
git checkout feature/your-feature
git rebase main
git push --force-with-lease origin feature/your-feature
```

### Resolve Conflicts

```bash
git fetch upstream
git rebase upstream/main
# Resolve conflicts
git add .
git rebase --continue
git push --force-with-lease
```

## Community Guidelines

### Code of Conduct

- Be respectful and inclusive
- Welcome newcomers
- Provide constructive feedback
- Focus on the code, not the person

### Communication

- Use clear, concise language
- Ask questions if unclear
- Provide context in issues/PRs
- Be patient with reviewers

### Getting Help

- 💬 [Discord](https://discord.gg/dMXTdpxMqY) - Ask questions
- 📚 [Documentation](../index.md) - Read the docs
- 🐛 [Issues](https://github.com/invoice-collector/invoice-collector/issues) - Search existing issues

## Recognition

Contributors are recognized in:

- CONTRIBUTORS.md file
- Release notes
- Project website

## License

By contributing, you agree that your contributions will be licensed under the project's license.

## Questions?

Don't hesitate to ask:

- Open a discussion on GitHub
- Ask in Discord
- Comment on relevant issues

Thank you for contributing! 🎉
