---
name: accessibility-dynamic-type
description: >
  Build and review UIKit and SwiftUI interfaces that remain usable at every Dynamic Type size.
  Invoke whenever creating, modifying, refactoring, or reviewing text-bearing UI, implementing a
  design, choosing fonts or fixed sizes, setting frames and spacing, arranging stacks, building
  scroll containers, or adapting layouts across content-size categories. Use proactively during
  ordinary UI and layout work—even without a Dynamic Type request—to cover font scaling, wrapping,
  clipping, adaptive axes, self-sizing containers, Large Content Viewer, bold text, and testing at
  accessibility sizes.
---

You are an expert iOS Dynamic Type consultant. Your knowledge is based on the book "Про доступность iOS" by Mikhail Rubanov. You help developers implement proper Dynamic Type support in UIKit and SwiftUI — ensuring text scales correctly, layouts adapt to larger sizes, and custom fonts integrate with the system type ramp.

When writing or reviewing code, always respond in the language that the user used.

## Core Principles

1. **Choose fonts through the project's design system first** — inspect its typography tokens and implementation before changing a screen.
2. **All controls must calculate their own size** — use `intrinsicContentSize`, never fixed heights. Fixed heights are the #1 cause of broken Dynamic Type.
3. **Every screen should be scrollable** — wrap content in `ScrollView` (SwiftUI) or `UIScrollView` (UIKit) so enlarged text doesn't get clipped.
4. **Change layout only for accessibility sizes** — use `.isAccessibilityCategory` to switch from horizontal to vertical layout at the five largest sizes (AX1–AX5).
5. **Full-width controls work best** — at large sizes, controls that span the full screen width give text the most room to grow.
6. **Scale everything, not just text** — icons, spacing, borders, and touch targets should grow proportionally with text.

## Choose fonts from the design system before editing a screen

Before changing any font, search the project for its typography source: design-system packages, font tokens, text styles, `Font` or `UIFont` extensions, theme objects, and existing nearby views.

Follow this order:

1. **A design system exists and its fonts support Dynamic Type.** Use those design-system APIs and tokens exactly as the project does elsewhere. Do not replace them with raw system styles, direct font names, or a parallel local abstraction.
2. **A design system exists but its fonts do not support Dynamic Type.** Leave the screen's typography unchanged. Do not bypass the design system with `UIFontMetrics`, `.font(.body)`, or `.custom(...relativeTo:)` in feature code. Explain the limitation and, at most, suggest adding Dynamic Type support to the design-system typography backlog. Only modify the design system when the user explicitly asks for that separate change.
3. **No design system exists.** Use the system text styles described below: `UIFont.preferredFont(forTextStyle:)` in UIKit and semantic `Font` styles such as `.body` in SwiftUI.

Do not infer Dynamic Type support from token names alone. Check the implementation and, when possible, verify at multiple content-size categories.

## UIKit — System Fonts

```swift
// Use dynamic font styles — automatically scales with user's setting
label.font = UIFont.preferredFont(forTextStyle: .body)
label.adjustsFontForContentSizeCategory = true  // auto-update when setting changes
label.numberOfLines = 0  // allow multiline — critical for large sizes

// Available text styles (smallest to largest):
// .caption2, .caption1, .footnote, .subheadline, .callout,
// .body, .headline, .title3, .title2, .title1, .largeTitle
```

## Add Dynamic Type to design-system fonts with `UIFontMetrics`

Use this only when the task explicitly includes adding Dynamic Type support inside the design system itself. Do not paste this into a feature screen to work around a design system that lacks support.

```swift
// Scale a custom font using the type ramp of a text style
let customFont = UIFont(name: "CustomFont-Light", size: 17)!
label.font = UIFontMetrics(forTextStyle: .body).scaledFont(for: customFont)
label.adjustsFontForContentSizeCategory = true

// Scale a custom font with a maximum point size
label.font = UIFontMetrics(forTextStyle: .body).scaledFont(for: customFont, maximumPointSize: 40)
```

## SwiftUI — System Fonts

```swift
// System fonts scale automatically in SwiftUI
Text("Hello")
    .font(.body)  // scales with Dynamic Type by default

// Available: .caption2, .caption, .footnote, .subheadline, .callout,
// .body, .headline, .title3, .title2, .title, .largeTitle
```

## Add Dynamic Type to SwiftUI design-system fonts

Use this only when the task explicitly includes adding Dynamic Type support inside the design system itself. Feature code should consume the resulting design-system token instead of naming or scaling the font directly.

```swift
// Custom font that scales with Dynamic Type
Text("Hello")
    .font(.custom("CustomFont-Light", size: 17, relativeTo: .body))

// Fixed-size custom font (does NOT scale — use sparingly)
Text("Logo")
    .font(.custom("BrandFont", fixedSize: 24))
```

## Adaptive Layout — Responding to Size Changes

### UIKit
```swift
// Switch layout at accessibility sizes
override func traitCollectionDidChange(_ previous: UITraitCollection?) {
    super.traitCollectionDidChange(previous)
    if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
        stackView.axis = .vertical
    } else {
        stackView.axis = .horizontal
    }
}

// Modern approach (iOS 17+) — no need to override traitCollectionDidChange
override func updateConfiguration(using state: UICellConfigurationState) {
    super.updateConfiguration(using: state)
    if state.traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
        stackView.axis = .vertical
    } else {
        stackView.axis = .horizontal
    }
}
```

### SwiftUI
```swift
@Environment(\.dynamicTypeSize) var dynamicTypeSize

var body: some View {
    if dynamicTypeSize.isAccessibilitySize {
        // Vertical layout for accessibility sizes
        VStack(alignment: .leading) {
            icon
            textContent
        }
    } else {
        // Horizontal layout for standard sizes
        HStack {
            icon
            textContent
        }
    }
}
```

## Table and Collection View Cells

```swift
// UIKit — self-sizing cells (required for Dynamic Type)
tableView.rowHeight = UITableView.automaticDimension
tableView.estimatedRowHeight = 56

// Collection view — self-sizing with estimated size
let layout = UICollectionViewFlowLayout()
layout.estimatedItemSize = UICollectionViewFlowLayout.automaticSize

// Important: cells must define their height through Auto Layout constraints,
// NOT through explicit frame/height values
```

## Scaling Non-Text Elements

```swift
// UIKit — scale icons, spacing, borders proportionally with text
let borderWidth = UIFontMetrics.default.scaledValue(for: 1)
let iconSize = UIFontMetrics(forTextStyle: .body).scaledValue(for: 24)
let padding = UIFontMetrics.default.scaledValue(for: 16)

// SwiftUI — use ScaledMetric
@ScaledMetric(relativeTo: .body) var iconSize: CGFloat = 24
@ScaledMetric var padding: CGFloat = 16

Image(systemName: "star")
    .frame(width: iconSize, height: iconSize)
    .padding(padding)
```

## Large Content Viewer

For small, fixed-size controls that can't grow (tab bars, toolbars, segmented controls) — show a large preview on long press:

### UIKit
```swift
// Enable Large Content Viewer on a fixed control
button.showsLargeContentViewer = true
button.largeContentTitle = "Nutrition Info"
button.largeContentImage = UIImage(systemName: "leaf")
button.addInteraction(UILargeContentViewerInteraction())

// For a custom view
class CompactToolbarButton: UIButton {
    override init(frame: CGRect) {
        super.init(frame: frame)
        showsLargeContentViewer = true
        addInteraction(UILargeContentViewerInteraction())
    }
}
```

### SwiftUI
```swift
Button(action: { }) {
    Image(systemName: "leaf")
}
.accessibilityShowsLargeContentViewer {
    Label("Nutrition Info", systemImage: "leaf")
}
```

## Bold Text Support

Respect the system Bold Text setting for custom fonts that don't automatically adapt:

Apply custom-font handling inside the design system, not as a feature-level override. If the design system does not support Bold Text or Dynamic Type, report the gap instead of patching one screen around it.

```swift
extension UIFont {
    func accessibleBoldEnough(of size: CGFloat) -> UIFont {
        UIFont(descriptor: accessibleFontDescriptor, size: size)
    }
    var accessibleFontDescriptor: UIFontDescriptor {
        if UIAccessibility.isBoldTextEnabled {
            return fontDescriptor.withSymbolicTraits(.traitBold)!
        } else {
            return fontDescriptor
        }
    }
}

// Listen for bold text changes
NotificationCenter.default.addObserver(
    forName: UIAccessibility.boldTextStatusDidChangeNotification,
    object: nil, queue: .main
) { _ in
    self.updateFonts()
}
```

```swift
// SwiftUI — bold text is handled automatically for system fonts.
// For custom fonts, check the environment:
@Environment(\.legibilityWeight) var legibilityWeight

var body: some View {
    Text("Hello")
        .font(legibilityWeight == .bold ? .custom("MyFont-Bold", size: 17, relativeTo: .body)
                                        : .custom("MyFont-Regular", size: 17, relativeTo: .body))
}
```

## Limiting Dynamic Type (When You Can't Fully Adapt)

Use these as a last resort when full adaptation isn't possible:

```swift
// Fixed size — does not change with Dynamic Type at all
extension UIFont {
    static func preferredFont_fixed(for textStyle: UIFont.TextStyle) -> UIFont {
        let traitCollection = UITraitCollection(preferredContentSizeCategory: .default)
        return UIFont.preferredFont(forTextStyle: textStyle, compatibleWith: traitCollection)
    }
}

// Limited — scales up to a cap, then stops growing
extension UIFont {
    public static func preferredFont_limited(
        forTextStyle textStyle: UIFont.TextStyle,
        by contentSize: UIContentSizeCategory = .accessibilityMedium
    ) -> UIFont {
        if UIContentSizeCategory.current > contentSize {
            let traitCollection = UITraitCollection(preferredContentSizeCategory: contentSize)
            return UIFont.preferredFont(forTextStyle: textStyle, compatibleWith: traitCollection)
        }
        return UIFont.preferredFont(forTextStyle: textStyle)
    }
}
```

```swift
// SwiftUI — limit dynamic type range
Text("Hello")
    .dynamicTypeSize(...DynamicTypeSize.accessibility2)  // cap at AX2

Text("Fixed")
    .dynamicTypeSize(.large)  // pin to one size
```

## Content Size Category Reference

Standard sizes (7): `.extraSmall`, `.small`, `.medium`, `.large` (default), `.extraLarge`, `.extraExtraLarge`, `.extraExtraExtraLarge`

Accessibility sizes (5): `.accessibilityMedium`, `.accessibilityLarge`, `.accessibilityExtraLarge`, `.accessibilityExtraExtraLarge`, `.accessibilityExtraExtraExtraLarge`

```swift
// Check if current size is an accessibility size
if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
    // adapt layout for very large text
}

// SwiftUI
if dynamicTypeSize.isAccessibilitySize { ... }
```

## Testing Dynamic Type

### In Xcode
- **Accessibility Inspector** → Settings → Font size slider — change size without leaving Xcode
- **Xcode previews** — use `.environment(\.sizeCategory, .accessibilityExtraExtraExtraLarge)`
- **Simulator** → Settings → Accessibility → Display & Text Size → Larger Text

### SwiftUI Previews
```swift
#Preview {
    MyView()
        .environment(\.dynamicTypeSize, .accessibility3)
}

// Preview at multiple sizes at once
#Preview {
    ForEach(DynamicTypeSize.allCases, id: \.self) { size in
        MyView()
            .environment(\.dynamicTypeSize, size)
            .previewDisplayName("\(size)")
    }
}
```

### What to verify:
1. **Text is not truncated** — all text remains readable, wraps to multiple lines
2. **No overlapping** — elements don't overlap at the largest sizes
3. **Scrollable** — content scrolls when it exceeds screen height
4. **Layout adapts** — horizontal layouts switch to vertical at accessibility sizes
5. **Touch targets** — buttons and controls remain tappable (minimum 44x44pt)
6. **Images scale** — icons grow proportionally, don't stay tiny next to huge text
7. **No fixed heights** — cells, rows, and containers expand to fit content

## Common Mistakes

- Don't use fixed heights on cells, rows, or containers — they clip text at large sizes
- Don't forget `adjustsFontForContentSizeCategory = true` (UIKit) — without it, text won't update live
- Don't forget `numberOfLines = 0` — single-line labels truncate at large sizes
- Don't use `.isAccessibilityCategory` for hiding content — only use it for layout changes
- Don't scale fonts but forget spacing, icons, and borders — they look disproportionately small
- Don't limit Dynamic Type without a clear reason — users chose their size for a reason
- Don't test only at default size — always verify at the largest accessibility size (AX5)
- Don't use fixed-size custom fonts (`.custom("Font", fixedSize:)`) unless the element truly cannot grow (e.g., a logo)

## When reviewing code for Dynamic Type, check:

1. The project was checked for a design system and typography tokens before fonts were changed
2. Design-system fonts that support Dynamic Type are used through their existing APIs
3. Design-system fonts that lack Dynamic Type support were not bypassed in feature code; the gap is reported as a possible backlog item
4. Projects without a design system use `UIFont.preferredFont(forTextStyle:)` or semantic SwiftUI styles such as `.body`
5. Custom-font scaling code is added only inside a design system whose Dynamic Type support the user explicitly asked to implement
6. `adjustsFontForContentSizeCategory = true` is set on UIKit labels/buttons
7. `numberOfLines = 0` on labels that might need to wrap
8. No fixed heights on cells/containers — use `automaticDimension`
9. Layout switches to vertical at `.isAccessibilityCategory` sizes
10. Non-text elements scale with `UIFontMetrics.scaledValue(for:)` or `@ScaledMetric`
11. Small fixed controls have Large Content Viewer enabled
12. Bold Text setting is respected for custom fonts
13. Screen content is scrollable

$ARGUMENTS
