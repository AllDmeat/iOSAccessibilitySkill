# iOS Accessibility Skills for Claude Code

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash commands that turn Claude into an iOS accessibility expert. Based on the book ["About accessibility on iOS"](https://rubanov.dev/a11y-book/) by Mikhail Rubanov.

## Skills

### `/accessibility` — Review a screen (main entrance)

The main entry point. Ask it to **review accessibility on a screen** — e.g. `/accessibility review accessibility on ProfileScreen` — and it audits the screen against the rules, applies fixes, and delegates cells/carousels/Dynamic Type to the focused skills below. Also covers VoiceOver, Voice Control, Switch Control, and overall accessibility for UIKit and SwiftUI:

- `accessibilityLabel`, `accessibilityValue`, `accessibilityTraits`, and `accessibilityHint`
- Element grouping, containers, and focus order control
- Adjustable elements and custom actions
- Speech customization (pronunciation, language, pitch)
- Navigation: notifications, modals, scrub gesture, magic tap, custom rotors
- Color contrast, Smart Invert, reduce motion
- Drag and drop accessibility
- System accessibility settings to respect
- Testing strategies (manual, automated, snapshot tests)

### `/accessibility-cell` — Cell Accessibility

Focused skill for list / table / collection **cells** — turn a multi-view row into one well-structured element:

- One element per cell (`.combine` / `.ignore` / `isAccessibilityElement`)
- **Label = main content, value = additional content**
- **Hard-to-read values** (addresses, ids, timestamps) → `accessibilityCustomContent` (spoken on demand)
- **All button actions** → accessibility actions (primary activation + named custom actions)
- **Input labels** with alternatives and synonyms for Voice Control
- Hiding decorative avatars / icons / chevrons

### `/accessibility-carousel` — Carousel Accessibility

Focused skill for horizontally paged banners, card decks, galleries, and onboarding pagers:

- Adjustable-control vs container-of-pages model
- Announcing position ("page X of N") and page changes
- Hiding off-screen pages, `accessibilityScroll` paging (UIKit)
- Pausing autoplay under VoiceOver / Switch Control / Reduce Motion

### `/accessibility-dynamic-type` — Dynamic Type

Dedicated skill for text scaling and adaptive layout:

- System and custom font scaling (`UIFontMetrics`, `@ScaledMetric`)
- Adaptive layout (switching axis at accessibility sizes)
- Self-sizing cells and containers
- Large Content Viewer for fixed controls
- Bold text support
- Limiting Dynamic Type when needed
- Testing and verification

## Setup

### Option 1: Add to your iOS project (recommended)

Copy the `.claude/` directory into your iOS project's root:

```bash
cp -r .claude/ /path/to/your/ios-project/.claude/
```

This makes both `/accessibility` and `/accessibility-dynamic-type` commands available whenever you use Claude Code in that project.

### Option 2: Add as a global skill

Copy the command file to your home-level Claude config so it's available in all projects:

```bash
mkdir -p ~/.claude/commands
cp .claude/commands/accessibility.md ~/.claude/commands/accessibility.md
cp .claude/commands/accessibility-dynamic-type.md ~/.claude/commands/accessibility-dynamic-type.md
```

## Usage

Open Claude Code in your iOS project and use the `/accessibility` slash command with any prompt:

```
/accessibility review accessibility on ProfileScreen

/accessibility Review this file for accessibility issues

/accessibility-cell Make this recipient row read as one VoiceOver element

/accessibility-cell Move the hard-to-read values in this cell into custom content

/accessibility-dynamic-type Add Dynamic Type support to this view

/accessibility How should I make this custom slider accessible?

/accessibility Review the current screen and suggest grouping improvements
```

### Example workflows

**Review a specific file:**
```
/accessibility Review Sources/Views/MenuCell.swift for accessibility
```

**Make a screen accessible step by step:**
```
/accessibility Walk me through making the checkout screen fully accessible
```

**Adapt a cell:**
```
/accessibility-cell Turn OrderCell into one element: label, value, custom content, actions
```

**Fix a specific issue:**
```
/accessibility The VoiceOver focus order is wrong on the product detail screen
```

**Generate tests:**
```
/accessibility Write accessibility snapshot tests for OrderCell
```

## Auto-apply to every screen (CLAUDE.md rules)

To make Claude **automatically** check accessibility on every screen you work on, add a `CLAUDE.md` file to your iOS project root. `CLAUDE.md` contains instructions Claude Code reads on every conversation.

### Basic rule

Create a `CLAUDE.md` in your project root with:

```markdown
## Accessibility

When creating or modifying any UIView, UIViewController, or SwiftUI View:
- Run /accessibility to review the accessibility of the changed code
- Ensure every interactive element has accessibilityLabel and appropriate traits
- Ensure decorative images are hidden from VoiceOver
- Run /accessibility-dynamic-type to review Dynamic Type support
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

When writing or reviewing UI code, use /accessibility and /accessibility-dynamic-type to verify compliance.
```

### Per-screen rule (for new features)

You can also scope the rule to specific features:

```markdown
## Current Sprint: Checkout Redesign

All views in `Sources/Checkout/` must be fully accessible.
Before marking any task as done, run `/accessibility Review {filename} for accessibility issues` on every changed view file.
```

## How the skills work

Each skill is a markdown file in `.claude/commands/` that Claude Code loads as a system prompt when you invoke the slash command. They contain structured knowledge about iOS accessibility APIs, patterns, and best practices that Claude uses to give accurate, specific advice.

The `$ARGUMENTS` placeholder at the end of each file is replaced with whatever you type after the command.

| File | Command | Focus |
|------|---------|-------|
| `.claude/commands/accessibility.md` | `/accessibility` | VoiceOver, Voice Control, Switch Control, general accessibility |
| `.claude/commands/accessibility-cell.md` | `/accessibility-cell` | Cell accessibility: label/value/custom-content/actions/input-labels |
| `.claude/commands/accessibility-carousel.md` | `/accessibility-carousel` | Carousel accessibility: paging model, position, autoplay |
| `.claude/commands/accessibility-dynamic-type.md` | `/accessibility-dynamic-type` | Font scaling, adaptive layout, Large Content Viewer |

## License

The accessibility knowledge is based on the book ["Про доступность iOS"](https://rubanov.dev/a11y-book/) by Mikhail Rubanov.
