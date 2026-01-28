# Orklys Design System

A comprehensive design system for building consistent, reusable components across the Orklys codebase.

---

## 1. Design Philosophy

### Aesthetic Direction

**Scandinavian Trust** — Clean, honest, and community-rooted. The design communicates reliability without corporate coldness. It feels like a handshake from a neighbor who happens to understand renewable energy.

### Core Principles

- **Clarity over cleverness** — Information hierarchy is paramount. Users should instantly understand what they're looking at.
- **Warmth through restraint** — The cream backgrounds and forest greens feel natural, not sterile. White space breathes.
- **Local authenticity** — Design choices reinforce "this is your community, your energy" — not a faceless corporation.
- **Component reuse** — Always check for existing components before creating new ones.

### Tone

Professional but approachable. No jargon. Direct language paired with generous whitespace. The design should feel like reading a well-organized document from someone you trust.

---

## 2. Color System

### Primary Palette (from `globals.css`)

```css
--color-primary: oklch(0.4194 0.0967 156.33); /* #0a5c36 - Primary green */
--color-primary-alt: oklch(79.2% 0.206 152.21); /* Lighter green variant */
--color-accent: oklch(
  93.94% 0.093 161.89
); /* #b2ffd8 - Light green (IconCircle bg) */
--color-neutral: oklch(
  96.49% 0.005 78.3
); /* #f5f2eb - Warm gray (section bg) */
--color-background: oklch(96.49% 0.005 78.3); /* Off-white */
--color-black: #02120a; /* Near-black for text */
```

### Gray Scale

```css
--gray-100: #f8f8f8; /* Subtle backgrounds */
--gray-200: #e8e8e8; /* Borders, dividers */
--gray-300: #d0d0d0; /* Input borders */
--gray-400: #a0a0a0; /* Placeholder text */
--gray-500: #707070; /* Secondary text */
--gray-600: #505050; /* Body text descriptions */
```

### Usage Rules

- **Primary green** → CTAs, key data points, headings, brand moments
- **Accent green** → IconCircle backgrounds, badges, soft highlights (never for text)
- **Neutral** → Alternating section backgrounds (never stack two neutral sections)
- **Gray-500/600** → Body copy; black only for headlines

---

## 3. Typography System

### Font Stack (defined in `app/layout.tsx` and `globals.css`)

```css
--font-family-display: "Space Grotesk"; /* Headings, CTAs, display text */
--font-family-body: "DM Sans"; /* Body text */
```

### Font Usage Rules

| Element           | Font          | Class                               |
| ----------------- | ------------- | ----------------------------------- |
| Headings (h1-h6)  | Space Grotesk | Applied via `Heading` component     |
| CTA Buttons       | Space Grotesk | `.font-display` in Button component |
| Body text         | DM Sans       | Default body font                   |
| Display/hero text | Space Grotesk | `.font-display` utility             |

### MANDATORY: Typography Components

**Always use `Heading` and `Text` components** — ESLint rules enforce this.

```tsx
import { Heading, Text } from "@/components/shared/Typography";

// Headings
<Heading level={1} size="5xl" weight="bold">Page Title</Heading>
<Heading level={2} size="4xl">Section Title</Heading>
<Heading level={3}>Card Title</Heading>

// Body text
<Text variant="description" size="lg">Description text</Text>
<Text variant="muted" size="sm">Helper text</Text>
<Text size="base">Default body text</Text>
```

### Heading Props

| Prop      | Values                                                  | Default  |
| --------- | ------------------------------------------------------- | -------- |
| `level`   | 1-6                                                     | 3        |
| `size`    | default, 2xl, 3xl, 4xl, 5xl, 6xl, 7xl, 8xl, 9xl         | default  |
| `variant` | default, inverted, muted, primary, destructive, display | default  |
| `weight`  | normal, medium, semibold, bold                          | semibold |
| `align`   | left, center, right                                     | left     |

### Text Props

| Prop      | Values                                                                                              | Default |
| --------- | --------------------------------------------------------------------------------------------------- | ------- |
| `size`    | xs, sm, base, lg, xl, 2xl, 3xl, 4xl, 5xl, 6xl                                                       | base    |
| `variant` | default, white, muted, description, helper, primary, secondary, destructive, success, warning, info | default |
| `weight`  | normal, medium, semibold, bold                                                                      | normal  |
| `as`      | p, span, div                                                                                        | p       |
| `spacing` | normal, tight, relaxed                                                                              | normal  |

### Convenience Components

```tsx
import {
  PageTitle,
  SectionTitle,
  SubsectionTitle,
  DisplayText,
} from "@/components/shared/Typography";
import {
  DescriptionText,
  HelperText,
  MutedText,
  MetricText,
} from "@/components/shared/Typography";
```

---

## 4. Component Library

### 4.1 MANDATORY Components (Always Use These)

#### IconCircle — For ALL Icon Displays

**Location:** `@/components/shared/IconCircle`

```tsx
import { IconCircle } from "@/components/shared/IconCircle";
import { Zap, User, Mail } from "lucide-react";

// Default: accent background, primary icon
<IconCircle Icon={Zap} size="md" />

// Inverse: primary background, light icon
<IconCircle Icon={Zap} size="lg" inverseColors />

// Custom background
<IconCircle Icon={Zap} backgroundColor="#f0f0f0" />
```

| Prop              | Values                                    | Default  |
| ----------------- | ----------------------------------------- | -------- |
| `Icon`            | LucideIcon                                | required |
| `size`            | sm (w-8), md (w-12), lg (w-16), xl (w-20) | lg       |
| `inverseColors`   | boolean                                   | false    |
| `backgroundColor` | string                                    | -        |
| `iconClassName`   | string                                    | -        |

**NEVER style Lucide icons directly:**

```tsx
// ❌ WRONG
<Zap className="h-6 w-6 text-primary bg-accent rounded-full p-2" />

// ✅ CORRECT
<IconCircle Icon={Zap} size="md" />
```

#### LabeledIconInput — For ALL Form Inputs

**Location:** `@/components/shared/LabeledIconInput`

```tsx
import LabeledIconInput from "@/components/shared/LabeledIconInput";
import { User, Mail, Briefcase } from "lucide-react";

<LabeledIconInput
  id="fullName"
  label="Full Name"
  icon={User}
  value={name}
  onChange={setName}
  placeholder="Enter your name"
  required
/>

// With validation
<LabeledIconInput
  id="email"
  label="Email"
  icon={Mail}
  value={email}
  onChange={setEmail}
  type="email"
  required
  showValidation={submitted}
  isValid={isEmailValid}
  errorMessage="Please enter a valid email"
/>
```

| Prop             | Values                  | Default  |
| ---------------- | ----------------------- | -------- |
| `id`             | string                  | required |
| `label`          | string                  | -        |
| `icon`           | LucideIcon              | required |
| `value`          | string                  | required |
| `onChange`       | (value: string) => void | required |
| `placeholder`    | string                  | -        |
| `type`           | string                  | "text"   |
| `height`         | default, large          | default  |
| `required`       | boolean                 | false    |
| `showValidation` | boolean                 | false    |
| `isValid`        | boolean                 | true     |
| `errorMessage`   | string                  | -        |

**NEVER use raw Input:**

```tsx
// ❌ WRONG
<div className="space-y-2">
  <label>Name</label>
  <Input placeholder="Enter name" />
</div>

// ✅ CORRECT
<LabeledIconInput id="name" icon={User} label="Name" value={name} onChange={setName} />
```

#### LabeledIconTextarea — For ALL Textareas

**Location:** `@/components/shared/LabeledIconTextarea`

Same API as LabeledIconInput, for multi-line text.

### 4.2 Layout Components

#### Dashboard Pages

**DashboardPageContainer** — Wrapper for dashboard content

```tsx
import DashboardPageContainer from "@/components/shared/DashboardPageContainer";

<DashboardPageContainer maxWidth="5xl">
  {/* Page content */}
</DashboardPageContainer>;
```

| Prop        | Values                                   | Default |
| ----------- | ---------------------------------------- | ------- |
| `maxWidth`  | sm, md, lg, xl, 2xl, 4xl, 6xl, 7xl, full | 5xl     |
| `noPadding` | boolean                                  | false   |

**DashboardPageHeader** — Page title and description

```tsx
import DashboardPageHeader from "@/components/shared/DashboardPageHeader";

<DashboardPageHeader
  title="Team Members"
  description="Manage your team members and their roles"
  actions={<Button>Add Member</Button>}
/>;
```

**TabbedPageLayout** — Dashboard pages with tabs

```tsx
import TabbedPageLayout from "@/components/features/dashboard/shared/TabbedPageLayout";
import { Users, FileText } from "lucide-react";

const tabs = [
  {
    key: "members",
    label: "Members",
    icon: Users,
    content: (community) => <MembersTab community={community} />,
  },
  {
    key: "documents",
    label: "Documents",
    icon: FileText,
    lazyContent: () => import("./DocumentsTab"),
  },
];

<TabbedPageLayout title="Team" description="Manage your team" tabs={tabs} />;
```

#### Public Page Sections

Use this pattern for all public-facing pages:

```tsx
<section className="py-24 bg-white">
  <div className="max-w-[1400px] mx-auto px-4 md:px-8">
    {/* Section badge (optional) */}
    <div className="text-center mb-12">
      <span className="inline-block px-4 py-2 bg-white border border-gray-300 rounded-full font-display text-[0.7rem] font-semibold tracking-[0.1em] uppercase">
        {badge}
      </span>
      <Heading
        level={2}
        size="5xl"
        weight="bold"
        align="center"
        className="mt-4"
      >
        {title}
      </Heading>
      <Text
        variant="description"
        size="lg"
        align="center"
        className="mt-4 max-w-2xl mx-auto"
      >
        {subtitle}
      </Text>
    </div>

    {/* Grid content */}
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
      {/* Cards */}
    </div>
  </div>
</section>
```

### 4.3 Display Components

#### InsightCard — Metrics and Data Display

```tsx
import { InsightCard } from "@/components/shared/InsightCard";
import { TrendingUp } from "lucide-react";

<InsightCard
  icon={TrendingUp}
  value="2,450"
  subtitle="kWh"
  description="Monthly energy production"
  variant="success"
/>

// Simple mode (for values/concepts)
<InsightCard
  icon={Heart}
  title="Community"
  description="Built on trust and collaboration"
  variant="simple"
/>
```

#### ActionCard — Clickable Action Items

```tsx
import { ActionCard } from "@/components/shared/ActionCard";
import { Calendar } from "lucide-react";

<ActionCard
  icon={Calendar}
  title="Schedule Consultation"
  description="Book a call with our team"
  cta="Book Now"
  onClick={() => setModalOpen(true)}
  isCompleted={hasBooked}
/>;
```

### 4.4 UI Components (shadcn/ui)

Import from `@/components/ui/`:

- `Button` — Primary actions
- `Card`, `CardHeader`, `CardContent`, `CardFooter` — Container components
- `Dialog`, `AlertDialog` — Modals
- `Input`, `Textarea` — Base inputs (use Labeled variants instead)
- `Label` — Form labels
- `Switch` — Toggle switches
- `Badge` — Status indicators
- `Table` — Data tables
- `Accordion` — Collapsible sections
- `Skeleton` — Loading states
- `Tooltip` — Hover information

---

## 5. Layout Patterns

### 5.1 Public Pages

**Container:**

```tsx
<div className="max-w-[1400px] mx-auto px-4 md:px-8">
```

**Section Padding:**

```tsx
<section className="py-24">
```

**Background Alternation:**

- white → neutral → white → neutral
- **Never stack two sections with the same background**

**Responsive Grid:**

```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
```

### 5.2 Dashboard Pages

Always use the layout components:

```tsx
<DashboardPageContainer maxWidth="5xl">
  <DashboardPageHeader title="..." description="..." />
  {/* Page content */}
</DashboardPageContainer>
```

For tabbed pages:

```tsx
<TabbedPageLayout title="..." description="..." tabs={[...]} />
```

### 5.3 Card Pattern

**Standard Card with Hover:**

```tsx
<div
  className={cn(
    "bg-white rounded-2xl shadow-md p-6",
    "transition-all duration-300 hover:-translate-y-1 hover:shadow-lg"
  )}
>
  <IconCircle Icon={icon} size="md" className="mb-4" />
  <Heading level={3} className="mb-2">
    {title}
  </Heading>
  <Text variant="description">{description}</Text>
</div>
```

**Interactive Card:**

```tsx
<Card className="card card-interactive p-6">
  {/* Uses card-interactive utility from globals.css */}
</Card>
```

**NEVER use flat cards:**

```tsx
// ❌ WRONG
<div className="bg-white rounded-lg p-4">

// ✅ CORRECT
<div className="bg-white rounded-2xl shadow-md p-6 transition-all duration-300 hover:-translate-y-1 hover:shadow-lg">
```

---

## 6. Animation Patterns

### Entry Animations (defined in `globals.css`)

**fadeInUp** — Default entry animation

```tsx
<div className="animate-fadeInUp">{content}</div>
```

**Staggered Animation:**

```tsx
{
  items.map((item, index) => (
    <div
      key={item.id}
      className="animate-fadeInUp"
      style={{ animationDelay: `${index * 0.1}s` }}
    >
      {/* Card content */}
    </div>
  ));
}
```

**slideInRight** — For side content

```tsx
<div className="animate-slideInRight" style={{ animationDelay: "0.2s" }}>
  {content}
</div>
```

### Hover Effects

**Button lift:**

```tsx
className = "hover:-translate-y-0.5 transition-transform";
```

**Card lift:**

```tsx
className = "transition-all duration-300 hover:-translate-y-1 hover:shadow-lg";
```

**Image zoom (in card):**

```tsx
<div className="overflow-hidden rounded-t-2xl">
  <img className="transition-transform duration-300 group-hover:scale-[1.03]" />
</div>
```

---

## 7. Anti-Patterns

### ❌ Direct Icon Styling

```tsx
// WRONG
<Zap className="h-6 w-6 text-primary bg-accent rounded-full p-2" />

// CORRECT
<IconCircle Icon={Zap} size="md" />
```

### ❌ Raw HTML Tags

```tsx
// WRONG
<h2 className="text-2xl font-bold">Title</h2>
<p className="text-gray-600">Description</p>

// CORRECT
<Heading level={2} size="4xl" weight="bold">Title</Heading>
<Text variant="description">Description</Text>
```

### ❌ Raw Input/Textarea

```tsx
// WRONG
<label>Name</label>
<Input placeholder="Name" />

// CORRECT
<LabeledIconInput id="name" icon={User} label="Name" value={name} onChange={setName} />
```

### ❌ Flat Cards

```tsx
// WRONG
<div className="bg-white rounded-lg p-4">

// CORRECT
<div className="bg-white rounded-2xl shadow-md p-6 transition-all duration-300 hover:-translate-y-1 hover:shadow-lg">
```

### ❌ Stacking Same Backgrounds

```tsx
// WRONG
<section className="bg-neutral">...</section>
<section className="bg-neutral">...</section>

// CORRECT
<section className="bg-neutral">...</section>
<section className="bg-white">...</section>
```

### ❌ Creating New Components Without Checking

Always search `components/shared/` and `components/ui/` before creating new components.

---

## 8. Reference Implementations

| Context               | Reference Path                                 |
| --------------------- | ---------------------------------------------- |
| Public page sections  | `components/features/community-page/sections/` |
| Dashboard tabs/pages  | `components/features/dashboard/organiser/`     |
| Onboarding forms      | `components/features/onboarding/organiser/`    |
| Landing page sections | `components/features/landing/`                 |
| Shared components     | `components/shared/`                           |

### Best Examples

- **BenefitsSection** — Card grid with animations: `community-page/sections/BenefitsSection.tsx`
- **TeamMembersTab** — Dashboard CRUD with modal: `dashboard/organiser/Team/TeamMembersTab.tsx`
- **TabbedPageLayout usage** — `dashboard/projects/page.tsx`

---

## 9. Icon Mapping for Forms

Standard icons to use with `LabeledIconInput`:

| Field Type      | Icon         | Import         |
| --------------- | ------------ | -------------- |
| Name            | `User`       | `lucide-react` |
| Email           | `Mail`       | `lucide-react` |
| Title/Role      | `Briefcase`  | `lucide-react` |
| Organization    | `Building2`  | `lucide-react` |
| Description     | `FileText`   | `lucide-react` |
| Phone           | `Phone`      | `lucide-react` |
| Address         | `MapPin`     | `lucide-react` |
| Website         | `Globe`      | `lucide-react` |
| Upload          | `Upload`     | `lucide-react` |
| Password        | `Lock`       | `lucide-react` |
| Search          | `Search`     | `lucide-react` |
| Date            | `Calendar`   | `lucide-react` |
| Amount/Currency | `DollarSign` | `lucide-react` |

---

## 10. Spacing & Border Radius

### Spacing (8-point grid)

| Token | Value | Use                      |
| ----- | ----- | ------------------------ |
| xs    | 4px   | Badge internals          |
| sm    | 8px   | Icon gaps                |
| md    | 16px  | Default padding          |
| lg    | 24px  | Card padding             |
| xl    | 32px  | Container padding        |
| 2xl   | 48px  | Section internal         |
| 3xl   | 64px  | Major section            |
| 4xl   | 96px  | Section vertical (py-24) |

### Border Radius

| Token | Value  | Use                     |
| ----- | ------ | ----------------------- |
| sm    | 4px    | Badges, small tags      |
| md    | 8px    | Inputs, small cards     |
| lg    | 12px   | Medium cards, FAQ       |
| xl    | 20px   | Large cards, modals     |
| 2xl   | 16px   | Standard cards          |
| full  | 9999px | Pills, buttons, avatars |

**Standard card:** `rounded-2xl`
**Buttons:** `rounded-full` (pill shape)
**Inputs:** `rounded-xl`

---

## 11. Shadows

```css
--shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05); /* Subtle, inputs */
--shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08); /* Cards at rest */
--shadow-lg: 0 8px 30px rgba(0, 0, 0, 0.12); /* Cards on hover */
--shadow-xl: 0 20px 50px rgba(0, 0, 0, 0.15); /* Hero, modals */
```

**Card pattern:** `shadow-md` → `hover:shadow-lg`

---

## 12. Checklist for New Components

Before creating any component:

1. [ ] Check `components/shared/` for existing components
2. [ ] Check `components/ui/` for base shadcn components
3. [ ] Check similar features in `components/features/`
4. [ ] Use `IconCircle` for all icons
5. [ ] Use `Heading` and `Text` for all typography
6. [ ] Use `LabeledIconInput` for all form inputs
7. [ ] Apply standard card pattern with hover effects
8. [ ] Add entry animations for public pages
9. [ ] Follow barrel export pattern for feature folders
