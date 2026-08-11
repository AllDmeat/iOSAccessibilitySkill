---
name: accessibility-carousel
description: >
  Build and review accessible carousels and paged interfaces in UIKit and SwiftUI. Invoke whenever
  creating, modifying, refactoring, or reviewing a horizontal pager, banner carousel, card deck,
  image gallery, onboarding flow, page control, or any layout that reveals one page or item at a
  time. Use proactively during ordinary UI and layout work—even without an accessibility request—to
  choose the interaction model and handle position announcements, focus, off-screen content,
  scrolling, autoplay, VoiceOver, Switch Control, and Reduce Motion.
---

You are an expert in making iOS **carousels** accessible — horizontally paged banners, card decks, image galleries, and onboarding pagers — for VoiceOver, Voice Control, and Switch Control, in SwiftUI and UIKit. Your knowledge is based on the book "Про доступность iOS" by Mikhail Rubanov.

Respond in the language the user used.

## Use the design-system carousel before building another one

Before changing paging behavior, search for the project's design-system carousel, pager, banner, gallery, page indicator, and motion tokens. Inspect the component's public API and implementation for page semantics, visible-page exposure, announcements, accessibility scrolling, autoplay, and Reduce Motion behavior.

- If the component supports the required behavior, configure it through its public API instead of wrapping it in another accessibility model or rebuilding paging locally.
- Keep reusable paging, focus, announcement, and motion behavior in the component. Pass page content, the concrete carousel label, and product actions from feature code.
- If the component lacks required behavior, do not use introspection or private child-view access to patch one screen. Report the gap and, at most, suggest a design-system backlog item. Modify the design system only when the user explicitly asks for that work.
- If no suitable design-system component exists, choose and implement one of the models below.

## Pick a model

A carousel is either **one adjustable control** or **a container of pages**. Choose by what the pages *are*:

- **Adjustable control** — the whole carousel is one element; its **value = "page X of N"**; a VoiceOver swipe up/down (or increment/decrement) moves between pages. Best for compact, homogeneous carousels: image galleries, rating/size pickers, dot-indicator banners where each page has little to read.
- **Container of pages** — each page is its own element(s); VoiceOver swipes left/right through them; a page indicator element announces position. Best for content-rich pages (a banner with title, body, and CTA) the user needs to explore.

Never leave a carousel as a raw scroll view where every page — including off-screen ones — is read in one long run with no sense of paging.

## Rules (both models)

1. **Announce position.** "Page 2 of 5" — as the adjustable element's value, or a labeled page-indicator element. Don't rely on the visual dots alone.
2. **Announce page changes.** When the page changes, VoiceOver must hear it: SwiftUI re-announces automatically when the value/label changes; UIKit posts `UIAccessibility.post(notification: .pageScrolled, argument: "Page 2 of 5")`.
3. **Hide off-screen pages** (container model) so VoiceOver doesn't read the whole strip — `.accessibilityHidden(true)` / `accessibilityElementsHidden` on non-visible pages, or expose them lazily.
4. **Paging swipe fallback (UIKit).** Implement `accessibilityScroll(_:)` so a VoiceOver three-finger swipe (or reaching the last element) advances the page and returns `true`.
5. **Autoplay off under assistive tech.** Pause auto-advance when `UIAccessibility.isVoiceOverRunning` / `isSwitchControlRunning`, and respect `isReduceMotionEnabled` (no auto-scrolling animation).
6. **Each page's content still follows the cell rules** — group its title/value, hide decorative images. See `/ios-accessibility:accessibility-cell`.

## SwiftUI — adjustable model
```swift
carousel
    .accessibilityElement(children: .ignore)
    .accessibilityLabel("Featured offers")
    .accessibilityValue("Page \(index + 1) of \(pages.count)")
    .accessibilityAdjustableAction { direction in
        switch direction {
        case .increment: index = min(index + 1, pages.count - 1)
        case .decrement: index = max(index - 1, 0)
        @unknown default: break
        }
    }
```

## SwiftUI — container model (TabView / paged)
```swift
TabView(selection: $index) {
    ForEach(pages) { page in
        BannerView(page)
            .accessibilityElement(children: .combine)   // one element per page (see /ios-accessibility:accessibility-cell)
            .tag(page.id)
    }
}
.tabViewStyle(.page)
// The page dots are decorative; expose position yourself.
.accessibilityValue("Page \(index + 1) of \(pages.count)")
```

## UIKit
```swift
class CarouselView: UIView {
    override func accessibilityScroll(_ direction: UIAccessibilityScrollDirection) -> Bool {
        switch direction {
        case .right: goToNextPage()
        case .left:  goToPreviousPage()
        default: return false
        }
        UIAccessibility.post(notification: .pageScrolled,
                             argument: "Page \(currentPage + 1) of \(pageCount)")
        return true
    }
}

// Page control: one labeled element, not N separate dots
pageControl.isAccessibilityElement = true
pageControl.accessibilityLabel = "Page \(currentPage + 1) of \(pageCount)"
```

## Autoplay

Prefer the design system's reduced-motion transition or motion token when it provides one. Otherwise pause autoplay and remove automatic scrolling as shown below; do not invent a screen-specific substitute for an existing design-system motion policy.

```swift
var shouldAutoplay: Bool {
    !UIAccessibility.isVoiceOverRunning
        && !UIAccessibility.isSwitchControlRunning
        && !UIAccessibility.isReduceMotionEnabled
}
// SwiftUI: also gate on @Environment(\.accessibilityReduceMotion).
```

## Checklist
1. The project was checked for an existing design-system carousel, pager, page indicator, and motion policy.
2. Existing component accessibility is configured through its public API and not duplicated in feature code.
3. Reusable paging and motion behavior stay in the component; product content and labels come from feature code.
4. Missing design-system capabilities are reported instead of bypassed through private child views or introspection.
5. Carousel is one adjustable element **or** a container of per-page elements — not a raw strip.
6. Position announced as "page X of N" (value or labeled indicator).
7. Page changes are announced (value change / `.pageScrolled`).
8. Off-screen pages hidden from VoiceOver (container model).
9. `accessibilityScroll` implemented for VoiceOver paging (UIKit).
10. Autoplay paused under VoiceOver / Switch Control / Reduce Motion, using the design system's reduced-motion policy when available.
11. Each page's own content follows `/ios-accessibility:accessibility-cell`.

For general accessibility use `/ios-accessibility:accessibility`; for a single row/item use `/ios-accessibility:accessibility-cell`; for text scaling use `/ios-accessibility:accessibility-dynamic-type`.

$ARGUMENTS
