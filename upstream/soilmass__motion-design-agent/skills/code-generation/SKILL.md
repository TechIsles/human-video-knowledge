---
name: code-generation
description: Generate production-ready animation code from specifications. Use when transforming animation specs into CSS, GSAP, Framer Motion, or Remotion implementations with token references, reduced motion support, and performance optimizations.
---

# Code Generation

Generate production-ready animation code from orchestrated parameters. This skill transforms animation specifications into platform-specific implementations with proper token references, reduced motion support, and performance optimizations.

## Supported Platforms

| Platform | Output Format | Template |
|----------|---------------|----------|
| CSS | Keyframes + Transitions | `css-keyframes.template.txt`, `css-transition.template.txt` |
| GSAP | Timeline functions | `gsap-timeline.template.txt` |
| Framer Motion | Variants + hooks | `framer-motion-variant.template.txt` |
| Remotion | React components | `remotion-component.template.txt` |

## Generation Pipeline

```
Animation Specification
         |
         v
+------------------+
|  Token Resolver  | -> Map tokens to actual values
+--------+---------+
         |
         v
+------------------+
| Platform Adapter | -> Convert to platform format
+--------+---------+
         |
         v
+------------------+
| Template Filler  | -> Interpolate values
+--------+---------+
         |
         v
+------------------+
| Reduced Motion   | -> Generate a11y variant
+--------+---------+
         |
         v
    Output Code
```

## Token Resolution

### Duration Tokens

| Token | ms | seconds | frames@30fps |
|-------|---:|--------:|-------------:|
| `short-1` | 50 | 0.05 | 2 |
| `short-2` | 100 | 0.1 | 3 |
| `short-3` | 150 | 0.15 | 5 |
| `short-4` | 200 | 0.2 | 6 |
| `medium-1` | 250 | 0.25 | 8 |
| `medium-2` | 300 | 0.3 | 9 |
| `medium-3` | 350 | 0.35 | 11 |
| `medium-4` | 400 | 0.4 | 12 |
| `long-1` | 450 | 0.45 | 14 |
| `long-2` | 500 | 0.5 | 15 |

### Easing Tokens

| Token | CSS | GSAP |
|-------|-----|------|
| `standard` | `cubic-bezier(0.2, 0, 0, 1)` | `power2.out` |
| `emphasized-decelerate` | `cubic-bezier(0.05, 0.7, 0.1, 1)` | `power3.out` |
| `emphasized-accelerate` | `cubic-bezier(0.3, 0, 0.8, 0.15)` | `power2.in` |
| `standard-decelerate` | `cubic-bezier(0, 0, 0, 1)` | `power2.out` |
| `linear` | `linear` | `none` |

### Spring Tokens

| Token | Damping | Stiffness | Mass |
|-------|--------:|----------:|-----:|
| `standard` | 15 | 150 | 1 |
| `snappy` | 25 | 300 | 0.5 |
| `bouncy` | 10 | 180 | 1 |
| `gentle` | 25 | 100 | 1.2 |
| `celebratory` | 8 | 180 | 1 |

## Platform Support Matrix

### Spring Physics

| Platform | Support | Implementation |
|----------|---------|----------------|
| CSS | Limited | `cubic-bezier(0.34, 1.56, 0.64, 1)` approximation |
| GSAP | Full | `elastic.out(1, 0.3)` |
| Framer Motion | Full | `{ type: 'spring', damping, stiffness, mass }` |
| Remotion | Full | `spring({ frame, fps, config })` |

### 3D Transform

| Feature | CSS | GSAP | Framer Motion | Remotion |
|---------|:---:|:----:|:-------------:|:--------:|
| `perspective` | Yes | Yes | Yes | Yes |
| `rotateX/Y/Z` | Yes | Yes | Yes | Yes |
| `translateZ` | Yes | Yes | Yes | Yes |
| `preserve-3d` | Yes | Yes | via style | Yes |

### Reduced Motion Detection

| Platform | Detection Method |
|----------|------------------|
| CSS | `@media (prefers-reduced-motion: reduce)` |
| GSAP | `window.matchMedia('(prefers-reduced-motion)')` |
| Framer Motion | `useReducedMotion()` hook |
| Remotion | N/A (video) - option: reduced complexity export |

## Generation Workflow

### Input Specification

```typescript
interface AnimationSpec {
  name: string;
  recipe?: string;
  duration: DurationToken;
  easing: EasingToken | SpringToken;
  from: TransformState;
  to: TransformState;
  principles: string[];
  stagger?: number;
  platform: Platform;
}
```

### Output Example

**Input:** "Modal entrance animation with spring physics"

**Orchestrator Output:**
```typescript
{
  name: 'modal-entrance',
  recipe: 'modal-entrance',
  duration: 'medium-1',
  easing: 'spring-enter',
  from: { opacity: 0, scale: 0.95, y: 16 },
  to: { opacity: 1, scale: 1, y: 0 },
  principles: ['anticipation', 'staging'],
}
```

**Generated Outputs:**
- CSS: Keyframe + reduced motion
- GSAP: Timeline function + reduced motion check
- Framer Motion: Variants + hook + reduced variants
- Remotion: Component + spring + stagger variant

## Reduced Motion Strategy

1. **Replace transform animations** with opacity fade
2. **Shorten duration** to 100ms maximum
3. **Remove decorative** secondary animations
4. **Keep essential** state indicators (focus rings, color changes)

### Fallback Mapping

| Animation Type | Reduced Motion Fallback |
|----------------|------------------------|
| Entrance (fade + transform) | `opacity: 1; transform: none;` |
| Exit (fade out) | `opacity: 0;` |
| State change (hover, press) | `opacity: 0.9;` or instant state |
| Loading (shimmer, pulse) | `animation: none;` (static) |
| Error (shake) | `border-color: var(--error);` (no motion) |
| Celebration (particles) | `opacity: 1;` (skip particles) |

## Output Checklist

- [ ] Token references documented
- [ ] Platform-specific format correct
- [ ] Reduced motion variant included
- [ ] Performance optimizations (will-change, etc.)
- [ ] Type definitions (TypeScript)
- [ ] Usage examples in comments

## Related Skills

- `animation-recipes` - Input recipes for code generation
- `easing` - Easing token definitions
- `duration` - Duration token definitions
- `accessibility` - Reduced motion requirements
