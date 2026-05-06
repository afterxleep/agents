---
name: pixel-perfect-design
description: Designer-grade UI implementation from mockup images. Systematically analyzes visual hierarchy, spacing rhythm, typography, color, shadows, and micro-interactions. Extracts precise specs from images when no Figma/design tokens exist. Drives a build-compare-refine loop via FlowDeck screenshots until implementation is indistinguishable from the original design. Use when implementing UI from any visual reference or when the user requests pixel-perfect fidelity.
---

# Pixel-Perfect Design Implementation

## MANDATORY TRIGGER

Activate this skill when ANY of these conditions are true:

**Explicit signals (user provides a design reference):**
- User attaches an image file (PNG, JPG, screenshot, mockup, exported comp)
- User says "build this", "create this screen", "implement this design", "make it look like this"
- User says "pixel perfect", "match the design", "design fidelity"

**Implicit signals (user is describing a UI to build):**
- User describes a specific screen layout with visual details (colors, spacing, typography)
- User references a design system, brand guidelines, or specific visual treatment
- User provides a sketch or wireframe (even hand-drawn)
- User asks to "recreate" or "clone" an existing app's UI from a screenshot

**During implementation (mid-task triggers):**
- You just implemented a UI view and haven't visually verified it yet — run the validation loop
- User says "does it look right?", "check the UI", "how does it look?"
- User reports the UI "doesn't match", "looks off", "spacing is wrong"
- You made changes to a view's layout, colors, typography, or effects — re-validate

**Figma links**: If the user provides a Figma URL (e.g., `figma.com/design/...`, `figma.com/file/...`), use the Figma MCP server to fetch exact design tokens, measurements, and specs. With Figma MCP data, skip Phase 0 and Phase 1 (measurement) and go directly to Phase 2 (Implementation). The rest of the workflow — layered implementation, optical corrections, and the FlowDeck validation loop — is the same regardless of design source.

## Core Philosophy

**Think like a designer, implement like an engineer.**

A pixel-perfect UI is not just correct measurements — it is correct *perception*. A mathematically centered element can look off-center. A color that matches hex values can feel wrong in context. A shadow with correct parameters can look heavy or flat depending on surrounding elements. This skill trains you to see and correct for these perceptual gaps.

---

## Phase 0: Design Reading (Before Touching Code)

Before measuring a single pixel, *read* the design. Understand what it communicates.

### Step 1: Identify Visual Hierarchy

Look at the mockup and answer:
1. **What is the primary focal point?** — The element the eye hits first (largest text, brightest color, most contrast)
2. **What is the secondary information?** — Supporting text, icons, metadata
3. **What is tertiary/ambient?** — Backgrounds, dividers, subtle borders
4. **What is the user's intended action?** — The primary CTA (call-to-action), its visual weight

Document this as a comment block before any code:
```swift
// VISUAL HIERARCHY
// 1. Primary: Hero illustration + title "Welcome Back" — large, bold, centered
// 2. Secondary: Subtitle explaining the screen purpose — lighter weight, muted color
// 3. Tertiary: Background gradient, decorative shapes
// 4. Action: "Continue" button — full-width, high-contrast, bottom-anchored
```

### Step 2: Identify the Design System

Scan the mockup for repeating patterns:
- **Spacing rhythm**: Are gaps consistently 8pt, 12pt, 16pt, 24pt? (Most designs use a 4pt or 8pt base grid)
- **Color palette**: Count distinct colors. Usually 2-3 primary + 2-3 neutral + 1-2 accent
- **Typography scale**: Count distinct text sizes/weights. Usually 3-5 levels
- **Corner radius pattern**: One or two standard radii used everywhere
- **Component repetition**: Cards, buttons, list items — same component styled consistently

### Step 3: Identify the Layout Strategy

Determine the structural approach:
- **Full-bleed vs. inset**: Does content go edge-to-edge or have margins?
- **Scroll vs. fixed**: Is this a scrollable list or a fixed layout?
- **Safe area behavior**: Does content respect safe areas or extend behind them?
- **Gravity**: Is content top-anchored, centered, or bottom-anchored?
- **Negative space**: Where is deliberate whitespace? It is as important as content.

---

## Phase 1: Precise Measurement from Images

When working from an image (not Figma), you must extract specs visually. Here is the systematic approach.

### Step 1: Establish Reference Dimensions

The mockup image may not be at 1x scale. Establish a baseline:
1. **Identify a known element** — Status bar height (54pt on modern iPhones), navigation bar (44pt), tab bar (49pt + safe area), or standard button height (44-50pt)
2. **Calculate the scale factor** — If the status bar measures 108px in the image, the scale is 2x
3. **Apply the scale factor** to all subsequent measurements

```swift
// REFERENCE: Status bar = 54pt (measured 108px in image = 2x scale)
// All measurements below are in points (image pixels / 2)
```

### Step 2: Measure the Spatial Grid

Work outside-in:
1. **Screen margins** — Distance from screen edges to outermost content
2. **Section spacing** — Gaps between major content blocks
3. **Element spacing** — Gaps between items within a section
4. **Internal padding** — Space inside containers (cards, buttons, bubbles)

Document with precision:
```swift
// SPATIAL GRID
// Screen margins: 20pt horizontal, content starts 60pt from safe area top
// Section spacing: 32pt between hero and form, 24pt between form and button
// Element spacing: 16pt between form fields, 8pt between label and field
// Card padding: 20pt horizontal, 16pt vertical
// Button padding: 16pt vertical (height derived from text + padding)
```

### Step 3: Extract Typography

For each distinct text style:
1. **Estimate size** — Compare against known references (body text is typically 15-17pt)
2. **Identify weight** — Regular (thin strokes), Medium (slightly thicker), Semibold/Bold (noticeably thicker)
3. **Check line height** — Measure baseline-to-baseline for multi-line text
4. **Note color** — Is it pure black? Dark gray? A tinted color?
5. **Note alignment and case** — Centered? Leading? Uppercase?

```swift
// TYPOGRAPHY
// Title: ~28pt Semibold, #1A1A1A, centered
// Subtitle: ~15pt Regular, #666666, centered, line-height ~22pt
// Button label: ~17pt Semibold, #FFFFFF, centered, uppercase
// Caption: ~13pt Regular, #999999, leading-aligned
```

### Step 4: Extract Colors

Systematically catalog every color:
1. **Backgrounds** — Screen bg, card bg, input bg, overlay bg
2. **Text** — Primary, secondary, tertiary, placeholder, link
3. **Interactive** — Button fill, button text, selection, focus ring
4. **Semantic** — Success (green), warning (amber), error (red), info (blue)
5. **Decorative** — Gradients, illustrations, divider lines

When extracting from images, describe colors precisely:
```swift
// COLORS (extracted from mockup)
// Screen background: near-white, ~#F8F8FA (not pure white — has cool tint)
// Card background: pure white #FFFFFF
// Primary text: near-black, ~#1A1A1A (not pure black — softer)
// Secondary text: medium gray, ~#6B7280
// Primary button: vibrant blue, ~#007AFF (system blue)
// Divider: very light gray, ~#E5E7EB at ~0.5 opacity
```

### Step 5: Extract Effects

For each visual effect:
1. **Shadows** — Estimate offset direction, blur radius, color/opacity
2. **Corner radii** — Measure against element size (small ~8pt, medium ~12-16pt, large ~20-28pt, pill ~height/2)
3. **Borders** — Width (hairline 0.5pt, thin 1pt, medium 2pt), color, opacity
4. **Blurs/Overlays** — Background blur, tinted overlays, gradients

```swift
// EFFECTS
// Card shadow: offset(0, 2), blur ~8pt, black @ 0.08 opacity
// Card radius: ~16pt
// Input border: 1pt, #E5E7EB, radius ~10pt
// Button radius: pill (height / 2)
// Overlay: black @ 0.4 opacity (for modals)
```

---

## Phase 2: Implementation — Structure First, Style Second

### Rule: Build in Layers

Implement in this exact order. Do not skip ahead.

#### Layer 1: Skeleton Layout (No Styling)

Build the spatial structure with placeholder content. Get every element positioned correctly with exact spacing. Use `spacing: 0` on all stacks and control gaps explicitly.

```swift
// ALWAYS use explicit spacing — never rely on defaults
VStack(spacing: 0) {
    // Hero section
    heroContent

    Spacer().frame(height: 32) // Section gap — explicit, measurable

    // Form section
    formContent

    Spacer().frame(height: 24) // Section gap

    // Action section
    actionContent
}
.padding(.horizontal, 20) // Screen margins
```

**Key patterns:**
```swift
// NEVER this — default padding is unpredictable
.padding()

// ALWAYS this — exact, intentional values
.padding(.horizontal, 20)
.padding(.top, 16)
.padding(.bottom, 12)

// For asymmetric padding, use EdgeInsets
.padding(EdgeInsets(top: 20, leading: 16, bottom: 12, trailing: 16))
```

#### Layer 2: Typography

Apply all text styles. Match size, weight, color, alignment, and line height.

```swift
Text("Welcome Back")
    .font(.system(size: 28, weight: .semibold))
    .foregroundStyle(Color(hex: "#1A1A1A"))
    .multilineTextAlignment(.center)

Text("Sign in to continue where you left off")
    .font(.system(size: 15, weight: .regular))
    .foregroundStyle(Color(hex: "#6B7280"))
    .lineSpacing(4) // Adjust to match design line height
    .multilineTextAlignment(.center)
```

**Line height in SwiftUI**: SwiftUI doesn't have a direct `lineHeight` property. Use `.lineSpacing()` which adds space *between* lines. To match a design's line height:
```swift
// Design says: font 15pt, line height 22pt
// SwiftUI default line height for 15pt ≈ 18pt
// lineSpacing = 22 - 18 = 4pt
.lineSpacing(4)
```

**Tracking/letter spacing**:
```swift
// Design says: 0.5pt letter spacing
.tracking(0.5)
// Or for tighter spacing (common in headlines)
.tracking(-0.3)
```

#### Layer 3: Colors and Backgrounds

Apply all background colors, text colors, and tints.

```swift
// Define colors once, use everywhere
enum DesignColors {
    static let background = Color(hex: "#F8F8FA")
    static let cardBackground = Color.white
    static let textPrimary = Color(hex: "#1A1A1A")
    static let textSecondary = Color(hex: "#6B7280")
    static let accent = Color(hex: "#007AFF")
}
```

#### Layer 4: Shapes, Borders, and Radii

Apply corner radii, borders, and shape clipping.

```swift
// Card with radius and border
.background(Color.white)
.clipShape(RoundedRectangle(cornerRadius: 16, style: .continuous))
// Use .continuous for Apple-style smooth corners (squircle)
// Use .circular only when the design explicitly uses geometric corners

// Border
.overlay(
    RoundedRectangle(cornerRadius: 16, style: .continuous)
        .stroke(Color(hex: "#E5E7EB"), lineWidth: 1)
)

// Pill shape (fully rounded)
.clipShape(Capsule())
```

**Important**: Always use `style: .continuous` for rounded rectangles unless the design clearly shows geometric (non-smooth) corners. Apple's design language uses continuous (squircle) corners everywhere.

#### Layer 5: Shadows and Effects

Apply shadows, blurs, and overlays last — they depend on correct shapes being in place.

```swift
// Subtle card shadow (common in modern iOS)
.shadow(color: Color.black.opacity(0.08), radius: 8, x: 0, y: 2)

// Elevated card shadow (more prominent)
.shadow(color: Color.black.opacity(0.04), radius: 2, x: 0, y: 1) // tight shadow
.shadow(color: Color.black.opacity(0.08), radius: 16, x: 0, y: 4) // ambient shadow

// Material blur background
.background(.ultraThinMaterial)
```

**Multiple shadows**: Real-world shadows typically need two layers — a tight shadow for definition and a larger ambient shadow for depth. Compare your single `.shadow()` against the design; if it looks flat, add a second layer.

---

## Phase 3: Optical Corrections

These are the details that separate engineer-quality from designer-quality UI.

### Optical Centering

Mathematical center ≠ visual center. Elements with visual weight at the bottom (like text descenders or icons with a base) need to shift slightly upward to *appear* centered.

```swift
// Text in a circle that looks optically centered
Text("A")
    .font(.system(size: 20, weight: .semibold))
    .frame(width: 40, height: 40)
    .offset(y: -1) // Nudge up 1pt to compensate for baseline
    .background(Circle().fill(Color.blue))
```

### Optical Spacing

Equal mathematical spacing between elements of different sizes looks uneven. Larger elements need more space around them.

```swift
// After a large header, use more spacing than between body elements
Text("Settings") // Large title
    .font(.system(size: 34, weight: .bold))
Spacer().frame(height: 24) // More space after large text

Text("Notifications") // Section header
    .font(.system(size: 17, weight: .semibold))
Spacer().frame(height: 8) // Less space after smaller text
```

### Icon-Text Alignment

Icons and text don't naturally align to the same baseline. Adjust with small offsets:

```swift
HStack(spacing: 8) {
    Image(systemName: "bell.fill")
        .font(.system(size: 16))
        .offset(y: -0.5) // Optical alignment with text baseline
    Text("Notifications")
        .font(.system(size: 17))
}
```

### Touch Targets

Minimum touch target is 44x44pt (Apple HIG). If a design shows a small icon button, extend the tap area:

```swift
Button(action: { }) {
    Image(systemName: "xmark")
        .font(.system(size: 14, weight: .medium))
        .frame(width: 44, height: 44) // Full touch target
}
```

### Color Perception in Context

Colors appear different depending on surroundings:
- A gray that looks right on white may look too light on a light-gray background
- Dark text on a vibrant background needs more weight to maintain readability
- Shadows on colored backgrounds should use the background's hue, not pure black

```swift
// Shadow on a blue card — tint the shadow
.shadow(color: Color.blue.opacity(0.3), radius: 8, x: 0, y: 4)

// Instead of pure black shadow
// .shadow(color: Color.black.opacity(0.15), radius: 8, x: 0, y: 4) // looks dirty on blue
```

---

## Phase 4: Build, Compare, Refine (The Loop)

This is the critical phase. You will iterate until the implementation is indistinguishable from the design.

### Step 1: Build and Capture

```bash
# Build and run the app
flowdeck run

# Start a UI session for continuous screenshots
flowdeck ui simulator session start -S "<simulator>" --json
# Parse JSON → save latest_screenshot path
```

Then use the Read tool on `latest_screenshot` to see the current UI.

### Step 2: Side-by-Side Comparison

With both the original design image and the implementation screenshot visible:

1. **Squint test** — Blur your vision. Do the two images have the same *weight distribution*? Same light/dark balance? Same visual rhythm?
2. **Scan top-to-bottom** — Check each element in order: position, size, color, typography
3. **Check edges** — Are margins consistent? Do elements align to the same grid?
4. **Check whitespace** — Is the breathing room between sections correct? Whitespace is as important as content.
5. **Check color temperature** — Does the overall tone match? Warm vs. cool, saturated vs. muted?

### Step 3: Document Discrepancies

Be specific. Not "spacing looks off" — instead:
```
// DISCREPANCIES FOUND:
// 1. Title top margin: implementation 52pt, design ~60pt → increase by 8pt
// 2. Card shadow: too heavy — reduce opacity from 0.12 to 0.08
// 3. Subtitle color: too dark — change from #555555 to #6B7280
// 4. Button corner radius: using circular, design shows continuous (squircle)
// 5. Section spacing between form and button: 16pt, design shows ~24pt
```

### Step 4: Fix One Thing at a Time

Make a single correction, rebuild, and re-check. Do not batch fixes — you need to see the effect of each change in isolation.

```bash
# After each code change:
flowdeck run
# Wait ~2 seconds, then read latest_screenshot to verify
```

### Step 5: Repeat Until Done

The loop ends when you cannot identify any visible difference between the design and the implementation at normal viewing distance (arm's length from screen).

---

## Phase 5: Responsive Verification

A pixel-perfect design on one device must remain proportionally correct across screen sizes.

### Strategy

Not everything should scale. Categorize each value:

| Category | Behavior | Example |
|----------|----------|---------|
| **Fixed** | Same on all devices | Button height (50pt), icon size (24pt), border width (1pt) |
| **Proportional** | Scales with screen width | Horizontal margins, card width |
| **Adaptive** | Changes at breakpoints | Grid columns (2 on SE, 3 on Pro Max), layout direction |
| **Content-driven** | Determined by content | Text wrapping, list item height |

### Verify on Key Sizes

Build and screenshot on at least:
1. **Smallest** — iPhone SE (375pt wide) — does content fit? Any overflow?
2. **Standard** — iPhone 16 (393pt wide) — the design target
3. **Largest** — iPhone 16 Pro Max (430pt wide) — does it stretch gracefully?

```bash
# Run on different simulators
flowdeck run -S "iPhone SE (3rd generation)"
flowdeck run -S "iPhone 16"
flowdeck run -S "iPhone 16 Pro Max"
```

### Common Responsive Issues

```swift
// AVOID: Fixed width that breaks on small screens
.frame(width: 350)

// PREFER: Proportional with max constraint
.frame(maxWidth: .infinity)
.padding(.horizontal, 20)

// AVOID: Fixed height for text containers
.frame(height: 100)

// PREFER: Let text drive height with padding
.padding(.vertical, 16)
```

---

## Phase 6: Dark Mode and Appearance

If the design includes dark mode variants, verify both. If not provided, at minimum ensure the implementation doesn't break in dark mode.

### Color Adaptation

```swift
// Adaptive colors that work in both modes
extension Color {
    static let cardBackground = Color(light: .white, dark: Color(hex: "#1C1C1E"))
    static let textPrimary = Color(light: Color(hex: "#1A1A1A"), dark: .white)
    static let divider = Color(light: Color(hex: "#E5E7EB"), dark: Color(hex: "#38383A"))
}
```

### Shadow Adaptation

Shadows are invisible on dark backgrounds. In dark mode, use lighter backgrounds or borders instead:
```swift
.shadow(color: colorScheme == .dark
    ? Color.clear
    : Color.black.opacity(0.08),
    radius: 8, x: 0, y: 2)
```

---

## Anti-Patterns (Never Do This)

### Layout Anti-Patterns
```swift
// NEVER: Default padding
.padding()
// ALWAYS: Explicit values
.padding(.horizontal, 20)

// NEVER: GeometryReader for simple layouts
GeometryReader { geo in
    content.frame(width: geo.size.width - 40)
}
// ALWAYS: Padding and flexible frames
content.padding(.horizontal, 20)

// NEVER: Spacer for precise spacing
VStack { item1; Spacer(); item2 } // Spacer fills all available space
// ALWAYS: Fixed spacers for precise gaps
VStack(spacing: 0) { item1; Spacer().frame(height: 16); item2 }

// NEVER: Magic numbers without context
.offset(y: -37)
// ALWAYS: Commented intent
.offset(y: -37) // Overlaps card by half button height (74pt / 2)
```

### Color Anti-Patterns
```swift
// NEVER: Named system colors for design-specific colors
.foregroundStyle(.gray) // Which gray? System gray changes across OS versions

// ALWAYS: Exact hex or design-token colors
.foregroundStyle(Color(hex: "#6B7280"))

// NEVER: Pure black text on white (too harsh)
.foregroundStyle(.black)

// USUALLY BETTER: Near-black for body text
.foregroundStyle(Color(hex: "#1A1A1A")) // Softer, professional
```

### Typography Anti-Patterns
```swift
// NEVER: Dynamic type styles when matching a fixed design
.font(.title) // Size varies with user settings

// WHEN MATCHING A DESIGN: Fixed size
.font(.system(size: 28, weight: .semibold))

// NOTE: For production apps, prefer Dynamic Type for accessibility.
// Use fixed sizes only when matching a specific design comp.
// Ideally, the design system maps to Dynamic Type text styles.
```

---

## Verification Checklist

Before declaring implementation complete, verify each category:

### Spatial Accuracy (within 1pt tolerance)
- [ ] Screen margins match on all edges
- [ ] Section spacing matches between all content blocks
- [ ] Element spacing matches within all sections
- [ ] Internal padding matches in all containers
- [ ] Content alignment matches (leading, center, trailing)

### Typography Accuracy
- [ ] All font sizes match
- [ ] All font weights match
- [ ] All text colors match
- [ ] Line heights match for multi-line text
- [ ] Letter spacing matches where specified
- [ ] Text alignment matches
- [ ] Text truncation behavior is correct

### Color Accuracy
- [ ] Background colors match
- [ ] All text colors match in context
- [ ] Interactive element colors match
- [ ] Border/divider colors and opacity match
- [ ] Gradient directions and stops match (if applicable)

### Shape and Effect Accuracy
- [ ] Corner radii match (and use .continuous style)
- [ ] Border widths and colors match
- [ ] Shadow offset, radius, and opacity match
- [ ] Blur effects match (if applicable)

### Perceptual Accuracy
- [ ] Squint test passes — overall weight and rhythm match
- [ ] No element feels "off" even if measurements are correct
- [ ] Whitespace feels balanced
- [ ] Visual hierarchy reads correctly

### Responsive Behavior
- [ ] Smallest device: no overflow, no clipping
- [ ] Standard device: matches design exactly
- [ ] Largest device: proportions remain balanced

---

## Hex Color Helper

If the project doesn't have a hex color initializer, add one:

```swift
extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet(charactersIn: "#"))
        let scanner = Scanner(string: hex)
        var rgbValue: UInt64 = 0
        scanner.scanHexInt64(&rgbValue)

        let r = Double((rgbValue & 0xFF0000) >> 16) / 255.0
        let g = Double((rgbValue & 0x00FF00) >> 8) / 255.0
        let b = Double((rgbValue & 0x0000FF)) / 255.0

        self.init(red: r, green: g, blue: b)
    }
}
```

---

## Integration with FlowDeck Workflow

This skill works hand-in-hand with FlowDeck:

1. **`flowdeck run`** — Build and launch to see the implementation
2. **`flowdeck ui simulator session start`** — Continuous screenshots for comparison
3. **Read `latest_screenshot`** — Your eyes on the UI after every change
4. **Read `latest_tree`** — Verify accessibility labels and element structure
5. **Iterate** — Change code, rebuild, re-read screenshot, compare with design
6. **`flowdeck run -S "<device>"`** — Test on different screen sizes

The build-compare-refine loop should be tight: change one thing, verify immediately, move to the next discrepancy. Never batch multiple changes before checking.
