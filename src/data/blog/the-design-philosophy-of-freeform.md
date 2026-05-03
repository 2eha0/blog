---
author: Zehao
pubDatetime: 2026-05-03T10:00:00Z
modDatetime: 2026-05-03T10:00:00Z
title: "The Design Philosophy of Freeform: Building a Schema-Driven UI with Great DX"
slug: the-design-philosophy-of-freeform
featured: true
draft: false
tags:
  - architecture
  - vue
  - dx
  - kong
description: A deep dive into the core architecture of the free-form component, exploring how to balance automation with extreme customizability in large-scale form engines.
---

To be honest, building forms is often the most exhausting yet unavoidable "grunt work" in frontend development.

Especially when you're dealing with a massive system like Kong, which has hundreds of plugins. Each plugin has its own set of configurations, dependencies, and nested structures. If you’re still hard-coding these forms, you’re in for a nightmare of maintenance and technical debt.

When I was designing [`free-form`](https://github.com/Kong/public-ui-components/blob/main/packages/entities/entities-plugins/src/components/free-form/README.md), my goal was to create a schema-driven dynamic form engine that manages to move from "manual driving" to "autopilot" without losing the ability to take the wheel when needed.

Today, I want to share the design philosophy behind this component and how I built it with **Developer Experience (DX)** at its core.

## Table of contents

## The "Three Musketeers" of Freeform

To understand how it works, let's dissect a real-world example: the `proxy-cache-advanced` plugin. This plugin is a perfect case study because it utilizes the three core pillars of the engine.

### 1. The Registry: Auto-Discovery Magic

In a system with hundreds of plugins, the first problem to solve is "how do we add a new one without touching the core logic?"

I built a `plugin-registry.ts` that handles auto-discovery. As a plugin developer, you just drop a `.ts` file into the `plugins/` directory and use `definePluginConfig`.

```typescript
export default definePluginConfig({
  // plugin configuration
})
```

The system automatically scans and registers it. This "zero-config" entry means you don't need to modify global configuration files or core library code. It’s clean, decoupled, and significantly reduces the cognitive load for contributors.

### 2. RenderRules: Declarative Business Logic

This is where the real power lies. In `proxy-cache-advanced`, we have logic like: "Only show the Redis configuration if the cache strategy is set to `redis`."

Instead of messy `v-if` chains in your templates, I opted for declarative `RenderRules`.

```typescript
renderRules: {
  dependencies: {
    'config.redis': ['config.strategy', 'redis'],
  },
}
```

This snippet tells the engine: "Only render `config.redis` when `config.strategy` equals `redis`." The logic is compressed into pure metadata. No manual state management, no complex watchers. It just works.

### 3. FieldRenderer: The Precision Scalpel

I know what some of you are thinking: "Schema is great for the 80% case, but what about that 20% of weird UI requirements that a schema can't describe?"

That’s where the `FieldRenderer` comes in. It’s the "escape hatch" I designed to ensure flexibility.

In our case study, we needed some specific fields to use a custom `EnumField` and enable multiple selection.

```typescript
fieldRenderers: [
  {
    match: ({ path }) => {
      return path === 'config.methods' || path === 'config.request_method'
    },
    component: EnumField,
    propsOverrides: { multiple: true },
  },
],
```

The `match` function gives you surgical precision. You can intercept any field by its path and replace it with a custom component or override its props. It provides the flexibility of "manual driving" within an automated system.

## Designing for Great DX

Why go through all this trouble for abstraction? Because the essence of DX isn't just about writing fewer lines of code—it's about **not getting stuck**.

When I designed `free-form`, I followed three key principles:

1.  **Data-First UI**: The schema should be the single source of truth whenever possible.
2.  **Escape Hatches are Mandatory**: Never try to box everyone into a single framework. Always provide slots and matchers for the edge cases.
3.  **Context Matters**: Leverage tools like Vue 3's `provide/inject` to share global form state (like `FormContext`) with deeply nested fields.

## Conclusion

Looking back at how `free-form` scales within Kong, it’s clear that moving towards decoupled, schema-driven patterns is the endgame for large-scale frontend projects. It allows us to support hundreds of plugins with minimal effort while maintaining a consistent user experience.

I hope these three pillars—Registry, RenderRules, and FieldRenderer—give you some inspiration for your own projects. DX is a journey, and sometimes, the best way to move forward is to build a system that knows when to get out of your way.

Stay curious, stay bold, and keep building.
