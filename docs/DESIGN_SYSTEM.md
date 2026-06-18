# Design System: Omie's Swift Logistics 🎨

## Executive Summary

A vibrant, modern design language optimized for the Nigerian market with energetic colors, minimal friction, and accessibility-first components.

---

## 1. Color Palette & Psychology

### Primary Colors

**Primary Blue: #1F5FFF** (Trust, Action, Professional)
RGB(31, 95, 255) | HSL(218°, 100%, 56%)
Usage: Primary CTAs, active navigation, links, loading states

**Secondary Yellow: #FFD700** (Energy, Attention, Urgency)
RGB(255, 215, 0) | HSL(51°, 100%, 50%)
Usage: Rider badges, accent highlights, peak hour indicators, surge pricing

**Success Green: #00D084** (Completion, Confidence)
RGB(0, 208, 132) | HSL(155°, 100%, 41%)
Usage: Order confirmation, completed statuses, payment success, rider accepted

**Warning Orange: #FF6B35** (Alert, Caution)
RGB(255, 107, 53) | HSL(12°, 100%, 60%)
Usage: Peak hour alerts, surge pricing, delayed orders, driver requests

**Danger Red: #E63946** (Error, Stop)
RGB(230, 57, 70) | HSL(357°, 75%, 56%)
Usage: Error messages, cancellations, failed payments, system failures

### Neutral Colors

- **Deep Navy: #0F1419** (Primary Text)
- **Off-white: #F8F9FA** (Card backgrounds)
- **Medium Gray: #6C757D** (Secondary text)
- **Light Gray: #E9ECEF** (Borders, dividers)

---

## 2. Typography System

### Font Family
- **Primary**: Inter (Google Fonts)
- **Monospace**: JetBrains Mono (prices, codes, numbers)

### Type Scale
- **Display**: 32px Bold (App title)
- **Heading 1**: 28px SemiBold (Page titles)
- **Heading 2**: 24px SemiBold (Card titles)
- **Body Large**: 18px Regular (Primary text)
- **Body Medium**: 16px Regular (Standard text)
- **Body Small**: 14px Regular (Secondary text)
- **Caption**: 12px Regular (Fine print)
- **Button**: 16px SemiBold (Button labels)
- **Price**: 20px Monospace Bold (Monetary values)

---

## 3. Component Library

### Buttons
- **Primary**: Blue bg, white text, 48px min height
- **Secondary**: Outline style, blue border
- **Ghost**: Transparent background
- **Icon Button**: Circular, 48px × 48px

### Cards
- **Rider Profile**: Photo, rating, distance, action buttons
- **Order Summary**: Location, distance, items, status
- **Payment Method**: Card info, default indicator

### Input Fields
- **Text Input**: Floating label, focus indicators
- **Search**: Icon + clear button, debounced
- **Number Input**: +/- controls for quantity/weight

### Modals & Dialogs
- **Confirmation Modal**: Backdrop overlay, smooth animation
- **Payment Modal**: Payment method selection

### Navigation
- **Top Header**: Sticky, logo + user menu
- **Bottom Tab Bar**: Mobile-first, 5 icons + labels

---

## 4. Layout & Spacing

### Spacing Scale (8px base)
- **xs**: 4px (small gaps)
- **sm**: 8px (default spacing)
- **md**: 16px (card padding)
- **lg**: 24px (major sections)
- **xl**: 32px (page margins)

### Responsive Breakpoints
- **Mobile**: 320px - 767px
- **Tablet**: 768px - 1023px
- **Desktop**: 1024px+

---

## 5. Shadow & Depth

- **Elevation 1**: `0 2px 8px rgba(0, 0, 0, 0.08)` (Cards)
- **Elevation 2**: `0 4px 16px rgba(0, 0, 0, 0.12)` (Dropdowns)
- **Elevation 3**: `0 8px 32px rgba(0, 0, 0, 0.16)` (Modals)
- **Elevation 4**: `0 12px 48px rgba(0, 0, 0, 0.20)` (FABs)

---

## 6. Animation & Transitions

- **Fast (100-150ms)**: Micro-interactions
- **Standard (200-300ms)**: Modals, transitions
- **Slow (400-500ms)**: Loaders, major shifts

### Motion Principles
- **Entrance**: Fade in + slide up (200ms)
- **Exit**: Fade out + slide down (150ms)
- **Hover**: Scale 1.02 + shadow increase (100ms)
- **Loading**: Smooth spinner (1s infinite loop)

---

## 7. Accessibility Standards

✅ **WCAG 2.1 AA Compliance**
- Minimum contrast ratio 4.5:1 for text
- Minimum 48px touch targets (mobile)
- Keyboard navigation support
- Screen reader labels on all interactive elements
- Focus visible indicators (2px blue border)
- No auto-playing audio/video

---

## 8. Brand Voice & Tone

### Writing Style
- **Friendly**: "Hey! Let's get your package delivered 🚀"
- **Nigerian-aware**: Uses local context ("Peak hours", "Go slow")
- **Action-oriented**: "Book Now", "Confirm Order"
- **Empathetic**: "Oops! Something went wrong. We've got you."

### Message Examples

**Success:**
```
✓ Order confirmed!
Your rider Chukwu will arrive in 4 minutes.
Sit back, we've got this! 🎉
```

**Error:**
```
❌ Payment failed
Your card was declined. Try another card or USSD.
Need help? Chat with us anytime.
```

---

**Last Updated**: June 2026 | **Version**: 1.0