# Sydstevns Energifællesskab — Design Reference

A design system reference for building consistent subpages and components.

---

## 1. Design Philosophy

### Aesthetic Direction

**Scandinavian Trust** — Clean, honest, and community-rooted. The design communicates reliability without corporate coldness. It feels like a handshake from a neighbor who happens to understand renewable energy.

### Core Principles

- **Clarity over cleverness** — Information hierarchy is paramount. Users should instantly understand what they're looking at.
- **Warmth through restraint** — The cream backgrounds and forest greens feel natural, not sterile. White space breathes.
- **Local authenticity** — Design choices reinforce "this is your community, your energy" — not a faceless corporation.

### Tone

Professional but approachable. No jargon. Direct language paired with generous whitespace. The design should feel like reading a well-organized document from someone you trust.

---

## 2. Color System

### Primary Palette

```css
--forest-green: #0a5c36; /* Primary actions, key highlights */
--forest-green-light: #3d7a4d; /* Hover states, gradients */
--forest-green-dark: #1d3a2d; /* Dark backgrounds, footer CTAs */
--forest-green-muted: #4a7c59; /* Secondary elements */
--accent-green: #b8d4be; /* Badges, soft highlights, borders */
```

### Neutral Palette

```css
--cream: #f5f2eb; /* Warm section backgrounds */
--cream-dark: #ebe6db; /* Hover on cream */
--white: #ffffff; /* Cards, primary backgrounds */
--black: #1a1a1a; /* Headlines, primary text */
```

### Gray Scale

```css
--gray-100: #f8f8f8; /* Subtle backgrounds (FAQ, chat) */
--gray-200: #e8e8e8; /* Borders, dividers */
--gray-300: #d0d0d0; /* Input borders, subtle outlines */
--gray-400: #a0a0a0; /* Placeholder text, muted labels */
--gray-500: #707070; /* Secondary text, descriptions */
--gray-600: #505050; /* Body text, nav links */
```

### Usage Rules

- **Forest green** is reserved for primary CTAs, key data points, and brand moments
- **Cream** alternates with white for section rhythm — never stack two cream sections
- **Gray-500/600** for body copy; black only for headlines
- **Accent green** for badges, source tags, and decorative borders (never for text)

---

## 3. Typography

### Font Stack

```css
--font-display: "Space Grotesk", sans-serif; /* Headlines, buttons, labels */
--font-body: "DM Sans", sans-serif; /* Body text, paragraphs */
```

### Type Scale

| Element       | Font          | Size                        | Weight  | Letter-spacing | Line-height |
| ------------- | ------------- | --------------------------- | ------- | -------------- | ----------- |
| Hero title    | Space Grotesk | clamp(2.5rem, 5vw, 4rem)    | 700     | -0.02em        | 1.1         |
| Section title | Space Grotesk | clamp(1.75rem, 3vw, 2.5rem) | 700     | -0.01em        | 1.15        |
| Card title    | Space Grotesk | 0.9-1.1rem                  | 700     | 0.02em         | —           |
| Body large    | DM Sans       | 1.05-1.15rem                | 400     | —              | 1.6-1.7     |
| Body default  | DM Sans       | 0.95-1rem                   | 400     | —              | 1.6         |
| Body small    | DM Sans       | 0.85-0.9rem                 | 400/500 | —              | 1.6         |
| Badge/Label   | Space Grotesk | 0.7rem                      | 600-700 | 0.05-0.1em     | —           |
| Button        | Space Grotesk | 0.85rem                     | 600     | 0.02em         | —           |

### Typography Rules

- Headlines use **Space Grotesk** with tight letter-spacing (-0.01 to -0.02em)
- Body text uses **DM Sans** at comfortable reading sizes (0.95-1.05rem)
- Labels and badges are UPPERCASE with loose tracking (0.05-0.1em)
- Never go below 0.75rem for any text
- Maximum line length: ~65-75 characters for readability

---

## 4. Spacing System

### Base Scale (8-point grid)

```css
--space-xs: 0.25rem; /* 4px — tight padding, badge internals */
--space-sm: 0.5rem; /* 8px — icon gaps, tight margins */
--space-md: 1rem; /* 16px — default padding, component gaps */
--space-lg: 1.5rem; /* 24px — card padding, section gaps */
--space-xl: 2rem; /* 32px — container padding, larger gaps */
--space-2xl: 3rem; /* 48px — section internal spacing */
--space-3xl: 4rem; /* 64px — major section spacing */
--space-4xl: 6rem; /* 96px — section vertical padding */
```

### Section Rhythm

- Major sections: `padding: var(--space-4xl) 0` (96px top/bottom)
- Container max-width: `1400px` (1100px for narrow content like FAQ)
- Container horizontal padding: `var(--space-xl)` (32px)

### Component Spacing Patterns

- **Cards**: `padding: var(--space-lg)` to `var(--space-xl)` (24-32px)
- **Buttons**: `padding: var(--space-md) var(--space-xl)` (16px 32px)
- **Form inputs**: `padding: var(--space-md) var(--space-lg)` (16px 24px)
- **Grid gaps**: `var(--space-xl)` (32px) between cards

---

## 5. Border Radius

```css
--radius-sm: 4px; /* Badges, small tags */
--radius-md: 8px; /* Inputs, small cards */
--radius-lg: 12px; /* Medium cards, FAQ items */
--radius-xl: 20px; /* Large cards, modals */
--radius-full: 9999px; /* Pills, buttons, avatars */
```

### Usage

- **Buttons**: Always `radius-full` (pill shape)
- **Cards**: `radius-xl` (20px) for prominent cards, `radius-lg` (12px) for smaller
- **Badges**: `radius-sm` (4px) for rectangular, `radius-full` for pill badges
- **Images in cards**: Match parent card radius or use `radius-lg`
- **Avatars**: Always `radius-full` (circle)

---

## 6. Shadow System

```css
--shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05); /* Subtle depth, inputs */
--shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08); /* Cards at rest */
--shadow-lg: 0 8px 30px rgba(0, 0, 0, 0.12); /* Cards on hover */
--shadow-xl: 0 20px 50px rgba(0, 0, 0, 0.15); /* Hero card, modals */
```

### Usage

- Cards at rest: `shadow-md`
- Cards on hover: `shadow-lg`
- Hero/featured elements: `shadow-xl`
- Floating elements (chat widget): `shadow-xl`

---

## 7. Component Patterns

### Buttons

**Primary (Forest Green)**

```css
background: var(--forest-green);
color: var(--white);
border: 2px solid var(--forest-green);
border-radius: var(--radius-full);
padding: var(--space-md) var(--space-xl);
font: 600 0.85rem/1 "Space Grotesk";
/* Hover: darker green, translateY(-2px), shadow-md */
```

**Outline (Black)**

```css
background: transparent;
color: var(--black);
border: 2px solid var(--black);
/* Hover: fills black, text white */
```

**Light (White on dark)**

```css
background: var(--white);
color: var(--forest-green-dark);
border: 2px solid var(--white);
/* Hover: cream background */
```

**Arrow animation**: On hover, arrow (`→`) translates 4px right

### Cards

**Standard Card Structure**

```
┌─────────────────────────────┐
│  [Image area with badge]    │  aspect-ratio: 16/10
├─────────────────────────────┤
│  Title (Space Grotesk)      │
│  Subtitle/location          │
│  ─────────────────────────  │  border-top divider
│  Stats row (3-col grid)     │
└─────────────────────────────┘
```

- Background: white
- Border-radius: `radius-xl`
- Shadow: `shadow-md` → `shadow-lg` on hover
- Image hover: `scale(1.03)` with overflow hidden
- Badge position: top-right of image, accent-green background

### Testimonial Cards

- Quote text with decorative `"` (3rem, accent-green, 50% opacity)
- Author row: avatar (44px circle) + name + role
- Featured variant: larger padding, larger quote text

### Team Cards

- Centered layout
- Photo: 100px circle with 3px accent-green border
- Name, role (forest-green color), bio paragraph

### Section Badge

```css
display: inline-block;
padding: var(--space-sm) var(--space-md);
background: var(--white);
border: 1px solid var(--gray-300);
border-radius: var(--radius-full);
font: 600 0.7rem "Space Grotesk";
letter-spacing: 0.1em;
text-transform: uppercase;
```

### Avatar Stack

- Avatars: 36px circles with 3px white border
- Overlap: `margin-left: -12px` (except first)
- Wrapper: gray-100 background, pill shape

### FAQ Accordion

- Container: gray-100 background, radius-lg
- Question: full-width button, flex space-between
- Icon: 20px `+` that rotates 45° to `×` when open
- Answer: max-height animation (0 → 200px)

---

## 8. Layout Patterns

### Grid Systems

```css
/* 2-column (hero, about) */
grid-template-columns: 1fr 1fr;
gap: var(--space-3xl); /* 64px */

/* 3-column (benefits, team) */
grid-template-columns: repeat(3, 1fr);
gap: var(--space-xl); /* 32px */

/* 4-column (steps, quick facts) */
grid-template-columns: repeat(4, 1fr);
gap: var(--space-xl);

/* Testimonials (asymmetric) */
grid-template-columns: 1fr 1.5fr 1fr;
```

### Section Structure

```
┌────────────────────────────────────────┐
│            [Section Badge]             │
│           [Section Title]              │
│         [Optional subtitle]            │
│                                        │
│   ┌──────┐  ┌──────┐  ┌──────┐         │
│   │ Card │  │ Card │  │ Card │         │
│   └──────┘  └──────┘  └──────┘         │
└────────────────────────────────────────┘
```

### Breakpoints

```css
@media (max-width: 1200px) /* Tablet landscape */ @media (max-width: 992px) /* Tablet portrait */ @media (max-width: 768px) /* Mobile landscape */ @media (max-width: 480px); /* Mobile portrait */
```

### Responsive Behavior

- **1200px**: Hero goes single column, 4-col → 2-col
- **992px**: 3-col → 1-col, sidebar layouts stack
- **768px**: Nav links hide, buttons go full-width, all grids → 1-col
- **480px**: Reduced spacing scale, smaller headlines

---

## 9. Motion & Transitions

### Timing

```css
--transition-fast: 150ms ease; /* Hovers, color changes */
--transition-base: 250ms ease; /* Most interactions */
--transition-slow: 400ms ease; /* Image zooms, reveals */
```

### Entrance Animations

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideInRight {
  from {
    opacity: 0;
    transform: translateX(30px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
```

### Stagger Pattern

- Cards animate with 0.1s delay increments
- Hero content: 0s, Hero card: 0.2s
- Cards in grid: 0.1s, 0.2s, 0.3s...

### Hover States

- **Buttons**: `translateY(-2px)`, shadow increase
- **Cards**: `translateY(-4px)`, shadow-md → shadow-lg
- **Images in cards**: `scale(1.03)` with overflow hidden
- **Links**: color transition to forest-green or black

---

## 10. Icon Style

### Characteristics

- Stroke-based (not filled)
- Stroke width: 2px
- Size: 18-24px typical, 48px for feature icons
- Color: inherits from parent or forest-green

### Icon Containers

```css
/* Feature icon box */
width: 48px;
height: 48px;
background: var(--accent-green);
border-radius: var(--radius-lg);
/* Icon inside: 24px, forest-green-dark color */
```

---

## 11. Special Elements

### CTA Section (Dark)

```css
background: linear-gradient(rgba(29, 58, 45, 0.9), rgba(29, 58, 45, 0.95)),
  url("background-image.jpg");
background-size: cover;
```

- Centered text
- White/light buttons
- Title: clamp(2rem, 4vw, 3rem)

### Chat Widget

- Fixed bottom-right position
- Forest-green pill button with icon + text
- Modal: 450px max-width, slides up from bottom-right
- Message bubbles: white (assistant) / forest-green (user)

### Navigation

- Fixed top, white background
- Logo left, links center, CTA right
- CTA button: smaller padding, radius-full
- Links hide on mobile (hamburger not implemented in current version)

---

## 12. Content Patterns

### Data Display

- Large numbers: Space Grotesk, 700 weight, forest-green color
- Labels below: 0.85-0.9rem, gray-500

### Dividers

- `border-top: 1px solid var(--gray-200)`
- Used in: project stats, partners section, financials rows

### Progress Bars

```css
height: 8px;
background: var(--gray-200);
border-radius: var(--radius-full);
/* Fill: gradient from forest-green to forest-green-light */
```

---

## 13. Do's and Don'ts

### Do

- Use generous whitespace between sections
- Keep text lines under 70 characters
- Use forest-green sparingly for emphasis
- Maintain consistent card shadow/radius patterns
- Animate on scroll-triggered reveals

### Don't

- Stack multiple sections with same background color
- Use forest-green for large text blocks
- Mix rounded and sharp corners in same context
- Over-animate (keep it subtle and purposeful)
- Use pure black (#000000) — always use #1a1a1a

---

## 14. File Reference

When implementing as React components, these are the key structural patterns:

```
<PageSection background="white|cream|dark">
  <SectionHeader badge="LABEL" title="..." subtitle="..." />
  <Grid columns={2|3|4}>
    <Card variant="standard|testimonial|team|benefit" />
  </Grid>
</PageSection>
```

Key reusable components to extract:

- `Button` (primary, outline, light, dark variants)
- `Card` (multiple variants)
- `SectionBadge`
- `AvatarStack`
- `FAQ Accordion`
- `ChatWidget`
- `StatDisplay`
- `IconBox`
