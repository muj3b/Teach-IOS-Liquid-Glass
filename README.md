# APPLE iOS 26 — LIQUID GLASS: COMPLETE ZERO-TO-MASTERY TEACHING PROMPT
# Copy everything below this line and paste it into any AI coder.
# That AI will know EVERYTHING about Liquid Glass after reading this.

---

You have **zero prior knowledge** of Apple's "Liquid Glass" design system. That is fine. This document will teach you everything — from what it is, to every public API, to every design rule, to how to build completely custom Liquid Glass creations of any kind. After reading this, you can implement any Liquid Glass UI a user requests: standard system components, custom buttons, morphing menus, floating clusters, glass cards, sliders, segmented controls, onboarding flows, dashboards, or anything else. You will also know exactly what NOT to do.

Read every word. This is your complete knowledge base.

---

## ═══════════════════════════════════════════════════
## CHAPTER 1 — WHAT IS LIQUID GLASS? (START HERE)
## ═══════════════════════════════════════════════════

### 1.1 The Simple Explanation

Liquid Glass is Apple's new **material system** introduced at WWDC 2025, shipping in iOS 26, iPadOS 26, macOS Tahoe 26, watchOS 26, tvOS 26, and visionOS 26. It is the biggest visual design change Apple has made since iOS 7 in 2013.

Think of it as a pane of real, physical glass floating above your app's content. It:
- Is **translucent** — content beneath it shows through
- **Refracts** light — it bends and concentrates what's behind it, like real glass does, NOT just blurs it
- **Reflects** the environment — shows specular highlights that shift when you tilt the device
- **Morphs and flows** — when you interact with it, it scales, bounces, and shimmers like a liquid
- **Adapts** — it reads the colors, brightness, and content behind it and adjusts its own appearance to stay legible

It is NOT a simple blur filter. It is NOT frosted glass. It is a live, physics-aware material.

### 1.2 The Core Principle — The Layer Cake Rule

This is the single most important design rule. **Never violate it.**

Imagine every iOS screen as a layer cake:

```
┌─────────────────────────────────────────────────┐
│  LAYER 3: Interactive highlights                │  ← Touch glow, specular shimmer (automatic)
├─────────────────────────────────────────────────┤
│  LAYER 2: Liquid Glass (navigation/controls)    │  ← THIS is where glass lives
│  Tab bars, toolbars, nav bars, FABs, sheets     │
├─────────────────────────────────────────────────┤
│  LAYER 1: Your app content (NO glass here)      │  ← Lists, text, photos, data, media
└─────────────────────────────────────────────────┘
```

**ALWAYS glass:** Tab bars, navigation bars, toolbars, floating action buttons, sheets, popovers, menus, sidebars, search bars, bottom accessories.

**NEVER glass:** List rows, scrolling content, photo grids, data cards, body text regions, primary content of any kind.

**NEVER glass-on-glass:** Do not put a glass element inside another glass element. Glass cannot visually sample through another glass surface — you will get visual noise, mud, and performance problems.

### 1.3 Physical Properties of the Material

When Apple's engineers and designers describe Liquid Glass, these are the physical properties they mean:

| Property | What it means | Visual effect |
|----------|---------------|---------------|
| **Lensing** | Bends/concentrates light from behind (NOT scatter/blur) | Content behind appears magnified or shifted near edges |
| **Refraction** | Light path changes as it passes through | Background appears displaced near element boundaries |
| **Specular highlights** | Mirror-like surface reflections | A bright rim or glint that moves when you tilt the device |
| **Materialization** | Appears by modulating lensing strength, not fading | Elements "grow from light" rather than fade in |
| **Fluidity** | Gel-like physical response | Scale-bounce on press, elastic morph between states |
| **Adaptivity** | Samples surroundings and adjusts | Automatically stays legible on any background |

### 1.4 The Three Glass Variants

There are exactly three public glass variants. Use only these. Do not invent others.

```
Glass.regular   →  Default. Medium translucency. Fully adaptive. Use this 90% of the time.
Glass.clear     →  More transparent. Less adaptive. USE WITH CAUTION (strict rules apply).
Glass.identity  →  No effect at all. Keeps layout stable. Use for conditional toggling.
```

**When to use `.clear`:** ALL three conditions must be met simultaneously:
1. The glass element sits over media-rich content (a photo, video, or map — NOT white space)
2. A darkening/dimming layer behind the glass will NOT hurt the content beneath
3. The foreground content ON TOP of the glass (text, icon) is bold and bright

If any condition is false, use `.regular` instead.

**When to use `.identity`:** When you need to conditionally disable glass without changing layout. Example: when `accessibilityReduceTransparency` is enabled, swap `.regular` for `.identity`.

---

## ═══════════════════════════════════════════════════
## CHAPTER 2 — SETUP & REQUIREMENTS
## ═══════════════════════════════════════════════════

### 2.1 Requirements

- **Xcode 26** (mandatory — the Liquid Glass APIs will not compile in older Xcode)
- **iOS 26.0+** deployment target for full features
- **Swift 6** (or later)
- Physical device with A15 Bionic or newer recommended for true visual accuracy (Simulator approximates)

### 2.2 Imports

```swift
import SwiftUI   // For all SwiftUI glass APIs
import UIKit     // For UIKit glass APIs
```

No third-party packages needed. Everything is in the standard SDK.

### 2.3 Availability Guarding (Critical for Apps Supporting Older iOS)

Always guard glass code when your app targets iOS older than 26:

```swift
// ✅ CORRECT — Runtime guard
if #available(iOS 26.0, *) {
    // glass code here
}

// ✅ CORRECT — Compile-time guard
@available(iOS 26.0, *)
struct MyGlassView: View { ... }

// ✅ CORRECT — ViewBuilder guard
@ViewBuilder
func adaptiveGlass() -> some View {
    if #available(iOS 26.0, *) {
        myView.glassEffect()
    } else {
        myView.background(.ultraThinMaterial).clipShape(Capsule())
    }
}
```

### 2.4 Automatic Adoption (What You Get for Free)

When you recompile ANY existing app with Xcode 26 targeting iOS 26, these components **automatically** receive Liquid Glass with zero code changes:
- `TabView` tab bars
- `NavigationStack` / `NavigationSplitView` navigation bars
- Toolbars
- `.sheet()` presentations
- Popovers, menus, alerts, context menus
- `UINavigationBar`, `UITabBar`, `UIToolbar`, `UISheetPresentationController`
- System controls (buttons, sliders, steppers, segmented controls)

After this automatic adoption, your job is:
1. **Remove** any custom backgrounds you added behind bars/sheets (they'll fight the glass)
2. **Audit** each screen for anything that looks wrong
3. **Add** custom glass only where needed for your unique UI

---

## ═══════════════════════════════════════════════════
## CHAPTER 3 — THE CORE SwiftUI API: `glassEffect()`
## ═══════════════════════════════════════════════════

### 3.1 Full Function Signature

```swift
func glassEffect<S: Shape>(
    _ glass: Glass = .regular,
    in shape: S = DefaultGlassEffectShape,   // Default shape is Capsule
    isEnabled: Bool = true
) -> some View
```

### 3.2 The `Glass` Type — Full API

```swift
struct Glass {
    // The three variants
    static var regular: Glass
    static var clear: Glass
    static var identity: Glass

    // Chainable modifiers
    func tint(_ color: Color) -> Glass         // Add color emphasis
    func interactive() -> Glass                // Enable touch behaviors (iOS only)
}
```

**Chaining is order-independent:**
```swift
.glassEffect(.regular.tint(.blue).interactive())
// Same as:
.glassEffect(.regular.interactive().tint(.blue))
```

### 3.3 Basic Usage — Every Form

```swift
// Absolute simplest — defaults to .regular, Capsule shape
Text("Hello")
    .padding()
    .glassEffect()

// Explicit variant
Text("Hello")
    .padding()
    .glassEffect(.regular)

// Explicit variant + shape
Text("Hello")
    .padding()
    .glassEffect(.regular, in: .capsule)

// With tint
Text("Hello")
    .padding()
    .glassEffect(.regular.tint(.blue))

// With tint + opacity
Text("Hello")
    .padding()
    .glassEffect(.regular.tint(.blue.opacity(0.6)))

// With interactive touch behavior
Button("Tap Me") { }
    .glassEffect(.regular.interactive())

// Fully loaded
Button("Full") { }
    .glassEffect(.regular.tint(.purple.opacity(0.7)).interactive())

// Conditional enable/disable
Button("Toggle") { }
    .glassEffect(.regular, isEnabled: someCondition)

// Clear variant (use sparingly, rules above apply)
Button("Over Photo") { }
    .glassEffect(.clear)

// Identity — no effect, stable layout
Button("No Glass") { }
    .glassEffect(.identity)
```

### 3.4 The `.tint()` Modifier — Rules

Tinting means adding a color wash to the glass. Use it to communicate **meaning**, NOT decoration.

```swift
// ✅ CORRECT uses of tint:
.glassEffect(.regular.tint(.blue))       // Primary action
.glassEffect(.regular.tint(.red))        // Destructive action
.glassEffect(.regular.tint(.green))      // Success / confirmation
.glassEffect(.regular.tint(.orange))     // Warning

// ✅ With opacity for subtlety:
.glassEffect(.regular.tint(.purple.opacity(0.5)))

// ❌ WRONG — Don't tint everything, don't tint for decoration:
// Every element tinted = no hierarchy, defeats the purpose
```

### 3.5 The `.interactive()` Modifier — What It Does

When you add `.interactive()`, the glass element automatically gains:
- **Press scale** — scales down slightly on touch
- **Bounce release** — springs back with elastic animation
- **Touch-point illumination** — glow begins under the finger on press
- **Propagation** — that glow radiates to nearby glass elements in the same container
- **Shimmer** — a subtle shimmer on interaction
- **Drag response** — responds to drag gestures

Only use on elements the user is supposed to tap or drag. Don't put it on decorative glass.

### 3.6 Shapes — Every Available Option

```swift
// Default capsule (pill shape) — most common for controls
.glassEffect(.regular, in: .capsule)

// Perfect circle — great for icon buttons
.glassEffect(.regular, in: .circle)

// Ellipse
.glassEffect(.regular, in: .ellipse)

// Rectangle with fixed corner radius
.glassEffect(.regular, in: RoundedRectangle(cornerRadius: 16))
.glassEffect(.regular, in: RoundedRectangle(cornerRadius: 12, style: .continuous))

// Rect shorthand
.glassEffect(.regular, in: .rect(cornerRadius: 12))

// Container-concentric — IMPORTANT: automatically matches the corner radius
// of whatever container/window/sheet this element lives inside.
// Use this for elements near the edges of sheets or cards.
.glassEffect(.regular, in: .rect(cornerRadius: .containerConcentric))

// Full rectangle (no rounding)
.glassEffect(.regular, in: .rect)

// Custom shape — any type conforming to the Shape protocol
struct HexagonShape: Shape {
    func path(in rect: CGRect) -> Path {
        var path = Path()
        let center = CGPoint(x: rect.midX, y: rect.midY)
        let radius = min(rect.width, rect.height) / 2
        for i in 0..<6 {
            let angle = Double(i) * 60.0 * .pi / 180.0
            let point = CGPoint(
                x: center.x + radius * cos(angle),
                y: center.y + radius * sin(angle)
            )
            if i == 0 { path.move(to: point) }
            else { path.addLine(to: point) }
        }
        path.closeSubpath()
        return path
    }
}

.glassEffect(.regular, in: HexagonShape())
```

### 3.7 THE GOLDEN RULE: Modifier Ordering

`glassEffect()` must be the **LAST** modifier in your chain that affects visual appearance. Apply everything else (padding, frame, font, foregroundStyle) before it.

```swift
// ✅ CORRECT — glass is last
Text("Label")
    .font(.headline)
    .foregroundStyle(.white)
    .padding(.horizontal, 16)
    .padding(.vertical, 10)
    .glassEffect()

// ❌ WRONG — modifiers after glass break rendering
Text("Label")
    .glassEffect()
    .padding()        // This breaks the glass effect
    .background(.red) // This especially breaks it
```

### 3.8 What NEVER to Do with glassEffect

```swift
// ❌ NEVER add .background() alongside glassEffect
.background(Color.white)   // Kills transparency entirely
.background(.ultraThinMaterial) // Redundant and breaks glass

// ❌ NEVER add .opacity() to a glass view
.opacity(0.7)  // Interferes with the material system

// ❌ NEVER add .blur() to a glass view
.blur(radius: 5)  // Creates double-blur, destroys lensing

// ❌ NEVER clip a glass view separately
.clipShape(Circle())  // Use the `in:` parameter instead

// ❌ NEVER stack glass inside glass without GlassEffectContainer
// (explained in Chapter 4)
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 4 — GlassEffectContainer (CRITICAL)
## ═══════════════════════════════════════════════════

### 4.1 Why This Container Exists (The Glass-on-Glass Problem)

Glass cannot visually sample through another glass surface. If you place two `.glassEffect()` views next to each other without a container, they will:
- Look visually inconsistent
- Not blend/merge when close together
- Cause double-sampling artifacts
- Perform worse (two separate GPU sampling passes)

`GlassEffectContainer` solves this by giving ALL contained glass elements a **single shared sampling region**. They all read from the same background snapshot, blend correctly, and can morph together.

**Rule: Whenever you have 2 or more glass elements near each other, wrap them in a GlassEffectContainer.**

### 4.2 The Full API

```swift
struct GlassEffectContainer<Content: View>: View {
    // No spacing parameter — uses system default merge threshold
    init(@ViewBuilder content: () -> Content)

    // With spacing — controls the point distance at which elements visually merge
    init(spacing: CGFloat? = nil, @ViewBuilder content: () -> Content)
}
```

**The `spacing` parameter:** When two glass elements are within `spacing` points of each other, they visually merge like water droplets — their outlines dissolve into a single connected blob. Set this based on how "connected" you want nearby elements to feel.

### 4.3 Basic Container Usage

```swift
// Three tool buttons that share sampling and can merge
GlassEffectContainer {
    HStack(spacing: 16) {
        Image(systemName: "pencil")
            .frame(width: 44, height: 44)
            .glassEffect(.regular.interactive())
        
        Image(systemName: "eraser")
            .frame(width: 44, height: 44)
            .glassEffect(.regular.interactive())
        
        Image(systemName: "crop")
            .frame(width: 44, height: 44)
            .glassEffect(.regular.interactive())
    }
}

// With custom merge threshold
GlassEffectContainer(spacing: 24) {
    HStack(spacing: 12) {
        Button("Cancel") { }
            .glassEffect()
        Button("Save") { }
            .glassEffect(.regular.tint(.blue))
    }
}
```

### 4.4 glassEffectUnion — Making Distant Elements Appear as One

For elements that are visually separate but should look like one connected glass shape (like Apple Maps' +/- zoom controls):

```swift
@Namespace private var ns

VStack(spacing: 0) {
    Button {
        // zoom in
    } label: {
        Image(systemName: "plus")
            .frame(width: 44, height: 44)
    }
    .glassEffect(.regular)
    .glassEffectUnion(id: "zoomControl", namespace: ns)

    Divider().frame(height: 0.5).opacity(0.3)

    Button {
        // zoom out
    } label: {
        Image(systemName: "minus")
            .frame(width: 44, height: 44)
    }
    .glassEffect(.regular)
    .glassEffectUnion(id: "zoomControl", namespace: ns)
}
// Both buttons now render as one single glass shape
```

**Full signature:**
```swift
func glassEffectUnion<ID: Hashable>(
    id: ID,
    namespace: Namespace.ID
) -> some View
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 5 — MORPHING: glassEffectID (THE MAGIC)
## ═══════════════════════════════════════════════════

### 5.1 What Morphing Is

Morphing is the signature Liquid Glass animation: when one glass element transforms into another, or when elements appear/disappear, the glass **flows and reforms** like liquid rather than crossfading or popping. A single pill might split into three, or three might collapse back into one.

### 5.2 The Three Requirements for Morphing

All three must be present simultaneously:

1. All elements must be inside the **same `GlassEffectContainer`**
2. Each element must have `.glassEffectID()` with a **shared `@Namespace`**
3. State changes must be wrapped in **`withAnimation(.bouncy)`** or similar spring animation

### 5.3 The glassEffectID API

```swift
func glassEffectID<ID: Hashable>(
    _ id: ID,
    in namespace: Namespace.ID
) -> some View
```

### 5.4 Basic Morphing — Expand/Collapse Pattern

```swift
struct MorphingToolbar: View {
    @State private var isExpanded = false
    @Namespace private var namespace   // ← Required

    var body: some View {
        GlassEffectContainer(spacing: 20) {   // ← Required
            HStack(spacing: 12) {
                // These only appear when expanded
                if isExpanded {
                    Button("Edit") { }
                        .glassEffect()
                        .glassEffectID("edit", in: namespace)   // ← Required

                    Button("Share") { }
                        .glassEffect()
                        .glassEffectID("share", in: namespace)

                    Button("Delete") { }
                        .glassEffect(.regular.tint(.red))
                        .glassEffectID("delete", in: namespace)
                }

                // This is always visible — it morphs as others appear/disappear
                Button {
                    withAnimation(.bouncy) {   // ← Required for fluid motion
                        isExpanded.toggle()
                    }
                } label: {
                    Image(systemName: isExpanded ? "xmark" : "ellipsis")
                        .frame(width: 44, height: 44)
                }
                .glassEffect(.regular.interactive())
                .glassEffectID("toggle", in: namespace)
            }
        }
    }
}
```

### 5.5 Advanced Morphing — Cross-Directional Photo Tools

```swift
struct PhotoEditingCluster: View {
    @State private var isOpen = false
    @Namespace private var ns

    var body: some View {
        GlassEffectContainer(spacing: 28) {
            VStack(spacing: 28) {
                // Top tool — appears above
                if isOpen {
                    toolButton("rotate.right")
                        .glassEffectID("rotate", in: ns)
                }

                HStack(spacing: 28) {
                    // Left tool
                    if isOpen {
                        toolButton("circle.lefthalf.filled")
                            .glassEffectID("contrast", in: ns)
                    }

                    // Center toggle — always visible
                    // When others appear, this morphs to accommodate
                    Button {
                        withAnimation(.bouncy(duration: 0.5)) {
                            isOpen.toggle()
                        }
                    } label: {
                        Image(systemName: isOpen ? "xmark" : "slider.horizontal.3")
                            .frame(width: 48, height: 48)
                            .contentTransition(.symbolEffect(.replace))
                    }
                    .buttonStyle(.glass)
                    .buttonBorderShape(.circle)
                    .glassEffectID("toggle", in: ns)

                    // Right tool
                    if isOpen {
                        toolButton("flip.horizontal")
                            .glassEffectID("flip", in: ns)
                    }
                }

                // Bottom tool
                if isOpen {
                    toolButton("crop")
                        .glassEffectID("crop", in: ns)
                }
            }
        }
    }

    @ViewBuilder
    func toolButton(_ icon: String) -> some View {
        Button { } label: {
            Image(systemName: icon)
                .frame(width: 44, height: 44)
        }
        .buttonStyle(.glass)
        .buttonBorderShape(.circle)
    }
}
```

### 5.6 glassEffectTransition — Controlling How Elements Appear/Disappear

Controls the appearance animation when glass elements enter or leave the view hierarchy:

```swift
func glassEffectTransition(
    _ transition: GlassEffectTransition,
    isEnabled: Bool = true
) -> some View

// Available transition types:
GlassEffectTransition.identity          // Instant — no animation
GlassEffectTransition.matchedGeometry  // Shape morphs from previous state (DEFAULT)
GlassEffectTransition.materialize      // Glass materializes from pure light (preferred for appear/disappear)
```

```swift
// Element materializes when it appears (grows from light)
Button("Action") { }
    .glassEffect()
    .glassEffectID("action", in: namespace)
    .glassEffectTransition(.materialize)

// Element uses matched geometry (smooth positional morph)
Button("Move") { }
    .glassEffect()
    .glassEffectID("move", in: namespace)
    .glassEffectTransition(.matchedGeometry)
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 6 — glassBackgroundEffect
## ═══════════════════════════════════════════════════

For applying a Liquid Glass background to an **entire view or container** (panels, sheets, floating cards):

```swift
func glassBackgroundEffect(
    in shape: some Shape = .rect,
    displayMode: GlassDisplayMode = .always
) -> some View

// Display modes:
GlassDisplayMode.always        // Always render glass background
GlassDisplayMode.whenScrolled  // Only appear when content has scrolled underneath
```

```swift
// Floating info panel with full glass background
VStack(alignment: .leading, spacing: 8) {
    Text("Now Playing").font(.headline)
    Text("Song Title").font(.subheadline)
}
.padding()
.glassBackgroundEffect(in: RoundedRectangle(cornerRadius: 20))

// Navigation bar that only shows glass when content scrolls under
NavigationStack {
    ScrollView { ContentView() }
}
// The nav bar glass appears automatically via .whenScrolled behavior

// Custom floating card
HStack {
    Image(systemName: "location.fill")
    Text("Current Location")
    Spacer()
    Text("2.3 mi")
}
.padding()
.glassBackgroundEffect(in: Capsule())
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 7 — BUTTON STYLES
## ═══════════════════════════════════════════════════

### 7.1 The Two Glass Button Styles

```swift
// .glass — translucent, background shows through
// Use for: secondary actions, tool buttons, cancel actions
Button("Cancel") { }
    .buttonStyle(.glass)

// .glassProminent — more opaque, primary action standout
// Use for: primary CTAs, confirmation actions, important actions
Button("Confirm") { }
    .buttonStyle(.glassProminent)
    .tint(.blue)
```

### 7.2 Full Customization

```swift
// Complete customization
Button("Primary Action") { }
    .buttonStyle(.glassProminent)
    .tint(.purple)
    .controlSize(.large)
    .buttonBorderShape(.capsule)

// Small circular tool button
Button("Tool") { }
    .buttonStyle(.glass)
    .buttonBorderShape(.circle)
    .controlSize(.regular)
    .tint(.orange)

// Large extra prominent CTA
Button("Get Started") { }
    .buttonStyle(.glassProminent)
    .tint(.blue)
    .controlSize(.extraLarge)
    .buttonBorderShape(.capsule)
```

### 7.3 All Control Sizes

```swift
.controlSize(.mini)
.controlSize(.small)
.controlSize(.regular)       // Default
.controlSize(.large)
.controlSize(.extraLarge)    // New in iOS 26
```

### 7.4 All Border Shapes

```swift
.buttonBorderShape(.capsule)                        // Pill shape (default)
.buttonBorderShape(.circle)                         // Perfect circle
.buttonBorderShape(.roundedRectangle(radius: 12))   // Custom corner radius
.buttonBorderShape(.automatic)                      // System decides
```

### 7.5 Known Issue + Fix

`.glassProminent` with `.circle` border shape has rendering artifacts in some OS versions:

```swift
// ❌ Has artifacts
Button("Icon") { }
    .buttonStyle(.glassProminent)
    .buttonBorderShape(.circle)

// ✅ Fixed with clipShape
Button("Icon") { }
    .buttonStyle(.glassProminent)
    .buttonBorderShape(.circle)
    .clipShape(Circle())   // This fixes the artifact
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 8 — TEXT & ICONS ON GLASS
## ═══════════════════════════════════════════════════

Text and icons placed over or inside a glass view receive automatic **vibrant treatment** — the system analyzes the background and adjusts the foreground color's brightness and saturation for maximum legibility.

```swift
// Text on glass — use .white, system adjusts automatically
Text("Glass Label")
    .font(.headline.bold())
    .foregroundStyle(.white)
    .padding()
    .glassEffect()

// SF Symbol icon button
Button { } label: {
    Image(systemName: "heart.fill")
        .font(.title2)
        .foregroundStyle(.white)
        .frame(width: 52, height: 52)
}
.glassEffect(.regular.interactive())

// Icon + text label
Label("Settings", systemImage: "gear")
    .font(.body.bold())
    .foregroundStyle(.white)
    .padding(.horizontal, 16)
    .padding(.vertical, 10)
    .glassEffect()

// Symbol content transitions (use inside glass for state changes)
Image(systemName: isPlaying ? "pause.fill" : "play.fill")
    .contentTransition(.symbolEffect(.replace))   // Smooth icon swap
    .frame(width: 44, height: 44)
    .glassEffect(.regular.interactive())
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 9 — TOOLBAR INTEGRATION
## ═══════════════════════════════════════════════════

### 9.1 Automatic Glass (Zero Code)

Any toolbar compiled with Xcode 26 targeting iOS 26 automatically becomes a floating Liquid Glass toolbar. Nothing extra needed for the glass itself.

```swift
NavigationStack {
    ContentView()
        .toolbar {
            ToolbarItem(placement: .cancellationAction) {
                Button("Cancel", systemImage: "xmark") { }
                // Automatically glass
            }

            ToolbarItem(placement: .confirmationAction) {
                Button("Done", systemImage: "checkmark") { }
                // Automatically glass AND automatically gets .glassProminent treatment
            }
        }
}
```

### 9.2 ToolbarSpacer — Grouping Controls (New in iOS 26)

```swift
.toolbar {
    ToolbarItemGroup(placement: .topBarTrailing) {
        Button("Draw", systemImage: "pencil") { }
        Button("Erase", systemImage: "eraser") { }
    }

    // Fixed gap — creates visual separation between groups
    ToolbarSpacer(.fixed, spacing: 16)

    ToolbarItem(placement: .topBarTrailing) {
        Button("Done", systemImage: "checkmark") { }
            .buttonStyle(.glassProminent)
    }
}

// Flexible spacer — pushes groups to opposite ends
ToolbarSpacer(.flexible)
```

### 9.3 Toolbar Badges

```swift
ToolbarItem(placement: .topBarLeading) {
    Button("Alerts", systemImage: "bell") { }
        .badge(3)
        .tint(.red)
}
```

### 9.4 Shared Background Visibility

```swift
// This item won't share the glass background with neighboring items
ToolbarItem {
    Button("Profile", systemImage: "person.circle") { }
        .sharedBackgroundVisibility(.hidden)
}
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 10 — TAB VIEW
## ═══════════════════════════════════════════════════

### 10.1 Automatic Glass Tab Bar

```swift
TabView {
    Tab("Home", systemImage: "house") { HomeView() }
    Tab("Explore", systemImage: "safari") { ExploreView() }
    Tab("Profile", systemImage: "person") { ProfileView() }
}
// Tab bar automatically gets floating Liquid Glass — zero extra code
```

### 10.2 Tab Bar Minimize on Scroll

```swift
TabView {
    // tabs
}
.tabBarMinimizeBehavior(.onScrollDown)  // Shrinks when scrolling down, expands up

// All options:
.tabBarMinimizeBehavior(.automatic)    // System decides
.tabBarMinimizeBehavior(.onScrollDown) // Shrinks while scrolling down
.tabBarMinimizeBehavior(.never)        // Always full-size
```

### 10.3 Search Tab Role (Floating Search Button)

Adds a special floating search button integrated with the tab bar:

```swift
TabView {
    Tab("Home", systemImage: "house") { HomeView() }

    // Creates a floating search button in the tab bar area
    Tab("Search", systemImage: "magnifyingglass", role: .search) {
        NavigationStack { SearchResultsView() }
    }
}
.searchable(text: $searchQuery)
```

### 10.4 Tab View Bottom Accessory (Mini Player Bar)

A persistent glass panel anchored above the tab bar — perfect for music players, active sessions, etc.:

```swift
TabView {
    // tabs
}
.tabViewBottomAccessory {
    HStack(spacing: 12) {
        Image(systemName: "music.note")
        Text("Now Playing — Song Name")
            .lineLimit(1)
        Spacer()
        Button("", systemImage: "play.fill") { }
        Button("", systemImage: "forward.fill") { }
    }
    .padding(.horizontal, 16)
}
```

**Adapting the accessory when it collapses (inline) with the tab bar:**

```swift
struct AdaptiveNowPlaying: View {
    @Environment(\.tabViewBottomAccessoryPlacement) var placement
    // placement is .expanded (full height) or .collapsed (inline with minimized tab bar)

    var body: some View {
        if placement == .expanded {
            // Full mini player with artwork, controls, etc.
            HStack {
                Image(systemName: "music.note")
                Text("Song Title")
                Spacer()
                Button("", systemImage: "play.fill") { }
                Button("", systemImage: "forward.fill") { }
            }
        } else {
            // Compact version that fits inline
            HStack {
                Text("Song Title")
                Spacer()
            }
        }
    }
}
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 11 — SHEET PRESENTATIONS
## ═══════════════════════════════════════════════════

### 11.1 Automatic Glass Sheets

Sheets in iOS 26 automatically get an inset Liquid Glass background. Do NOT set a custom `presentationBackground` — it overrides the glass.

```swift
.sheet(isPresented: $showSheet) {
    SheetContent()
        .presentationDetents([.medium, .large])
    // Glass background applied automatically — don't set presentationBackground
}
```

### 11.2 Sheet Morphing from a Toolbar Button (Zoom Transition)

A sheet can appear to morph out of a toolbar button using matched transitions:

```swift
struct ContentWithMorphSheet: View {
    @Namespace private var transition
    @State private var showInfo = false

    var body: some View {
        NavigationStack {
            MainContent()
                .toolbar {
                    ToolbarItem(placement: .topBarTrailing) {
                        Button("Info", systemImage: "info.circle") {
                            showInfo = true
                        }
                        .matchedTransitionSource(id: "info", in: transition)
                    }
                }
                .sheet(isPresented: $showInfo) {
                    InfoSheet()
                        .navigationTransition(.zoom(sourceID: "info", in: transition))
                }
        }
    }
}
```

### 11.3 Removing Conflicting Backgrounds Inside Sheets

When you have `Form` or `List` inside a sheet, their default backgrounds can conflict with glass:

```swift
.sheet(isPresented: $show) {
    Form {
        Section("Settings") {
            Toggle("Enable", isOn: $setting)
        }
    }
    .scrollContentBackground(.hidden)           // Remove Form/List default background
    .containerBackground(.clear, for: .navigation) // Remove navigation container background
}
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 12 — NAVIGATION SPLIT VIEW & SIDEBARS
## ═══════════════════════════════════════════════════

```swift
NavigationSplitView {
    List(items, id: \.id) { item in
        NavigationLink(item.title, value: item)
    }
    .navigationTitle("Library")
} detail: {
    DetailView()
}
// Sidebar automatically floats with glass refraction over content
// Extends background behind the sidebar automatically
// Zero extra code needed for glass sidebar behavior
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 13 — UIKit IMPLEMENTATION (Complete)
## ═══════════════════════════════════════════════════

For UIKit-based apps or hybrid apps, use `UIGlassEffect` and `UIGlassContainerEffect`.

### 13.1 UIGlassEffect — Basic Application

```swift
import UIKit

if #available(iOS 26.0, *) {
    // Step 1: Create the effect
    let glassEffect = UIGlassEffect()

    // Step 2: Apply via UIVisualEffectView
    let effectView = UIVisualEffectView(effect: glassEffect)
    effectView.frame = CGRect(x: 20, y: 100, width: 200, height: 50)

    // Step 3: Add content INSIDE contentView (not on effectView itself)
    let label = UILabel()
    label.text = "Glass Label"
    label.textAlignment = .center
    label.frame = effectView.contentView.bounds
    label.autoresizingMask = [.flexibleWidth, .flexibleHeight]
    effectView.contentView.addSubview(label)

    view.addSubview(effectView)
}
```

### 13.2 UIGlassEffect — All Properties

```swift
if #available(iOS 26.0, *) {
    let glass = UIGlassEffect()

    // Tinting — same semantic rules as SwiftUI
    glass.tintColor = UIColor.systemBlue.withAlphaComponent(0.5)

    // Interactive — enables scale/bounce/shimmer on touch
    glass.isInteractive = true

    // Apply
    let effectView = UIVisualEffectView(effect: glass)
    view.addSubview(effectView)
}
```

### 13.3 UIKit — Shape / Corner Configuration

```swift
if #available(iOS 26.0, *) {
    let effectView = UIVisualEffectView(effect: UIGlassEffect())
    effectView.frame = CGRect(x: 20, y: 200, width: 240, height: 56)

    // Fixed corner radius
    effectView.cornerConfiguration = UIViewCornerConfiguration(
        corners: .allCorners,
        cornerRadius: 28   // Capsule-like for 56pt height
    )

    // Container-relative (adapts to parent container's corners — like SwiftUI's .containerConcentric)
    effectView.cornerConfiguration = UIViewCornerConfiguration(
        corners: .allCorners,
        cornerRadius: .containerRelative
    )

    view.addSubview(effectView)
}
```

### 13.4 UIKit — CRITICAL: How to Animate Glass In/Out

**NEVER use `.alpha` to show/hide glass.** Always animate the `effect` property:

```swift
if #available(iOS 26.0, *) {
    let effectView = UIVisualEffectView()
    view.addSubview(effectView)

    // ✅ CORRECT — Materializes with proper glass animation
    UIView.animate(withDuration: 0.4) {
        effectView.effect = UIGlassEffect()
    }

    // ✅ CORRECT — Dematerializes with proper glass animation
    UIView.animate(withDuration: 0.4) {
        effectView.effect = nil
    }

    // ❌ WRONG — Skips the materialize animation, looks wrong
    effectView.alpha = 0
    UIView.animate(withDuration: 0.4) { effectView.alpha = 1 }
}
```

### 13.5 UIGlassContainerEffect — Multiple Glass Elements

```swift
if #available(iOS 26.0, *) {
    // The container provides the shared sampling region
    let containerEffect = UIGlassContainerEffect()
    containerEffect.spacing = 12  // Elements within 12pt will merge visually

    let containerView = UIVisualEffectView(effect: containerEffect)
    containerView.frame = CGRect(x: 20, y: 300, width: 320, height: 72)
    view.addSubview(containerView)

    // Add individual glass elements INSIDE the container's contentView
    let glass1 = UIVisualEffectView(effect: UIGlassEffect())
    glass1.frame = CGRect(x: 8, y: 14, width: 140, height: 44)
    glass1.cornerConfiguration = UIViewCornerConfiguration(
        corners: .allCorners, cornerRadius: 22
    )
    containerView.contentView.addSubview(glass1)

    let glass2 = UIVisualEffectView(effect: UIGlassEffect())
    glass2.frame = CGRect(x: 160, y: 14, width: 140, height: 44)
    glass2.cornerConfiguration = UIViewCornerConfiguration(
        corners: .allCorners, cornerRadius: 22
    )
    containerView.contentView.addSubview(glass2)
}
```

### 13.6 UIKit — Tab Bar Minimize + Bottom Accessory

```swift
if #available(iOS 26.0, *) {
    // Minimize tab bar on scroll
    tabBarController.tabBarMinimizeBehavior = .onScrollDown

    // Add a Now Playing accessory above the tab bar
    let miniPlayerView = NowPlayingMiniPlayerView()
    let accessory = UITabAccessory(contentView: miniPlayerView)
    tabBarController.bottomAccessory = accessory
}
```

### 13.7 UIKit — Adapting Accessory to Inline State

```swift
final class NowPlayingMiniPlayerView: UIView {
    private let titleLabel = UILabel()
    private let fullControls = UIStackView()

    // iOS 26 recommended: use updateProperties() for trait-based updates
    override func updateProperties() {
        super.updateProperties()
        if #available(iOS 26.0, *) {
            let isInline = traitCollection.tabAccessoryEnvironment == .inline
            fullControls.isHidden = isInline
            titleLabel.font = isInline
                ? .preferredFont(forTextStyle: .caption1)
                : .preferredFont(forTextStyle: .headline)
        }
    }
}
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 14 — ACCESSIBILITY (Automatic + Manual)
## ═══════════════════════════════════════════════════

### 14.1 What Happens Automatically

The glass material system automatically responds to these iOS Accessibility settings:

| Setting | What Glass Does |
|---------|----------------|
| Reduce Transparency | Becomes more opaque/frosted, less see-through |
| Increase Contrast | Adds stark borders, higher contrast colors |
| Reduce Motion | Disables elastic bounce, morphing animations, shimmer |
| iOS 26.1 Tinted Mode | User-controlled opacity increase in settings |

**You do not need to write code for these.** The system handles it.

### 14.2 Manual Accessibility Overrides (When Needed)

```swift
@Environment(\.accessibilityReduceTransparency) var reduceTransparency
@Environment(\.accessibilityReduceMotion) var reduceMotion

var body: some View {
    Button("Action") { }
        // Swap to .identity if transparency is reduced
        .glassEffect(reduceTransparency ? .identity : .regular)

    Button("Animated") {
        // Only use spring animation if motion is OK
        if reduceMotion {
            isExpanded.toggle()
        } else {
            withAnimation(.bouncy) { isExpanded.toggle() }
        }
    }
    .glassEffect(.regular.interactive())
}
```

### 14.3 VoiceOver — Glass Elements Need Accessibility Labels

```swift
// Glass view (not a button) needs a label
Image(systemName: "heart.fill")
    .frame(width: 44, height: 44)
    .glassEffect(.regular.interactive())
    .accessibilityLabel("Like")
    .accessibilityAddTraits(.isButton)
    .accessibilityHint("Double-tap to like this post")

// Custom glass container needs containment
GlassEffectContainer {
    HStack { /* controls */ }
}
.accessibilityElement(children: .contain)
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 15 — PERFORMANCE RULES
## ═══════════════════════════════════════════════════

Liquid Glass is GPU-intensive because of real-time lensing and specular computation. Follow these rules:

1. **Always use `GlassEffectContainer`** for groups of 2+ glass elements. It cuts GPU sampling calls dramatically by sharing one background region.

2. **Keep glass surfaces small.** Navigation bars, tool clusters, floating buttons — these are small. Don't glass an entire scrollable page.

3. **Never continuously animate glass.** Short event-driven transitions (appear, expand, collapse) are fine. Constant rotation or pulsing on a glass surface will cause heat and battery drain.

4. **Don't stack glass layers.** Each glass layer = another GPU pass. Keep it to one glass layer per screen region.

5. **Use `.drawingGroup()` if complex glass hierarchies feel sluggish:**

```swift
GlassEffectContainer {
    // Many glass elements
}
.drawingGroup()  // Rasterizes to Metal texture before compositing
```

6. **Make glass views equatable when possible:**

```swift
struct MyGlassButton: View, Equatable {
    let title: String
    // SwiftUI can skip re-render if equal
    static func == (lhs: Self, rhs: Self) -> Bool { lhs.title == rhs.title }
    var body: some View { /* ... */ }
}
```

7. **Test on real device.** Simulator approximates the effect. Real rendering behavior (and thermal impact) only shows on device.

---

## ═══════════════════════════════════════════════════
## CHAPTER 16 — ANIMATION CURVES FOR LIQUID GLASS
## ═══════════════════════════════════════════════════

These animation curves are specifically tuned for glass interactions:

```swift
// PRIMARY: Use .bouncy for morphing and expand/collapse
withAnimation(.bouncy) { }
withAnimation(.bouncy(duration: 0.4)) { }
withAnimation(.bouncy(duration: 0.5, extraBounce: 0.1)) { }

// SMOOTH: For slower, more deliberate transitions
withAnimation(.smooth) { }
withAnimation(.smooth(duration: 0.35)) { }

// SNAPPY: For quick state flips
withAnimation(.snappy) { }
withAnimation(.snappy(duration: 0.25)) { }

// SPRING: For full control
withAnimation(.spring(response: 0.4, dampingFraction: 0.7)) { }
withAnimation(.spring(duration: 0.45, bounce: 0.2)) { }

// For icon swaps inside glass
.contentTransition(.symbolEffect(.replace))
.contentTransition(.symbolEffect(.bounce))
.contentTransition(.symbolEffect(.pulse))

// NEVER use linear for glass motion — it feels mechanical and wrong
withAnimation(.linear) { }  // ❌ Avoid for glass
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 17 — COMMON PITFALLS & FIXES
## ═══════════════════════════════════════════════════

| Problem | Cause | Fix |
|---------|-------|-----|
| Glass appears black/opaque | Solid background behind glass | Ensure real content (gradient, photo, etc.) exists behind glass |
| Glass doesn't morph | Missing `GlassEffectContainer` or `@Namespace` | Wrap elements in container, add IDs to both elements |
| Elements don't merge | Spacing threshold too large or container missing | Add/reduce `GlassEffectContainer(spacing:)` value |
| `.glassProminent` + `.circle` = artifacts | Known OS issue | Add `.clipShape(Circle())` after `buttonBorderShape(.circle)` |
| Animation feels wrong | Using `.linear` or `.easeInOut` | Switch to `.bouncy`, `.spring`, or `.smooth` |
| Glass over white/light background | Glass barely visible | Glass needs visual content behind it — add gradient or imagery to background |
| Sheet looks opaque | Custom `presentationBackground` overriding glass | Remove the custom `presentationBackground` |
| Glass on scroll content feels heavy | Glass applied to list rows/cards | Move glass to navigation layer only; remove from content |
| UIKit button action not firing | `UIVisualEffectView` intercepting touches | Place tappable subviews inside `contentView`, not on the effect view |

---

## ═══════════════════════════════════════════════════
## CHAPTER 18 — APPLE DESIGN RULES (Never Violate These)
## ═══════════════════════════════════════════════════

These are Apple's guiding principles. Breaking them makes apps feel wrong and out of place on iOS 26:

1. **Navigation layer only.** Glass floats above content. It is never IN the content.

2. **Tint = semantic meaning.** Blue means primary. Red means destructive. Not decoration.

3. **One glass world.** Use `GlassEffectContainer` to connect related elements. Isolated, scattered individual glass elements look chaotic.

4. **Fluid state changes.** No instant pops. Use `withAnimation(.bouncy)` for every glass state change.

5. **Symbol first.** In toolbars and tab bars, prefer SF Symbols over text labels.

6. **Concentric corners everywhere.** Use `.containerConcentric` for glass nested inside containers.

7. **Never fight the material.** No competing shadows, blurs, or backgrounds. Let glass do its thing.

8. **Every touch must feel physical.** Use `.interactive()` on anything tappable.

9. **Content is king.** Glass must not distract from what the user came to see.

10. **Prominent = primary action only.** `.glassProminent` is for the ONE most important action on screen, not for emphasis in general.

---

## ═══════════════════════════════════════════════════
## CHAPTER 19 — BACKWARD COMPATIBILITY
## ═══════════════════════════════════════════════════

When your app must support iOS versions older than 26:

```swift
// Centralized adapter — call this instead of using glassEffect directly
extension View {
    @ViewBuilder
    func adaptiveGlassEffect(
        variant: Glass? = nil,
        shape: AnyShape = AnyShape(.capsule),
        tint: Color? = nil,
        interactive: Bool = false
    ) -> some View {
        if #available(iOS 26.0, *) {
            var glass: Glass = variant ?? .regular
            if let tint { glass = glass.tint(tint) }
            if interactive { glass = glass.interactive() }
            self.glassEffect(glass, in: shape)
        } else {
            // Graceful fallback: material + stroke approximation
            self
                .background {
                    shape
                        .fill(.ultraThinMaterial)
                        .overlay(
                            shape.strokeBorder(
                                Color.white.opacity(0.25),
                                lineWidth: 0.5
                            )
                        )
                }
        }
    }
}

// Usage — works on iOS 17+ and iOS 26+
Button("Action") { }
    .padding()
    .adaptiveGlassEffect(tint: .blue, interactive: true)
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 20 — COMPLETE CUSTOM LIQUID GLASS CREATIONS
## ═══════════════════════════════════════════════════

This is the creative library. Every example below is production-ready and teaches a unique pattern. Use these as templates to build anything the user asks for.

---

### CUSTOM CREATION 1: Floating Action Button Cluster (Radial Burst)

A single button that expands into a ring of action buttons, each morphing out of the center.

```swift
import SwiftUI

struct RadialGlassFAB: View {
    @State private var isOpen = false
    @Namespace private var ns

    struct Action: Identifiable {
        let id: String
        let icon: String
        let tint: Color
        let angle: Double  // degrees
    }

    let actions: [Action] = [
        Action(id: "camera", icon: "camera.fill", tint: .pink, angle: 270),
        Action(id: "gallery", icon: "photo.fill", tint: .purple, angle: 330),
        Action(id: "mic",    icon: "mic.fill",    tint: .orange, angle: 30),
        Action(id: "doc",    icon: "doc.fill",    tint: .blue,   angle: 90),
        Action(id: "link",   icon: "link",        tint: .green,  angle: 150),
        Action(id: "map",    icon: "map.fill",    tint: .teal,   angle: 210),
    ]

    let radius: CGFloat = 96

    var body: some View {
        ZStack {
            // Action buttons radiate outward
            ForEach(actions) { action in
                if isOpen {
                    let angle = action.angle * .pi / 180
                    let x = cos(angle) * radius
                    let y = sin(angle) * radius

                    Button { withAnimation(.bouncy) { isOpen = false } } label: {
                        Image(systemName: action.icon)
                            .font(.title3)
                            .frame(width: 52, height: 52)
                    }
                    .buttonStyle(.glassProminent)
                    .buttonBorderShape(.circle)
                    .tint(action.tint)
                    .glassEffectID(action.id, in: ns)
                    .offset(x: x, y: y)
                    .transition(.scale(scale: 0.3).combined(with: .opacity))
                }
            }

            // Center toggle button — always visible
            GlassEffectContainer(spacing: 120) {
                Button {
                    withAnimation(.bouncy(duration: 0.5)) { isOpen.toggle() }
                } label: {
                    Image(systemName: isOpen ? "xmark" : "plus")
                        .font(.title2.bold())
                        .frame(width: 60, height: 60)
                        .contentTransition(.symbolEffect(.replace))
                }
                .buttonStyle(.glassProminent)
                .buttonBorderShape(.circle)
                .tint(.indigo)
                .glassEffectID("center", in: ns)
            }
        }
        .frame(width: radius * 2 + 80, height: radius * 2 + 80)
    }
}
```

---

### CUSTOM CREATION 2: Expanding Music Player (Collapsed ↔ Full)

A compact now-playing bar that expands into a full player using glass morphing.

```swift
import SwiftUI

struct GlassMusicPlayer: View {
    @State private var isExpanded = false
    @State private var isPlaying = false
    @Namespace private var ns

    var body: some View {
        ZStack {
            // Rich background — glass needs content behind it
            LinearGradient(
                colors: [.purple, .indigo, .black],
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            .ignoresSafeArea()

            VStack {
                Spacer()

                GlassEffectContainer(spacing: 20) {
                    if isExpanded {
                        expandedPlayer
                    } else {
                        collapsedBar
                    }
                }
                .padding()
            }
        }
    }

    // ── Collapsed bar ──────────────────────────────────
    var collapsedBar: some View {
        HStack(spacing: 12) {
            // Artwork
            RoundedRectangle(cornerRadius: 8)
                .fill(.white.opacity(0.2))
                .frame(width: 36, height: 36)
                .overlay(Image(systemName: "music.note").font(.callout))
                .glassEffect(.regular.interactive())
                .glassEffectID("artwork", in: ns)

            Text("Song Title")
                .font(.subheadline.bold())
                .lineLimit(1)

            Spacer()

            // Play/pause
            Button {
                withAnimation(.bouncy) { isPlaying.toggle() }
            } label: {
                Image(systemName: isPlaying ? "pause.fill" : "play.fill")
                    .font(.title3)
                    .frame(width: 44, height: 44)
                    .contentTransition(.symbolEffect(.replace))
            }
            .buttonStyle(.glassProminent)
            .buttonBorderShape(.circle)
            .tint(.white)
            .glassEffectID("playPause", in: ns)

            // Expand chevron
            Button {
                withAnimation(.bouncy(duration: 0.5)) { isExpanded = true }
            } label: {
                Image(systemName: "chevron.up")
                    .frame(width: 36, height: 36)
            }
            .buttonStyle(.glass)
            .buttonBorderShape(.circle)
            .glassEffectID("expand", in: ns)
        }
        .padding(.horizontal, 16)
        .padding(.vertical, 12)
        .glassBackgroundEffect(in: RoundedRectangle(cornerRadius: 20))
    }

    // ── Expanded player ────────────────────────────────
    var expandedPlayer: some View {
        VStack(spacing: 24) {
            // Dismiss
            HStack {
                Spacer()
                Button {
                    withAnimation(.bouncy(duration: 0.5)) { isExpanded = false }
                } label: {
                    Image(systemName: "chevron.down")
                        .frame(width: 36, height: 36)
                }
                .buttonStyle(.glass)
                .buttonBorderShape(.circle)
                .glassEffectID("expand", in: ns)
            }

            // Artwork (morphs from small)
            RoundedRectangle(cornerRadius: 20)
                .fill(.white.opacity(0.15))
                .frame(width: 220, height: 220)
                .overlay(
                    Image(systemName: "music.note")
                        .font(.system(size: 64))
                )
                .glassEffect(.regular.interactive())
                .glassEffectID("artwork", in: ns)

            // Track info
            VStack(spacing: 4) {
                Text("Song Title")
                    .font(.title2.bold())
                Text("Artist Name")
                    .font(.body)
                    .foregroundStyle(.secondary)
            }

            // Progress bar placeholder
            Capsule()
                .fill(.white.opacity(0.3))
                .frame(height: 4)
                .overlay(alignment: .leading) {
                    Capsule()
                        .fill(.white)
                        .frame(width: 120, height: 4)
                }

            // Playback controls
            HStack(spacing: 28) {
                Button { } label: {
                    Image(systemName: "backward.fill")
                        .font(.title2)
                        .frame(width: 52, height: 52)
                }
                .buttonStyle(.glass)
                .buttonBorderShape(.circle)

                Button {
                    withAnimation(.bouncy) { isPlaying.toggle() }
                } label: {
                    Image(systemName: isPlaying ? "pause.fill" : "play.fill")
                        .font(.title)
                        .frame(width: 68, height: 68)
                        .contentTransition(.symbolEffect(.replace))
                }
                .buttonStyle(.glassProminent)
                .buttonBorderShape(.circle)
                .tint(.white)
                .glassEffectID("playPause", in: ns)

                Button { } label: {
                    Image(systemName: "forward.fill")
                        .font(.title2)
                        .frame(width: 52, height: 52)
                }
                .buttonStyle(.glass)
                .buttonBorderShape(.circle)
            }
        }
        .padding(24)
        .glassBackgroundEffect(in: RoundedRectangle(cornerRadius: 32))
    }
}
```

---

### CUSTOM CREATION 3: Glass Segmented Control (Custom)

A fully custom segmented control with morphing glass selection indicator.

```swift
import SwiftUI

struct GlassSegmentedControl: View {
    let options: [String]
    @Binding var selection: String
    @Namespace private var ns

    var body: some View {
        GlassEffectContainer(spacing: 4) {
            HStack(spacing: 2) {
                ForEach(options, id: \.self) { option in
                    Button {
                        withAnimation(.bouncy(duration: 0.3)) {
                            selection = option
                        }
                    } label: {
                        Text(option)
                            .font(.subheadline.bold())
                            .padding(.horizontal, 16)
                            .padding(.vertical, 8)
                            .frame(maxWidth: .infinity)
                    }
                    .buttonStyle(selection == option ? .glassProminent : .glass)
                    .tint(selection == option ? .blue : .clear)
                    .glassEffectID(option, in: ns)
                }
            }
            .padding(4)
            .glassBackgroundEffect(in: Capsule())
        }
    }
}

// Usage:
@State private var selectedPeriod = "Week"

GlassSegmentedControl(
    options: ["Day", "Week", "Month", "Year"],
    selection: $selectedPeriod
)
```

---

### CUSTOM CREATION 4: Glass Notification Toast

An animated glass notification banner that slides in from top and auto-dismisses.

```swift
import SwiftUI

struct GlassToast: View {
    let title: String
    let message: String
    let icon: String
    let accent: Color
    @Binding var isVisible: Bool

    var body: some View {
        if isVisible {
            GlassEffectContainer {
                HStack(spacing: 12) {
                    // Icon
                    Image(systemName: icon)
                        .font(.title3)
                        .frame(width: 40, height: 40)
                        .glassEffect(.regular.tint(accent).interactive())

                    // Text
                    VStack(alignment: .leading, spacing: 2) {
                        Text(title)
                            .font(.subheadline.bold())
                        Text(message)
                            .font(.caption)
                            .foregroundStyle(.secondary)
                            .lineLimit(2)
                    }

                    Spacer()

                    // Dismiss
                    Button {
                        withAnimation(.bouncy) { isVisible = false }
                    } label: {
                        Image(systemName: "xmark")
                            .frame(width: 28, height: 28)
                    }
                    .buttonStyle(.glass)
                    .buttonBorderShape(.circle)
                    .controlSize(.small)
                }
                .padding(.horizontal, 16)
                .padding(.vertical, 12)
            }
            .glassBackgroundEffect(in: RoundedRectangle(cornerRadius: 18))
            .shadow(color: accent.opacity(0.15), radius: 20, y: 8)
            .transition(.move(edge: .top).combined(with: .opacity))
            .onAppear {
                // Auto-dismiss after 4 seconds
                DispatchQueue.main.asyncAfter(deadline: .now() + 4) {
                    withAnimation(.bouncy) { isVisible = false }
                }
            }
        }
    }
}

// Usage:
@State private var showToast = false

VStack {
    Button("Show Toast") { withAnimation(.bouncy) { showToast = true } }
}
.overlay(alignment: .top) {
    GlassToast(
        title: "Message Sent",
        message: "Your message was delivered successfully.",
        icon: "checkmark.circle.fill",
        accent: .green,
        isVisible: $showToast
    )
    .padding(.top, 16)
    .padding(.horizontal, 16)
}
```

---

### CUSTOM CREATION 5: Glass Context Menu (Custom Floating)

A custom context menu with glass morphing between states.

```swift
import SwiftUI

struct GlassContextMenu: View {
    @Binding var isShowing: Bool
    let items: [(label: String, icon: String, isDestructive: Bool, action: () -> Void)]
    @Namespace private var ns

    var body: some View {
        if isShowing {
            GlassEffectContainer(spacing: 0) {
                VStack(spacing: 0) {
                    ForEach(Array(items.enumerated()), id: \.offset) { index, item in
                        Button {
                            item.action()
                            withAnimation(.bouncy) { isShowing = false }
                        } label: {
                            HStack {
                                Text(item.label)
                                    .font(.body)
                                    .foregroundStyle(item.isDestructive ? .red : .primary)
                                Spacer()
                                Image(systemName: item.icon)
                                    .foregroundStyle(item.isDestructive ? .red : .secondary)
                            }
                            .padding(.horizontal, 16)
                            .padding(.vertical, 13)
                            .frame(minWidth: 220)
                        }
                        .buttonStyle(.glass)
                        .glassEffectID("item_\(index)", in: ns)

                        if index < items.count - 1 {
                            Rectangle()
                                .fill(.white.opacity(0.1))
                                .frame(height: 0.5)
                        }
                    }
                }
                .glassBackgroundEffect(in: RoundedRectangle(cornerRadius: 14))
            }
            .shadow(color: .black.opacity(0.25), radius: 24, y: 8)
            .transition(.scale(scale: 0.85, anchor: .top).combined(with: .opacity))
        }
    }
}

// Usage:
@State private var showContextMenu = false

Text("Long press me")
    .onLongPressGesture {
        withAnimation(.bouncy) { showContextMenu = true }
    }
    .overlay(alignment: .topTrailing) {
        GlassContextMenu(
            isShowing: $showContextMenu,
            items: [
                ("Edit",   "pencil",    false, { /* edit action */ }),
                ("Share",  "square.and.arrow.up", false, { /* share */ }),
                ("Delete", "trash",     true,  { /* delete */ }),
            ]
        )
        .offset(x: 0, y: 32)
    }
```

---

### CUSTOM CREATION 6: Glass Dashboard Card Grid

Health/stats-style dashboard with tinted glass cards.

```swift
import SwiftUI

struct GlassDashboardCard: View {
    let title: String
    let value: String
    let unit: String
    let icon: String
    let accent: Color
    let trend: String

    var body: some View {
        VStack(alignment: .leading, spacing: 14) {
            HStack {
                Image(systemName: icon)
                    .font(.callout)
                    .frame(width: 34, height: 34)
                    .glassEffect(.regular.tint(accent).interactive())

                Spacer()

                Text(trend)
                    .font(.caption2)
                    .foregroundStyle(.secondary)
            }

            VStack(alignment: .leading, spacing: 2) {
                HStack(alignment: .firstTextBaseline, spacing: 3) {
                    Text(value)
                        .font(.title.bold().monospacedDigit())
                    if !unit.isEmpty {
                        Text(unit)
                            .font(.caption)
                            .foregroundStyle(.secondary)
                    }
                }
                Text(title)
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }
        }
        .padding(16)
        .glassBackgroundEffect(in: RoundedRectangle(cornerRadius: 20))
        .overlay(
            RoundedRectangle(cornerRadius: 20)
                .strokeBorder(accent.opacity(0.2), lineWidth: 1)
        )
    }
}

struct GlassDashboard: View {
    let columns = [GridItem(.flexible()), GridItem(.flexible())]

    var body: some View {
        ScrollView {
            LazyVGrid(columns: columns, spacing: 14) {
                GlassDashboardCard(
                    title: "Heart Rate",
                    value: "72",
                    unit: "BPM",
                    icon: "heart.fill",
                    accent: .red,
                    trend: "+3 today"
                )
                GlassDashboardCard(
                    title: "Steps",
                    value: "8,420",
                    unit: "steps",
                    icon: "figure.walk",
                    accent: .green,
                    trend: "80% goal"
                )
                GlassDashboardCard(
                    title: "Sleep",
                    value: "7h 23m",
                    unit: "",
                    icon: "moon.zzz.fill",
                    accent: .indigo,
                    trend: "Good"
                )
                GlassDashboardCard(
                    title: "Calories",
                    value: "1,840",
                    unit: "kcal",
                    icon: "flame.fill",
                    accent: .orange,
                    trend: "On track"
                )
            }
            .padding()
        }
    }
}
```

---

### CUSTOM CREATION 7: Morphing Onboarding Flow

Multi-step glass onboarding with morphing step indicator and page transitions.

```swift
import SwiftUI

struct GlassOnboardingFlow: View {
    @State private var step = 0
    @Namespace private var ns

    struct Step {
        let icon: String
        let title: String
        let description: String
        let accent: Color
    }

    let steps: [Step] = [
        Step(icon: "sparkles",          title: "Welcome",      description: "Experience the next generation of iOS design.",      accent: .purple),
        Step(icon: "hand.tap.fill",     title: "Touch Glass",  description: "Every element responds to your touch with life.",    accent: .blue),
        Step(icon: "rays",              title: "Always Fresh", description: "Glass adapts to whatever is behind it, always.",     accent: .orange),
        Step(icon: "checkmark.seal.fill", title: "Ready",     description: "You're all set. Enjoy the experience.",              accent: .green),
    ]

    var body: some View {
        ZStack {
            // Dynamic gradient background (glass needs real content behind it)
            MeshGradient(
                width: 3, height: 3,
                points: [[0,0],[0.5,0],[1,0],[0,0.5],[0.5,0.5],[1,0.5],[0,1],[0.5,1],[1,1]],
                colors: [.purple,.blue,.cyan,.indigo,.purple,.blue,.cyan,.indigo,.purple]
            )
            .ignoresSafeArea()
            .animation(.easeInOut(duration: 3).repeatForever(autoreverses: true), value: step)

            VStack(spacing: 48) {
                Spacer()

                // Icon (morphs between steps)
                GlassEffectContainer(spacing: 30) {
                    Image(systemName: steps[step].icon)
                        .font(.system(size: 56, weight: .semibold))
                        .frame(width: 140, height: 140)
                        .glassEffect(
                            .regular.tint(steps[step].accent.opacity(0.3)).interactive()
                        )
                        .glassEffectID("stepIcon", in: ns)
                        .contentTransition(.symbolEffect(.replace))
                }

                // Text
                VStack(spacing: 10) {
                    Text(steps[step].title)
                        .font(.largeTitle.bold())
                        .foregroundStyle(.white)
                        .transition(.blurReplace)
                        .id("title_\(step)")

                    Text(steps[step].description)
                        .font(.body)
                        .foregroundStyle(.white.opacity(0.8))
                        .multilineTextAlignment(.center)
                        .padding(.horizontal, 40)
                        .transition(.blurReplace)
                        .id("desc_\(step)")
                }

                Spacer()

                // Navigation controls
                VStack(spacing: 20) {
                    // Step dots
                    GlassEffectContainer(spacing: 8) {
                        HStack(spacing: 8) {
                            ForEach(0..<steps.count, id: \.self) { i in
                                Capsule()
                                    .fill(i == step ? .white : .white.opacity(0.3))
                                    .frame(width: i == step ? 24 : 8, height: 8)
                                    .glassEffect(.identity)
                                    .glassEffectID("dot_\(i)", in: ns)
                                    .animation(.bouncy, value: step)
                            }
                        }
                        .padding(.horizontal, 16)
                        .padding(.vertical, 10)
                        .glassBackgroundEffect(in: Capsule())
                    }

                    // Next / Get Started button
                    Button {
                        withAnimation(.bouncy(duration: 0.45)) {
                            if step < steps.count - 1 { step += 1 }
                        }
                    } label: {
                        Text(step == steps.count - 1 ? "Get Started" : "Continue")
                            .font(.headline)
                            .padding(.horizontal, 40)
                            .padding(.vertical, 16)
                    }
                    .buttonStyle(.glassProminent)
                    .tint(steps[step].accent)
                    .controlSize(.large)
                    .buttonBorderShape(.capsule)
                    .disabled(step == steps.count - 1)
                }
                .padding(.bottom, 48)
            }
        }
    }
}
```

---

### CUSTOM CREATION 8: Glass Floating Sidebar Nav

A floating vertical pill navigation bar hovering over content.

```swift
import SwiftUI

struct GlassFloatingSidebar: View {
    @State private var selected = "home"
    @Namespace private var ns

    struct NavItem: Identifiable {
        let id: String
        let icon: String
        let label: String
    }

    let items: [NavItem] = [
        NavItem(id: "home",     icon: "house.fill",      label: "Home"),
        NavItem(id: "search",   icon: "magnifyingglass", label: "Search"),
        NavItem(id: "messages", icon: "message.fill",    label: "Messages"),
        NavItem(id: "profile",  icon: "person.fill",     label: "Profile"),
        NavItem(id: "settings", icon: "gear",            label: "Settings"),
    ]

    var body: some View {
        HStack {
            // The floating sidebar
            GlassEffectContainer(spacing: 6) {
                VStack(spacing: 6) {
                    ForEach(items) { item in
                        Button {
                            withAnimation(.bouncy(duration: 0.35)) {
                                selected = item.id
                            }
                        } label: {
                            VStack(spacing: 4) {
                                Image(systemName: item.icon)
                                    .font(selected == item.id ? .title3 : .body)
                                if selected == item.id {
                                    Text(item.label)
                                        .font(.caption2.bold())
                                        .transition(.opacity.combined(with: .scale))
                                }
                            }
                            .frame(width: 52)
                            .padding(.vertical, selected == item.id ? 12 : 10)
                        }
                        .buttonStyle(selected == item.id ? .glassProminent : .glass)
                        .tint(selected == item.id ? .blue : .clear)
                        .buttonBorderShape(.roundedRectangle(radius: 14))
                        .glassEffectID(item.id, in: ns)
                    }
                }
                .padding(.vertical, 8)
                .padding(.horizontal, 6)
                .glassBackgroundEffect(in: RoundedRectangle(cornerRadius: 26))
            }
            .padding(.leading, 12)

            Spacer()

            // Main content area
            VStack {
                Text("Main Content")
                    .font(.title)
                Spacer()
            }
            .padding()
        }
    }
}
```

---

### CUSTOM CREATION 9: Glass Maps-Style Control Cluster

Floating controls over a map-like view, with union and merging.

```swift
import SwiftUI

struct GlassMapsControls: View {
    @State private var mapStyle = "standard"
    @Namespace private var ns

    var body: some View {
        ZStack {
            // Simulated map background
            LinearGradient(colors: [.green.opacity(0.3), .blue.opacity(0.2), .brown.opacity(0.3)],
                           startPoint: .topLeading, endPoint: .bottomTrailing)
                .ignoresSafeArea()

            VStack {
                HStack {
                    Spacer()

                    // Right-side control column
                    VStack(spacing: 12) {
                        // Map style toggle
                        GlassEffectContainer(spacing: 0) {
                            VStack(spacing: 0) {
                                Button { withAnimation(.bouncy) { mapStyle = "standard" } } label: {
                                    Image(systemName: "map.fill")
                                        .frame(width: 44, height: 44)
                                }
                                .buttonStyle(mapStyle == "standard" ? .glassProminent : .glass)
                                .tint(mapStyle == "standard" ? .blue : .clear)
                                .glassEffectUnion(id: "mapStyle", namespace: ns)

                                Rectangle().fill(.white.opacity(0.15)).frame(height: 0.5)

                                Button { withAnimation(.bouncy) { mapStyle = "satellite" } } label: {
                                    Image(systemName: "globe.americas.fill")
                                        .frame(width: 44, height: 44)
                                }
                                .buttonStyle(mapStyle == "satellite" ? .glassProminent : .glass)
                                .tint(mapStyle == "satellite" ? .blue : .clear)
                                .glassEffectUnion(id: "mapStyle", namespace: ns)
                            }
                        }

                        // Zoom controls
                        GlassEffectContainer(spacing: 0) {
                            VStack(spacing: 0) {
                                Button { } label: {
                                    Image(systemName: "plus")
                                        .frame(width: 44, height: 44)
                                }
                                .buttonStyle(.glass)
                                .glassEffectUnion(id: "zoom", namespace: ns)

                                Rectangle().fill(.white.opacity(0.15)).frame(height: 0.5)

                                Button { } label: {
                                    Image(systemName: "minus")
                                        .frame(width: 44, height: 44)
                                }
                                .buttonStyle(.glass)
                                .glassEffectUnion(id: "zoom", namespace: ns)
                            }
                        }

                        // Location button
                        Button { } label: {
                            Image(systemName: "location.fill")
                                .frame(width: 44, height: 44)
                        }
                        .buttonStyle(.glass)
                        .buttonBorderShape(.circle)
                    }
                    .padding(.trailing, 16)
                }

                Spacer()
            }
            .padding(.top, 60)
        }
    }
}
```

---

### CUSTOM CREATION 10: Glass Search Bar with Live Suggestions

An expanding search bar with glass suggestion chips.

```swift
import SwiftUI

struct GlassSearchBar: View {
    @State private var query = ""
    @State private var isSearching = false
    @State private var suggestions = ["SwiftUI", "Liquid Glass", "iOS 26", "Apple", "Xcode"]
    @Namespace private var ns
    @FocusState private var isFocused: Bool

    var filteredSuggestions: [String] {
        query.isEmpty ? suggestions : suggestions.filter { $0.localizedCaseInsensitiveContains(query) }
    }

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Search input
            GlassEffectContainer(spacing: 12) {
                HStack(spacing: 10) {
                    Image(systemName: "magnifyingglass")
                        .foregroundStyle(.secondary)

                    TextField("Search...", text: $query)
                        .focused($isFocused)
                        .onSubmit { isSearching = true }

                    if !query.isEmpty {
                        Button {
                            withAnimation(.bouncy) {
                                query = ""
                                isSearching = false
                            }
                        } label: {
                            Image(systemName: "xmark.circle.fill")
                                .foregroundStyle(.secondary)
                        }
                        .buttonStyle(.plain)
                    }
                }
                .padding(.horizontal, 14)
                .padding(.vertical, 12)
                .glassBackgroundEffect(in: Capsule())
                .glassEffectID("searchBar", in: ns)
            }
            .onChange(of: isFocused) { _, focused in
                withAnimation(.bouncy) { isSearching = focused }
            }

            // Suggestion chips — appear when searching
            if isSearching && !filteredSuggestions.isEmpty {
                GlassEffectContainer(spacing: 8) {
                    FlowLayout(spacing: 8) {
                        ForEach(filteredSuggestions, id: \.self) { suggestion in
                            Button {
                                withAnimation(.bouncy) {
                                    query = suggestion
                                    isFocused = false
                                    isSearching = false
                                }
                            } label: {
                                Text(suggestion)
                                    .font(.subheadline)
                                    .padding(.horizontal, 14)
                                    .padding(.vertical, 8)
                            }
                            .buttonStyle(.glass)
                            .buttonBorderShape(.capsule)
                            .glassEffectID(suggestion, in: ns)
                        }
                    }
                }
                .transition(.move(edge: .top).combined(with: .opacity))
            }
        }
    }
}

// Simple flow layout for wrapping chips
struct FlowLayout: Layout {
    let spacing: CGFloat

    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        let result = FlowResult(in: proposal.replacingUnspecifiedDimensions().width, subviews: subviews, spacing: spacing)
        return result.size
    }

    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        let result = FlowResult(in: bounds.width, subviews: subviews, spacing: spacing)
        for (index, subview) in subviews.enumerated() {
            subview.place(at: CGPoint(x: bounds.minX + result.frames[index].minX,
                                     y: bounds.minY + result.frames[index].minY),
                         proposal: ProposedViewSize(result.frames[index].size))
        }
    }

    struct FlowResult {
        var frames: [CGRect] = []
        var size: CGSize = .zero

        init(in maxWidth: CGFloat, subviews: Subviews, spacing: CGFloat) {
            var x: CGFloat = 0, y: CGFloat = 0, rowHeight: CGFloat = 0
            for subview in subviews {
                let size = subview.sizeThatFits(.unspecified)
                if x + size.width > maxWidth && x > 0 {
                    x = 0; y += rowHeight + spacing; rowHeight = 0
                }
                frames.append(CGRect(origin: CGPoint(x: x, y: y), size: size))
                rowHeight = max(rowHeight, size.height)
                x += size.width + spacing
            }
            self.size = CGSize(width: maxWidth, height: y + rowHeight)
        }
    }
}
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 21 — COMPLETE API QUICK-REFERENCE
## ═══════════════════════════════════════════════════

```swift
// ────────────────────────────────────────────────
// SwiftUI — Core Modifiers
// ────────────────────────────────────────────────
.glassEffect()                                                    // Basic
.glassEffect(_ glass: Glass)                                      // With variant
.glassEffect(_ glass: Glass, in shape: some Shape)                // With shape
.glassEffect(_ glass: Glass, in shape: some Shape, isEnabled: Bool) // Conditional

.glassEffectID(_ id: some Hashable, in namespace: Namespace.ID)   // For morphing
.glassEffectUnion(id: some Hashable, namespace: Namespace.ID)     // Shared shape
.glassEffectTransition(_ t: GlassEffectTransition)                // Appear/disappear style
.glassBackgroundEffect(in: some Shape, displayMode: GlassDisplayMode) // Container background

// ────────────────────────────────────────────────
// Glass Type
// ────────────────────────────────────────────────
Glass.regular          // Default, adaptive
Glass.clear            // Transparent (use with strict rules)
Glass.identity         // No effect

.tint(_ color: Color)  // Chain on Glass
.interactive()         // Chain on Glass

// ────────────────────────────────────────────────
// Transition Types
// ────────────────────────────────────────────────
GlassEffectTransition.identity
GlassEffectTransition.matchedGeometry
GlassEffectTransition.materialize

// ────────────────────────────────────────────────
// Display Modes
// ────────────────────────────────────────────────
GlassDisplayMode.always
GlassDisplayMode.whenScrolled

// ────────────────────────────────────────────────
// Container
// ────────────────────────────────────────────────
GlassEffectContainer { }
GlassEffectContainer(spacing: CGFloat) { }

// ────────────────────────────────────────────────
// Button Styles
// ────────────────────────────────────────────────
.buttonStyle(.glass)
.buttonStyle(.glassProminent)

// ────────────────────────────────────────────────
// Border Shapes
// ────────────────────────────────────────────────
.buttonBorderShape(.capsule)
.buttonBorderShape(.circle)
.buttonBorderShape(.roundedRectangle(radius: CGFloat))
.buttonBorderShape(.automatic)

// ────────────────────────────────────────────────
// Control Sizes
// ────────────────────────────────────────────────
.controlSize(.mini)
.controlSize(.small)
.controlSize(.regular)
.controlSize(.large)
.controlSize(.extraLarge)

// ────────────────────────────────────────────────
// Shapes for `in:` parameter
// ────────────────────────────────────────────────
.capsule
.circle
.ellipse
.rect
.rect(cornerRadius: CGFloat)
.rect(cornerRadius: .containerConcentric)
RoundedRectangle(cornerRadius: CGFloat)
RoundedRectangle(cornerRadius: CGFloat, style: .continuous)
AnyCustomShape()   // Any type conforming to Shape

// ────────────────────────────────────────────────
// TabView
// ────────────────────────────────────────────────
.tabBarMinimizeBehavior(.automatic)
.tabBarMinimizeBehavior(.onScrollDown)
.tabBarMinimizeBehavior(.never)
.tabViewBottomAccessory { }
@Environment(\.tabViewBottomAccessoryPlacement) var placement
// placement == .expanded or .collapsed

Tab("Label", systemImage: "icon", role: .search)

// ────────────────────────────────────────────────
// Toolbar
// ────────────────────────────────────────────────
ToolbarSpacer(.fixed, spacing: CGFloat)
ToolbarSpacer(.flexible)
.sharedBackgroundVisibility(.hidden)
.badge(Int)

// ────────────────────────────────────────────────
// Accessibility Environment
// ────────────────────────────────────────────────
@Environment(\.accessibilityReduceTransparency) var reduceTransparency
@Environment(\.accessibilityReduceMotion) var reduceMotion

// ────────────────────────────────────────────────
// UIKit APIs
// ────────────────────────────────────────────────
UIGlassEffect()
UIGlassEffect().tintColor = UIColor
UIGlassEffect().isInteractive = Bool

UIGlassContainerEffect()
UIGlassContainerEffect().spacing = CGFloat

UIVisualEffectView(effect: UIGlassEffect())
UIView.animate { effectView.effect = UIGlassEffect() }  // Materialize
UIView.animate { effectView.effect = nil }              // Dematerialize

effectView.cornerConfiguration = UIViewCornerConfiguration(
    corners: .allCorners,
    cornerRadius: CGFloat           // Fixed radius
)
effectView.cornerConfiguration = UIViewCornerConfiguration(
    corners: .allCorners,
    cornerRadius: .containerRelative  // Matches parent
)

// UIKit Tab Bar
tabBarController.tabBarMinimizeBehavior = .onScrollDown
let accessory = UITabAccessory(contentView: yourView)
tabBarController.bottomAccessory = accessory
// traitCollection.tabAccessoryEnvironment == .inline (collapsed)
// traitCollection.tabAccessoryEnvironment == .default (expanded)

// ────────────────────────────────────────────────
// Animation Curves (use these for glass)
// ────────────────────────────────────────────────
withAnimation(.bouncy) { }
withAnimation(.bouncy(duration: 0.4)) { }
withAnimation(.bouncy(duration: 0.5, extraBounce: 0.1)) { }
withAnimation(.smooth) { }
withAnimation(.smooth(duration: 0.35)) { }
withAnimation(.snappy) { }
withAnimation(.spring(response: 0.4, dampingFraction: 0.7)) { }
withAnimation(.spring(duration: 0.45, bounce: 0.2)) { }
```

---

## ═══════════════════════════════════════════════════
## CHAPTER 22 — HOW TO BUILD ANYTHING CUSTOM
## ═══════════════════════════════════════════════════

When a user asks you to build a custom Liquid Glass component that isn't in this document, follow this decision tree:

**Step 1: Identify the layer**
Is this a navigation/control element (glass OK) or content (no glass)?

**Step 2: Choose the variant**
- Over rich media? → `.clear` (check all 3 conditions first)
- Everything else? → `.regular`
- Toggling on/off? → `.identity` for the off state

**Step 3: Choose the shape**
- Pill buttons, search bars, tags → `Capsule` / `.capsule`
- Icon-only buttons → `Circle` / `.circle`
- Cards, panels, sheets → `RoundedRectangle(cornerRadius: 16–32)`
- Nested in container → `.rect(cornerRadius: .containerConcentric)`
- Unique brand shapes → Custom `Shape` struct

**Step 4: Decide if morphing is needed**
- States change and glass should flow between them? → `GlassEffectContainer` + `@Namespace` + `.glassEffectID`
- Static elements that just need to coexist? → `GlassEffectContainer` (no IDs)
- Single isolated element? → Just `.glassEffect()` alone

**Step 5: Add interactivity**
- User will tap it? → `.interactive()` and `.buttonStyle(.glass)` or `.glassProminent`
- Decorative glass only? → No `.interactive()`

**Step 6: Background**
- Always ensure the background behind your glass has real, interesting content (gradient, photo, blur, etc.)
- Never place glass over a pure white or pure black solid background — it will be invisible or look wrong

**Step 7: Animation**
- Always use `.bouncy` or `.spring` for state changes. Never `.linear` or `.easeInOut` alone.

---

*END OF LIQUID GLASS MASTER TEACHING PROMPT*
*This document contains complete, confirmed knowledge of Apple's iOS 26 Liquid Glass system.*
*Minimum required: Xcode 26, iOS 26.0, Swift 6.*
*All code in this document is production-ready and idiomatic SwiftUI/UIKit.*
```
