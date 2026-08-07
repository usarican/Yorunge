---
name: Yörünge Observatory Design System
colors:
  surface: '#FFFFFF'
  surface-dim: '#E3E8F3'
  surface-bright: '#FFFFFF'
  surface-container-lowest: '#FFFFFF'
  surface-container-low: '#F6F8FC'
  surface-container: '#EEF1F7'
  surface-container-high: '#E3E8F3'
  surface-container-highest: '#D5DAEA'
  on-surface: '#121630'
  on-surface-variant: '#565F82'
  inverse-surface: '#161B33'
  inverse-on-surface: '#F0EDE4'
  outline: '#D5DAEA'
  outline-variant: '#D9DEEC'
  surface-tint: '#2F4FAE'
  primary: '#121630'
  on-primary: '#FFFFFF'
  primary-container: '#EEF1F7'
  on-primary-container: '#121630'
  inverse-primary: '#6C9BFF'
  secondary: '#2F4FAE'
  on-secondary: '#FFFFFF'
  secondary-container: '#D9DEEC'
  on-secondary-container: '#121630'
  tertiary: '#A85F14'
  on-tertiary: '#FFFFFF'
  tertiary-container: '#FFEAD2'
  on-tertiary-container: '#A85F14'
  error: '#A13A54'
  on-error: '#FFFFFF'
  error-container: '#FFDCE2'
  on-error-container: '#A13A54'
  background: '#EEF1F7'
  on-background: '#121630'
  surface-variant: '#F6F8FC'
typography:
  headline-xl:
    fontFamily: Newsreader
    fontSize: 38px
    fontWeight: '500'
    lineHeight: 46px
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Newsreader
    fontSize: 28px
    fontWeight: '500'
    lineHeight: 34px
    letterSpacing: -0.02em
  headline-md:
    fontFamily: Newsreader
    fontSize: 22px
    fontWeight: '500'
    lineHeight: 28px
    letterSpacing: -0.01em
  body-lg:
    fontFamily: Hanken Grotesk
    fontSize: 16.5px
    fontWeight: '400'
    lineHeight: 24px
    letterSpacing: '0'
  body-md:
    fontFamily: Hanken Grotesk
    fontSize: 15px
    fontWeight: '400'
    lineHeight: 22px
    letterSpacing: '0'
  label-lg:
    fontFamily: Space Mono
    fontSize: 13.5px
    fontWeight: '400'
    lineHeight: 18px
    letterSpacing: 0.02em
  label-sm:
    fontFamily: Space Mono
    fontSize: 11.5px
    fontWeight: '400'
    lineHeight: 14px
    letterSpacing: 0.05em
rounded:
  sm: 4px
  DEFAULT: 10px
  md: 10px
  lg: 16px
  xl: 24px
  full: 9999px
spacing:
  base: 8px
  gutter: 16px
  margin-lateral: 24px
---

# Yörünge Observatory Design System (Light & Senior Dev Edition)

## Brand & Style Philosophy

Yörünge is a continuous developer assessment and growth platform built on the metaphor of **Orbital Mechanics** (concentric trajectory rings, individual speed, universal physics/rubric). It is NOT a course platform or a bootcamp. It evaluates real GitHub code and CV evidence to chart a personalized growth trajectory.

The visual style is **Daylight Observatory / Mission Control Panel**. It avoids cold tech clichés and AI-slop (such as generic Space Grotesk + Inter pairings or plain dark card grids).

### Key Pillars:
1. **Direct & Senior:** Crisp typography, zero fluff, high signal-to-noise ratio.
2. **Cold Daylight Canvas (`#EEF1F7`):** Inspired by daylight sky over an observatory rather than warm editorial parchment or sterile plain white.
3. **Concentric Radial Orbit Canvas:** Visualizing developer skills as concentric interactive orbital nodes around the developer's core.
4. **Code-Review / PR-Style Dual Panel:** Dual-pane interface where the AI Mentor provides direct senior feedback alongside GitHub AST code evidence.

## Color System

- **Background:** `#EEF1F7` (Daylight Sky Canvas)
- **Primary Text:** `#121630` (Ink Navy)
- **Primary Surface:** `#FFFFFF` (Observatory Panel)
- **Secondary Surface:** `#F6F8FC` (Elevated Sub-panel)
- **Orbit Blue (Active Trajectory):** `#2F4FAE`
- **Amber (Completed Orbit / Verified Milestone):** `#A85F14`
- **Pink/Flame (Gap / Needs Attention):** `#A13A54`
- **Border:** `#D5DAEA`

## Typography

- **Display & Headings:** `Newsreader` (Italic accent for milestone personal moments)
- **Body & Interface:** `Hanken Grotesk` (Readable, clean, non-AI-slop sans)
- **Telemetry & Data:** `Space Mono` (Mission control terminal character)

## Components

### Radial Orbit Canvas
Concentric 0.5pt dashed trajectory circles around a central core ("Sen" amber node). Sub-nodes represent Android Core, Architecture, Coroutines, Jetpack Compose, Memory & Performance, and Automated Testing.

### Dual-Pane Code-Review AI Mentor
- **Left Pane:** Senior mentor analysis, unsoftened yet constructive feedback, RAG sources.
- **Right Pane:** GitHub code snippet evidence (`AppModule.kt:L42-L65`), AST tree tags, telemetry metrics.
