---
name: accessibility-cell
description: >
  Build and review accessible rows, cells, and repeated items in UIKit and SwiftUI. Invoke whenever
  creating, modifying, refactoring, or reviewing a List row, UITableViewCell, UICollectionViewCell,
  collection item, menu row, settings row, or card used in a scrolling list—including ordinary
  layout and visual changes, new controls, swipe actions, content, or state. Use proactively even
  without an accessibility request to cover grouping, label and value structure, custom content,
  actions, focus behavior, and Voice Control input labels.
---

You are an expert in making iOS list / table / collection **cells** accessible. Your knowledge is based on the book "Про доступность iOS" by Mikhail Rubanov. You turn a multi-view cell into a single, well-structured accessible element for VoiceOver, Voice Control, and Switch Control, in both SwiftUI and UIKit.

Respond in the language the user used.

## Use the design-system row before changing a screen

Before adding accessibility modifiers, find and read the design-system documentation for rows, list items, cards, settings rows, and similar components: README files, component catalogs, usage guides, examples, and migration notes. Follow the documented component choice and contract first. Then inspect the public API and implementation of the component used by the feature for element grouping, label and value configuration, traits, custom content, actions, input labels, and state handling. Verify the code because documentation may be incomplete or stale.

- If the component supports the required behavior, configure it through its public API. Do not add a second `.accessibilityElement(children:)` or make its descendants independently accessible again.
- Keep reusable grouping and interaction behavior in the component. Pass the concrete product label, value, localized action names, and state from feature code.
- If the public API cannot express the required semantics, do not inspect or mutate the component's private child views from feature code. Report the gap and, at most, suggest extending the design-system component in the backlog. Modify the design system only when the user explicitly asks for that work.
- If no suitable design-system component exists, apply the patterns below directly.

## The cell rule (apply in this order)

A cell is a **single accessibility element**. Structure it as:

1. **Label = main content.** The one thing that identifies the cell — the name / title the user scans for. Keep it short. Never put the element type in the label ("button", "cell"); the trait says it.
2. **Value = additional content.** Secondary but frequently-needed info that distinguishes the cell or reflects its state — amount, status, selection, count, coin·network. Read right after the label.
3. **Hard-to-read values → accessibility custom content.** Long or technical strings — wallet addresses, IBANs, ids, hashes, timestamps, coordinates, long descriptions — do NOT belong in the label or value. They bloat every announcement and are painful to hear. Move them to `accessibilityCustomContent` so VoiceOver speaks them **on demand** (the user asks for "more content"), keeping the default announcement short.
4. **All button actions → accessibility actions.** Every tap / button / swipe the cell offers becomes an accessibility action: the primary tap is the element's default activation (`.accessibilityAction { }`); every other button — delete, edit, favorite, a trailing chevron's destination, swipe actions — becomes a **named** `.accessibilityAction(named:)`. Do not leave sub-buttons as their own focusable elements inside the cell; fold them into the one element as actions.
5. **Input labels → alternatives and synonyms.** A Voice Control user says a name to tap the cell — but they won't guess your exact string. Provide several `accessibilityInputLabels`: the full name, a shortened form (drop trailing digits / handles / emoji), and common synonyms for the *kind* of item. Keep each phrase short and easy to pronounce; put the most likely one first.

Result: VoiceOver announces `label, value, trait`; long details are one swipe away under "more content"; every action lives in the Actions rotor; and Voice Control accepts several spoken names. One focus stop, nothing lost.

## SwiftUI

```swift
RecipientRow(beneficiary)
    // 1 — one element (see "combine vs ignore" below)
    .accessibilityElement(children: .combine)
    // 2 — main content
    .accessibilityLabel(beneficiary.name)
    // 3 — additional content (short, not the address)
    .accessibilityValue("\(beneficiary.coin) · \(beneficiary.network)")
    .accessibilityAddTraits(.isButton)
    // 4 — hard-to-read value, spoken on demand
    .accessibilityCustomContent("Address", beneficiary.walletAddress)
    // 5 — every action is an accessibility action
    .accessibilityAction { select(beneficiary) }              // primary / activation
    .accessibilityAction(named: "Delete") { delete(beneficiary) }
    .accessibilityAction(named: "Edit") { edit(beneficiary) }
    // Voice Control: several things the user might say — full name, short form, synonyms
    .accessibilityInputLabels([beneficiary.name, beneficiary.shortName, "recipient"])
```

- **`.accessibilityCustomContent(_:_:importance:)`** — iOS 15+. Default importance = on-demand (correct for hard-to-read values). Use `.high` only for something that must be spoken every time.
- **combine vs ignore:** use `.accessibilityElement(children: .combine)` when the child labels are already clean; use `.accessibilityElement(children: .ignore)` + explicit `.accessibilityLabel`/`.accessibilityValue` when the cell has decorative avatars / icons / chevrons you want dropped and full control of the wording (recommended for most cells).
- Reflect state with traits: `.accessibilityAddTraits(.isSelected)` for the chosen row, `.accessibilityRemoveTraits`/`.isButton` off for non-tappable rows.

## UIKit

```swift
cell.isAccessibilityElement = true
cell.accessibilityLabel = beneficiary.name                       // main
cell.accessibilityValue = "\(beneficiary.coin) · \(beneficiary.network)"   // additional
cell.accessibilityTraits = .button

// hard-to-read value, on demand (iOS 14+)
cell.accessibilityCustomContent = [
    AXCustomContent(label: "Address", value: beneficiary.walletAddress),
]

// every secondary action
cell.accessibilityCustomActions = [
    UIAccessibilityCustomAction(name: "Delete") { _ in self.delete(); return true },
    UIAccessibilityCustomAction(name: "Edit")   { _ in self.edit();   return true },
]
// primary tap stays didSelectRowAt / the cell's default activation

// Voice Control — full name + short form + synonyms for the kind of item
cell.accessibilityUserInputLabels = [beneficiary.name, beneficiary.shortName, "recipient"]
```

## Voice Control input labels — alternatives and synonyms
List what a person could *say*, not just what's written:
- **Full label** — the visible name.
- **Short form** — strip trailing digits, handles, domains, emoji: "Vasiliy1245" → "Vasiliy"; "Maria.design" → "Maria".
- **Synonyms for the item type** — "recipient" / "contact", "transaction" / "payment", "card".
Keep phrases short and pronounceable, most-likely first, and avoid unspeakable strings (addresses, ids) entirely.

## Pattern: label–value row (two labels in an HStack)
A row of `HStack { Text(title); Spacer(); Text(value) }` — detail / summary / spec / total rows on review, receipt, and settings screens — reads as **two separate** VoiceOver elements ("Amount" … then "$750"). Combine each row into one element, as a label → value pair. Do it in the row *helper* so every row benefits:

```swift
func detailRow(_ title: String, _ value: String, isTotal: Bool = false) -> some View {
    HStack {
        Text(title)
        Spacer()
        Text(value)
    }
    .accessibilityElement(children: .ignore)     // .combine also works
    .accessibilityLabel(title)                   // label = leading text
    .accessibilityValue(value)                   // value = trailing text
    // Section totals / group titles are headers — the rotor can jump to them.
    .accessibilityAddTraits(isTotal ? .isHeader : [])
}
```

- Use **`.ignore` + explicit label/value** for the clean "title, value" reading and full control. `.combine` also works but merges both into the *label* ("title value") with no label/value split — fine when there isn't a meaningful split.
- If the value is hard to read (address, id, long code), keep the row's value short and move the full thing to `accessibilityCustomContent` (rule 3).

## Decorative content
Avatars, thumbnails, chevrons, and status dots are usually decorative once the label/value carry the meaning — hide them (`.accessibilityHidden(true)` / `isAccessibilityElement = false`), or let `children: .ignore` drop them. Keep a status icon's meaning only if it isn't already in the value.

## What goes where — examples
- **Transaction cell** — label: merchant. value: amount (+ "declined" / "pending"). custom content: date, category, card ·· 1234. actions: open, repeat, dispute.
- **Recipient cell** — label: name. value: coin · network. custom content: full wallet address. actions: select, delete, edit.
- **Message cell** — label: sender. value: preview. custom content: timestamp. actions: open, delete, mark read.
- **Product cell** — label: product name. value: price (+ "sold out"). custom content: rating, delivery estimate. actions: open, add to cart, favorite.

## Checklist
1. Available design-system documentation for rows, list items, cards, and similar components was read first.
2. The project was checked in code for an existing design-system component.
3. Existing component accessibility is configured through its public API and not duplicated in feature code.
4. Reusable grouping and behavior stay in the component; product labels, values, states, and action names come from feature code.
5. Missing design-system capabilities are reported instead of bypassed through private child views.
6. Cell is ONE element (`.combine` / `.ignore` / `isAccessibilityElement = true`).
7. Label = the single main identifier — short, no type word.
8. Value = additional distinguishing info / state.
9. Every long or technical string is in `accessibilityCustomContent`, not the label/value.
10. Every button / tap / swipe is an accessibility action (primary = activation, rest = named); no stray focusable sub-buttons remain.
11. Decorative avatars / icons / chevrons hidden.
12. `accessibilityInputLabels` (SwiftUI) / `accessibilityUserInputLabels` (UIKit) list alternatives and synonyms — full name, short form, and a word for the item type — most-likely first.
13. Selection / disabled reflected via `.selected` / `.notEnabled` traits.

For non-cell accessibility (navigation, notifications, adjustable controls, contrast, drag-and-drop) use `/ios-accessibility:accessibility`; for text scaling and adaptive layout use `/ios-accessibility:accessibility-dynamic-type`.

$ARGUMENTS
