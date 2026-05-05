# Attention Style Guide

Attention's visual identity emphasizes AI-forward sophistication, clarity, and modern B2B SaaS confidence — anchored by a near-black base, generous whitespace, and a signature violet accent with subtle multi-hue gradients.

## 1. Color Palette
### Primary Colors
Violet (Primary UI Accent / Brand CTA)
Default: #8F78FF (Signature Violet — primary buttons, key accents)
Hover/Dark: #523DBC (Deep Violet — primary button hover state)
Pale: #E4DCFF (Pale Violet — subtle highlights / hover backgrounds)
Translucent: rgba(143, 120, 255, 0.24) — for soft glow / surface gradients

Near-Black (Brand Foundation)
Default: #0D0E16 (Brand Primary — logos, dark surfaces, tertiary button text)
Deep: #141025 (Primary Text — headings & body copy)
Translucent Border: rgba(13, 14, 22, 0.08) — #0D0E1614, hairline dividers

### Neutral Colors
Gray (Text & UI Elements)
Primary Text: #141025 (Near-Black — headlines, body)
Secondary Text: #666666 (Medium Gray — captions, supporting copy)
Tertiary Text: #BFBFBF (Light Gray — placeholders, disabled states)
Border: rgba(20, 16, 37, 0.08) — #14102514, soft outlines on light surfaces

White & Surface Tones
Surface 1: #FFFFFF (White — primary background, cards)
Surface 2: #F9F9FB (Off-White — alternating sections, subtle layering)
Surface 3: #F3F3F7 (Pale Gray — input chips, muted panels)
Overlay: rgba(0, 0, 0, 0.25) — #00000040, modals & image overlays

### Functional / Gradient Accent Colors
Attention's signature visual flourish is a tri-color gradient palette used for hero text gradients, glow halos behind cards, and decorative surface washes.

Gradient Blue: #5C82FF (text gradient stop / surface glow #688BFF at 24% alpha → #688BFF3D)
Gradient Magenta: #CE51E4 (text gradient stop / surface glow #FF71FD at 24% alpha → #FF71FD3D)
Gradient Amber: #FFB255 (text gradient stop / surface glow #FFC978 at 24% alpha → #FFC9783D)

Status (inferred from supporting palette):
Success: #5C82FF / Light: #E4F5FF
Warning: #FFB255 / Light: #FFC978
Error: #EA384C / Light: #FFCFC0
Info: #5C82FF / Light: #C0DCE4

**Emphasis on Layered Light Surfaces:** Attention uses near-white surface tones (#F9F9FB, #F3F3F7) and very low-alpha gradient washes (3D hex / 24% alpha) to create depth without heavy color blocking. The violet primary is reserved for moments of action, keeping the canvas calm and the CTA loud.

## 2. Typography
### Font Family
Title Font: Hanken Grotesk (`Hankengrotesk, sans-serif`) — geometric, slightly humanist sans for headlines
Body Font: Uncut Sans (`Uncutsans, sans-serif`) — neutral, highly legible workhorse for paragraphs and UI
Fallback stack: sans-serif

### Heading Styles
Title XXL: 5rem (80px), font-family Hanken Grotesk, line-height 1.1em — hero display
Title XL: 3.5rem (56px), Hanken Grotesk, line-height 1.1em — section heroes
Title L: 3rem (48px), Hanken Grotesk, line-height 1.1em — major section heads
Title M: 2rem (32px), Hanken Grotesk, line-height 1.1em — sub-sections
Title S: 1.375rem (22px), Hanken Grotesk, line-height 1.1em — card titles, eyebrows

### Body Styles
Body L: 1.25rem (20px), Uncut Sans, line-height 1.4em — lead paragraphs
Body M: 1.125rem (18px), Uncut Sans, line-height 1.4em — default body
Body S: 1rem (16px), Uncut Sans, line-height 1.4em — secondary copy
Body XS: 0.875rem (14px), Uncut Sans, line-height 1.4em — captions / fine print

### Label & Link Styles
Label M: 1.25rem — pill labels, tags
Label S: 0.875rem — compact tags
Link M: 1.125rem — inline links
Link S: 1rem — secondary links
Link Nav: 1rem — top navigation

### Text Colors
Primary text: #141025 (Near-Black)
Secondary text: #666666 (Medium Gray)
Tertiary text: #BFBFBF (Light Gray)
Gradient text effect: linear-gradient(#5C82FF → #CE51E4 → #FFB255) — reserved for hero word emphasis

## 3. Spacing & Borders
### Spacing Scale (rem)
Section Hero: 10rem (160px) — top of page heroes
Section L: 7.5rem (120px) — between major sections
Section M: 5rem (80px) — standard section gap
Section S: 3rem (48px) — tight section gap
Element L: 4rem (64px) — between large blocks
Element M: 2.5rem (40px) — between component groups
Element S: 2rem (32px) — between components
Element XS: 0.75rem (12px) — tight stacks
Text L: 1.5rem / Text M: 1rem / Text S: 0.5rem / Text XS: 0.25rem
Container padding: 3rem (48px)
Layout column/row gap: 2.5rem (40px)

### Border Radius
Button M: 0.75rem (12px)
Button S: 0.5rem (8px)
Card S: 0.75rem (12px) | Card M: 1rem (16px) | Card L: 1.5rem (24px)
Pill S: 0.5rem | Pill M: 0.75rem
Tab: 0.5rem
Input: 1rem (16px) — distinctively rounded inputs
Icon container: 0.5rem
Checkbox: 0.25rem

### Border Colors
Default border: rgba(13, 14, 22, 0.08) — #0D0E1614 (hairline on light)
Secondary border: rgba(20, 16, 37, 0.08) — #14102514 (button outlines)
Primary button border: rgba(255, 255, 255, 0.12) — #FFFFFF1F (inner highlight on violet)

## 4. Components
### Button Variants
Primary Button (.attention-button-primary): Violet background (#8F78FF), white text (#FFFFFF), inner border rgba(255,255,255,0.12), hover background #523DBC, radius 0.75rem, padding 1rem / 1.25rem.
Secondary Button (.attention-button-secondary): White background (#FFFFFF), dark text (#0D0E16) with secondary text option #666666, border rgba(20,16,37,0.08), hover background rgba(20,16,37,0.08).
Tertiary / Text Button (.attention-button-tertiary): Transparent background, text color #0D0E16, hover keeps text color, often paired with arrow icon (→).
Small Button: Padding 0.625rem / 1.125rem, radius 0.5rem.
Icon Button (.attention-button-icon): Square icon container, padding 0.5rem (small) / 1rem (medium), radius 0.5rem.
Social Icon Button: Padding 0.75rem, circular treatment for social/auth providers.

#### Button States Consistency
- **Hover:** Primary deepens to #523DBC; secondary fills to rgba(20,16,37,0.08); tertiary keeps color but may add underline or arrow translation.
- **Focus:** Inputs and buttons use the violet (#8F78FF) as focus border accent.
- **Active/Processing:** Maintain button color with reduced opacity (~0.6) and subtle inline spinner.
- **Transitions:** Smooth fades and minor translate on arrow icons; avoid dramatic scale.

### Real-time / Marketing Display Elements
Hero Gradient Headline (.attention-hero-title): Hanken Grotesk at Title XXL/XL with linear-gradient text fill (#5C82FF → #CE51E4 → #FFB255).
Glow Surface (.attention-glow): Decorative blurred radial wash using gradient solids at 24% alpha (#688BFF3D, #FF71FD3D, #FFC9783D) behind hero/card sections.
Testimonial Card (.attention-testimonial-card): Surface 2 (#F9F9FB) background, radius 1rem, padding 2rem / 1.5rem, headshot + logo + quote layout.
Logo Wall (.attention-logo-wall): White or Surface 2 background, monochrome customer logos, generous column gap 2.5rem.
Stat Pill (.attention-stat-pill): Pale violet or surface 3 (#F3F3F7) background, radius 0.5rem (S) / 0.75rem (M), padding 0.375rem / 0.75rem (S) or 0.75rem / 1rem (M).

### Form Elements
Input (.attention-input): White surface (#FFFFFF), hover surface #FAFAFA, radius 1rem, padding 0.25rem / 1.25rem, placeholder #BFBFBF, focus border #8F78FF, hover shadow `0 .3rem 1rem -.5rem rgba(0,0,0,0.25)`.
Textarea (.attention-textarea): Mirrors input styling with the same 1rem radius and surface treatment.
Checkbox (.attention-checkbox): Radius 0.25rem, accent color #8F78FF when checked.

### Authentication / Lead Capture
Auth Container (.attention-auth):
- Container: Surface 1 (#FFFFFF) with subtle hairline border #0D0E1614, radius 1.5rem (Card L).
- Input Fields: Match .attention-input styling.
- Primary CTA: Follows .attention-button-primary patterns (violet #8F78FF).
- Social/SSO Buttons: Use .attention-button-secondary with provider icon.
- Error Messages: #EA384C with subtle fade-in.
- Success Messages: #5C82FF or violet check, fade-in animation.

### Cards and Containers
Card (.attention-card): Surface 1 (#FFFFFF) background, hairline border #0D0E1614, radius 1rem, padding 2rem / 1.5rem, very subtle shadow on hover only.
Large Card (.attention-card-l): Radius 1.5rem, padding 4rem / 4rem — for feature/case-study modules.
Small Card (.attention-card-s): Radius 0.75rem, padding 1rem — for compact list items.
Overlay Card (.attention-overlay-card): Surface 1 over rgba(0,0,0,0.25) overlay (#00000040), stronger shadow, radius 1.5rem.
Gradient-Glow Card: Card on a Surface 1/2 base with a low-alpha gradient wash behind (#688BFF3D / #FF71FD3D / #FFC9783D) — used for premium feature highlights.

### Tables
Table (.attention-table): Hairline borders #0D0E1614, alternating rows on Surface 2 (#F9F9FB).
Table Header: Hanken Grotesk semi-bold, Surface 3 (#F3F3F7) background, primary text #141025.
Table Row Hover: Surface 2 (#F9F9FB) with smooth fade.

### Status Indicators
Status Indicator (.attention-status-indicator): Small dot, 0.5rem diameter, with colored fill.
Active Status: Violet (#8F78FF) — brand-aligned default
Positive/Success: Gradient Blue (#5C82FF)
Warning: Gradient Amber (#FFB255)
Error: #EA384C
Inactive: Light Gray (#BFBFBF)

### Progress Tracking
Progress Bar: Violet (#8F78FF) animated fill on Surface 3 (#F3F3F7) track, radius 0.5rem.
Progress Steps: Connected dots, completed in violet (#8F78FF), inactive in #BFBFBF, hairline connector in #0D0E1614.

### Iconography
Icon sizes: XS 1.25rem | S 1.5rem | M 2rem | L 3.5rem
Icon style: Line/stroke icons, minimal weight, paired with arrow (→) for links and CTAs. Color inherits from text color of the parent.

### Imagery & Illustration
- High-quality executive headshots, square crops with rounded corners.
- Product screenshots presented in clean rounded-corner frames over gradient-glow backgrounds.
- Customer logos rendered as monochrome SVG to maintain visual rhythm on logo walls.
- Avoid heavy illustration; favor real product UI and faint gradient atmospherics.

### Voice & Brand Tone Alignment
- Confident, AI-forward, results-oriented ("70% close-rate increase," "tens of hours saved").
- Headlines short and assertive in Hanken Grotesk; supporting copy detailed in Uncut Sans.
- CTAs use direct verbs: "Read Case Study," "Learn more," "Discover," "Book a demo" — always paired with an arrow icon.
