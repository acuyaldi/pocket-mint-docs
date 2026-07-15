# Screen Specification Template

Version: 1.0

Status: Template

Owner: Pocket Mint

Category: Screen Specification

---

# Purpose

Describe the primary purpose of this screen.

This section answers:

Why does this screen exist?

The purpose should be concise and focused.

Avoid describing implementation details.

---

# Product Role

Explain how this screen fits into Pocket Mint.

Examples:

- Primary Workspace
- Detail View
- Management Screen
- Supporting Screen

This establishes the importance of the screen within the application.

---

# User Goals

Describe what users want to achieve.

Examples:

- Understand
- Record
- Review
- Manage
- Update
- Navigate

User goals should be task-oriented.

---

# Primary User Questions

List the questions this screen should answer.

Example:

What am I looking at?

What can I do here?

What information matters?

What should I do next?

Every screen should answer a small number of clear questions.

---

# Information Priority

Describe information importance.

Example:

Highest

↓

High

↓

Medium

↓

Low

↓

Supporting

Priority should remain stable across responsive layouts.

---

# Primary Sections

List every major section.

For each section define:

Purpose

Displayed Information

Primary Actions

Navigation Destination

Priority

This becomes the contract between Product and UI.

---

# Available Actions

List every action available on the screen.

Separate actions into:

Primary

Secondary

Contextual

Overflow

Actions should support user goals.

---

# Data Requirements

Define required information.

Required

Optional

Calculated

Derived

Unavailable

Do not describe implementation.

Describe business expectations.

---

# Business Rules

Define business constraints.

Examples:

Always

Never

Only

Maximum

Minimum

Business rules should remain independent from UI.

---

# Interaction Rules

Describe interaction expectations.

Examples:

Tap

Click

Long Press

Swipe

Expand

Collapse

Navigate

Interactions should be predictable.

---

# States

Document all supported states.

Normal

Loading

Empty

Partial

Error

Offline

Permission Restricted

Every screen should define these states.

---

# Navigation Behaviour

Define navigation relationships.

Where users arrive from.

Where users continue.

What happens after actions.

---

# Responsive Behaviour

Describe layout expectations.

Desktop

Tablet

Mobile

Do not define spacing here.

Layout belongs to composition.

---

# Accessibility

Define accessibility expectations.

Readable

Keyboard Accessible

Screen Reader Friendly

Touch Friendly

Semantic

---

# Privacy

Describe privacy expectations.

Sensitive Information

Hidden Information

Masked Information

Optional Visibility

Pocket Mint should always prioritize privacy.

---

# Performance Considerations

Define loading priorities.

Critical

High

Medium

Low

The most important information should appear first.

---

# Success Criteria

Describe measurable success.

Users can complete their primary goals.

Information is understandable.

Navigation is clear.

No unnecessary cognitive load exists.

---

# Things to Avoid

List prohibited patterns.

Examples:

Do not overload the screen.

Do not duplicate functionality.

Do not introduce unrelated information.

Do not display fake data.

Do not violate product hierarchy.

---

# Definition of Done

The screen specification is complete when:

✓ Purpose is defined.

✓ User goals are documented.

✓ Information hierarchy is fixed.

✓ Business rules are complete.

✓ States are documented.

✓ Accessibility is considered.

✓ Navigation relationships are defined.

✓ Success criteria are measurable.

This document becomes the functional contract for the screen.