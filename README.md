# iOS Accessibility Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash command that turns Claude into an iOS accessibility expert. Based on the book ["Про доступность iOS"](https://rubanov.dev/a11y/) by Mikhail Rubanov.

## What it does

The `/a11y` skill gives Claude deep knowledge of iOS accessibility APIs and best practices for both UIKit and SwiftUI, including:

- VoiceOver, Voice Control, and Switch Control support
- `accessibilityLabel`, `accessibilityValue`, `accessibilityTraits`, and `accessibilityHint`
- Element grouping and containers
- Adjustable elements and custom actions
- Dynamic Type support
- Navigation: notifications, modals, scrub gesture, magic tap, custom rotors
- System accessibility settings to respect
- Testing strategies (manual, automated, snapshot tests)

## Setup

### Option 1: Add to your iOS project (recommended)

Copy the `.claude/` directory into your iOS project's root:

```bash
cp -r .claude/ /path/to/your/ios-project/.claude/
```

This makes the `/a11y` command available whenever you use Claude Code in that project.

### Option 2: Add as a global skill

Copy the command file to your home-level Claude config so it's available in all projects:

```bash
mkdir -p ~/.claude/commands
cp .claude/commands/a11y.md ~/.claude/commands/a11y.md
```

## Usage

Open Claude Code in your iOS project and use the `/a11y` slash command with any prompt:

```
/a11y Review this file for accessibility issues

/a11y Make this cell accessible with VoiceOver

/a11y Add Dynamic Type support to this view

/a11y How should I make this custom slider accessible?

/a11y Review the current screen and suggest grouping improvements
```

### Example workflows

**Review a specific file:**
```
/a11y Review Sources/Views/MenuCell.swift for accessibility
```

**Make a screen accessible step by step:**
```
/a11y Walk me through making the checkout screen fully accessible
```

**Fix a specific issue:**
```
/a11y The VoiceOver focus order is wrong on the product detail screen
```

**Generate tests:**
```
/a11y Write accessibility snapshot tests for OrderCell
```

## Auto-apply to every screen (CLAUDE.md rules)

To make Claude **automatically** check accessibility on every screen you work on, add a `CLAUDE.md` file to your iOS project root. `CLAUDE.md` contains instructions Claude Code reads on every conversation.

### Basic rule

Create a `CLAUDE.md` in your project root with:

```markdown
## Accessibility

When creating or modifying any UIView, UIViewController, or SwiftUI View:
- Run /a11y to review the accessibility of the changed code
- Ensure every interactive element has accessibilityLabel and appropriate traits
- Ensure decorative images are hidden from VoiceOver
- Ensure Dynamic Type is supported (no fixed heights, use dynamic fonts)
```

### Comprehensive rule

For stricter enforcement:

```markdown
## Accessibility Requirements

Every UI change MUST pass these checks before being considered complete:

1. **Labels**: Every interactive element has `accessibilityLabel` and appropriate `accessibilityTraits`
2. **Grouping**: Related elements are combined (`.accessibilityElement(children: .combine)` in SwiftUI, or `isAccessibilityElement = true` with combined label in UIKit)
3. **Decorative elements**: Hidden from VoiceOver (`isAccessibilityElement = false` / `.accessibilityHidden(true)`)
4. **Dynamic Type**: No fixed heights, use `UIFont.preferredFont(forTextStyle:)` or `.font(.body)`, support `.isAccessibilityCategory` layout changes
5. **Navigation**: Headers marked with `.header` trait, modals set `accessibilityViewIsModal = true`, custom views implement `accessibilityPerformEscape()`
6. **State changes**: Dynamic content posts `UIAccessibility.post(notification:)` appropriately
7. **System settings**: Respect `isReduceMotionEnabled`, `shouldDifferentiateWithoutColor`, `isBoldTextEnabled`, and other accessibility preferences

When writing or reviewing UI code, use /a11y to verify compliance.
```

### Per-screen rule (for new features)

You can also scope the rule to specific features:

```markdown
## Current Sprint: Checkout Redesign

All views in `Sources/Checkout/` must be fully accessible.
Before marking any task as done, run `/a11y Review {filename} for accessibility issues` on every changed view file.
```

## How the skill works

The skill is a markdown file at `.claude/commands/a11y.md` that Claude Code loads as a system prompt when you invoke `/a11y`. It contains structured knowledge about iOS accessibility APIs, patterns, and best practices that Claude uses to give accurate, specific advice.

The `$ARGUMENTS` placeholder at the end of the file is replaced with whatever you type after `/a11y`.

## License

The accessibility knowledge is based on the book ["Про доступность iOS"](https://rubanov.dev/a11y/) by Mikhail Rubanov.
