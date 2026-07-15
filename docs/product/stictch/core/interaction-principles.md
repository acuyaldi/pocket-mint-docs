# Pocket Mint Interaction Principles

Version: 1.0

Status: Foundation

Owner: Pocket Mint

Category: Interaction Principles

---

# Purpose

This document defines how users interact with Pocket Mint.

Interaction should always feel:

- predictable
- responsive
- calm
- consistent

Interactions should never surprise users.

The interface should respond immediately to user actions while remaining visually restrained.

---

# Interaction Philosophy

Interaction exists to support financial tasks.

Not to entertain users.

Every interaction should communicate:

- what happened
- why it happened
- what users can do next

Interaction should reduce uncertainty.

Never increase it.

---

# Core Principles

Every interaction should be:

Predictable

↓

Immediate

↓

Understandable

↓

Reversible whenever possible

Users should feel in control at all times.

---

# Primary Interaction

Primary actions should require minimal effort.

Examples

Add Transaction

Transfer

Pay Installment

Create Wallet

Primary actions should always be immediately recognizable.

---

# Secondary Interaction

Secondary actions support workflows.

Examples

Filter

Sort

View All

Export

Rename

Secondary actions should remain visually quieter.

---

# Contextual Interaction

Contextual actions appear only when relevant.

Examples

Delete Wallet

Edit Transaction

Archive Wallet

Mark Installment Paid

Avoid exposing contextual actions globally.

---

# Hover States

Desktop only.

Hover should communicate:

This element is interactive.

Hover should not dramatically change layout.

Avoid:

Scaling

Large shadows

Strong animations

Hover should remain subtle.

---

# Focus States

Every interactive element should have a visible focus state.

Focus should improve accessibility.

Never remove focus indicators.

---

# Pressed States

Pressed states should provide immediate confirmation.

Avoid dramatic visual changes.

Interaction should feel responsive.

---

# Selected States

Selection should remain visually obvious.

Examples

Navigation

Tabs

Filters

Wallet Selection

Selected state should persist until another selection is made.

---

# Disabled States

Disabled components should remain understandable.

Users should know:

Why the action is unavailable.

Avoid making disabled elements appear broken.

---

# Loading Behaviour

Loading should preserve layout.

Prefer:

Skeleton

Inline Loading

Progress Indicators

Avoid blocking the entire interface.

Critical information should appear first.

---

# Success Feedback

Successful actions should receive subtle confirmation.

Examples

Transaction Added

Wallet Created

Installment Paid

Feedback should disappear automatically when appropriate.

---

# Error Feedback

Errors should explain:

What happened.

Why.

How to recover.

Avoid technical language.

Avoid generic:

"Something went wrong."

---

# Confirmation

Only request confirmation for destructive actions.

Examples

Delete Wallet

Delete Transaction

Delete Installment

Avoid unnecessary confirmations.

---

# Undo

Whenever possible,

prefer:

Undo

instead of

Confirmation Dialog

Undo reduces interruption.

---

# Forms

Forms should guide users naturally.

Validation should occur as early as helpful.

Avoid displaying every validation error at once.

Support progressive completion.

---

# Input Behaviour

Inputs should always communicate state.

Normal

Focused

Filled

Disabled

Error

Read Only

State transitions should remain subtle.

---

# Financial Input

Currency inputs should prioritize clarity.

Always format values consistently.

Avoid changing cursor position unexpectedly.

---

# Lists

Lists should support fast scanning.

Selection should remain obvious.

Scrolling should remain smooth.

Avoid excessive interaction inside a single list row.

---

# Navigation Interaction

Navigation should never interrupt the user's mental model.

Switching screens should feel immediate.

The current location should always remain obvious.

---

# Motion Philosophy

Motion supports understanding.

Never decoration.

Motion should communicate:

Appearance

Disappearance

Expansion

Collapse

Loading

Completion

Nothing more.

---

# Animation

Animations should be:

Fast

Subtle

Consistent

Avoid:

Bounce

Elastic

Overshoot

Playful transitions

Financial software should feel stable.

---

# Transition Consistency

Every transition should follow the same visual language.

Avoid introducing unique animations for individual screens.

Consistency builds confidence.

---

# Feedback Hierarchy

Immediate

↓

Inline

↓

Toast

↓

Dialog

Choose the least disruptive feedback mechanism possible.

---

# Interruptions

Avoid interrupting users unnecessarily.

Examples

Don't show popups for successful saves.

Don't display unnecessary confirmations.

Don't require excessive clicks.

Interaction should remain efficient.

---

# Accessibility

Every interaction should support:

Keyboard

Touch

Mouse

Screen Reader

High Contrast

Reduced Motion

Accessibility is part of interaction.

Not an optional enhancement.

---

# Responsive Behaviour

Interaction patterns remain consistent.

Desktop

Hover available.

Tablet

Touch-first.

Mobile

Touch-first.

Only interaction method changes.

Behaviour remains the same.

---

# Anti Patterns

Do not:

- surprise users
- hide important actions
- require unnecessary confirmations
- overload dialogs
- animate excessively
- delay interaction feedback
- block the interface unnecessarily
- introduce inconsistent behaviour

Interaction should always support confidence.

---

# Acceptance Criteria

Interaction is successful when:

✓ Users immediately understand what is interactive.

✓ Feedback is immediate.

✓ Errors are understandable.

✓ Success is communicated clearly.

✓ Navigation feels predictable.

✓ Forms feel guided.

✓ Motion supports understanding.

✓ Users remain in control.

---

# Definition of Done

The interaction system is complete when every Pocket Mint screen behaves consistently regardless of feature or platform.

This document serves as the authoritative interaction reference for all future UI generation and frontend implementation.