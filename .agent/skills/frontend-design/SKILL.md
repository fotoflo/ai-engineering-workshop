---
name: frontend-design
description: Create clean, professional frontend interfaces for Flexbike's motorcycle rental marketplace. Use this skill when building React components, pages, or UI elements that match Flexbike's design system with teal branding, Nunito typography, and modern UX patterns.
license: Complete terms in LICENSE.txt
---

This skill guides creation of clean, professional frontend interfaces that perfectly match Flexbike's motorcycle rental marketplace design system. Implement production-ready React components with exceptional attention to usability, accessibility, and brand consistency.

## Flexbike Design System Overview

Flexbike uses a clean, modern aesthetic focused on motorcycle rental discovery and booking. The design emphasizes:

- **Trust & Professionalism**: Clean layouts, verified badges, instant booking indicators
- **Adventure & Exploration**: Motorcycle imagery, location-based search, travel-focused UX
- **Simplicity & Usability**: Intuitive navigation, clear information hierarchy, responsive design

## Design Thinking Process

Before coding, understand the motorcycle rental context:

- **User Journey**: Bike discovery → Location search → Booking flow → Payment → Confirmation
- **Trust Signals**: Verified companies, ratings, instant booking, secure payments
- **Mobile-First**: Many users browse on phones while traveling
- **Performance**: Fast loading, smooth interactions, offline-capable features

**CRITICAL**: Every interface element should support the goal of helping users find and book motorcycles confidently.

## Flexbike Frontend Guidelines

### Colors & Branding

- **Primary**: `#08b7b7` (teal) - Trust, adventure, booking actions
- **Secondary**: `#242420` (charcoal) - Text, strong elements
- **Light**: `#F6F5F1` (eggshell) - Backgrounds, subtle elements
- **Accent**: `#EBE8E1` (dark eggshell) - Borders, secondary backgrounds
- **Status Colors**: Emerald (instant booking), Blue (verified), Yellow (highlights)

### Typography

- **Primary Font**: Nunito (weights: Regular, SemiBold, Bold, ExtraBold, Black)
- **Usage**: All text elements, headings, buttons, navigation
- **Hierarchy**: Clear size relationships, proper contrast ratios
- **Accessibility**: Minimum 4.5:1 contrast ratio for text

### Layout & Spacing

- **Grid**: Responsive 12-column system with proper gutters
- **Spacing Scale**: Consistent 4px increments (4, 8, 12, 16, 20, 24, 32, 40, 48, 64px)
- **Containers**: Max width 1400px, centered with proper padding
- **Cards**: Rounded corners (8-12px), subtle shadows, clean borders

### Components & Patterns

#### Navigation

- Clean header with logo, navigation links, CTA buttons
- Backdrop blur effects for modern glass-morphism feel
- Mobile-responsive with proper hamburger menu

#### Cards & Listings

- Bike cards with high-quality images, pricing, location info
- Hover effects with subtle scaling and shadow changes
- Badges for features (Electric ⚡, Instant Book, Verified)
- Consistent aspect ratios and loading states

#### Forms & Inputs

- Rounded input fields with proper focus states
- Clear labels and helpful placeholder text
- Validation feedback with appropriate colors
- Accessible form controls with proper ARIA labels

#### Buttons & CTAs

- Primary: Teal gradient (`from-emerald-600 to-teal-600`)
- Secondary: Blue gradient (`from-blue-600 to-indigo-600`)
- Consistent sizing (h-10, h-12) with rounded-xl corners
- Hover animations with subtle transforms

### Motion & Interactions

#### Loading States

- Skeleton loaders matching content structure
- Smooth fade-in animations with staggered delays
- Progressive image loading with blur-to-sharp transitions

#### Hover Effects

- Subtle scaling (1.02x) and shadow elevation
- Color transitions for interactive elements
- Smooth transforms (200-300ms duration)

#### Page Transitions

- Smooth route changes with proper loading states
- Maintain scroll position where appropriate
- Error boundaries with graceful fallbacks

### Responsive Design

#### Mobile-First Approach

- Touch-friendly button sizes (minimum 44px height)
- Readable text sizes (minimum 16px)
- Proper spacing for thumb navigation

#### Tablet & Desktop

- Enhanced layouts with more columns
- Hover states and advanced interactions
- Optimized for mouse and keyboard navigation

### Accessibility Standards

#### WCAG 2.1 AA Compliance

- Proper heading hierarchy (h1-h6)
- Alt text for all images
- Sufficient color contrast ratios
- Keyboard navigation support
- Screen reader compatibility

#### Focus Management

- Visible focus indicators
- Logical tab order
- Focus trapping in modals
- Skip links for keyboard users

### Performance Considerations

#### Image Optimization

- Next.js Image component with proper sizing
- WebP format with fallbacks
- Lazy loading for below-the-fold content
- Proper aspect ratios to prevent layout shift

#### Bundle Optimization

- Code splitting for large components
- Tree shaking for unused dependencies
- Proper loading strategies (eager/lazy)

### Error Handling

#### Graceful Degradation

- Fallback images and content
- Offline-capable features where possible
- Clear error messages with actionable next steps

#### User Feedback

- Loading spinners and skeleton states
- Success/error toast notifications
- Clear call-to-action buttons

**REMEMBER**: Flexbike interfaces should feel trustworthy, adventurous, and easy to use. Every design decision should support users in their motorcycle rental journey. Focus on clarity, performance, and delightful micro-interactions that enhance the booking experience.
