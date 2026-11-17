# Frontend patterns

## Overview
Large frontends grow in three ways: more teams, more state, and more heavy work in the browser. Micro-frontends help with teams, the right state tool helps with state, and shared memory between workers helps with heavy work.

## Key concepts
*   **Micro-frontend (MFE)**: A part of the page built and deployed by one team on its own. A shell app puts the parts together.
*   **Composition**: At build time (npm packages), on the server (includes), or at runtime (Module Federation, import maps, Web Components, iframes).
*   **Shared concerns**: The shell owns the URL. Shared libraries load once. Parts talk through the URL, custom events, or a small shared store.
*   **Server-cache libraries (TanStack Query, SWR)**: Fetch and cache API data. Much of what used to live in Redux is really cached server data.
*   **Redux (Toolkit)**: One store, actions, pure reducers. Predictable, good devtools, more code.
*   **Zustand / Jotai**: Small hook-based stores. Components subscribe to the slice they need.
*   **State machines (XState)**: Explicit states and transitions for complex flows like wizards or editors.
*   **Signals (Solid, Preact Signals, Angular, Svelte 5)**: A value that knows who reads it. A change updates only those readers, not the whole component. Vue tracks reads the same way but still re-renders whole components with a virtual DOM.
*   **`SharedArrayBuffer` and `Atomics`**: Memory that the main thread and workers can all read and write. `Atomics` gives safe operations and wait/notify.
*   **Core Web Vitals**: Google's three page-experience metrics. LCP (Largest Contentful Paint): how fast the main content shows. INP (Interaction to Next Paint, which replaced FID in 2024): how fast the page reacts to clicks and typing. CLS (Cumulative Layout Shift): how much the layout jumps while loading.
*   **Bundle size**: Less JavaScript means faster loading and reacting. Split code by route, lazy-load what isn't needed at first, let the bundler drop unused exports (tree shaking), and measure with a bundle analyzer.

## Real-world usage
*   **Large shop**: Cart, search, and account are separate MFEs, each released by its own team.
*   **Editors**: Redux or state machines keep many connected UI states predictable.
*   **Live dashboards**: Signals update one price cell, not the whole table.
*   **Audio in the browser**: A worker writes samples into a `SharedArrayBuffer`, and an `AudioWorklet` reads them, so main-thread lag does not cause glitches.

## Tradeoffs
*   **MFEs vs one app**: Teams ship on their own, but you get version mismatches (two Reacts on one page), bigger bundles unless libraries are shared, harder end-to-end tests, and less consistent UI.
*   **Redux vs React Context**: Redux re-renders only components whose selected value changed. Context re-renders every consumer when its value changes, so it is a poor fit for state that changes often.
*   **Shared memory vs messages**: Shared memory avoids copying, but brings back race conditions. Prefer `postMessage` with transferable objects unless copying is a measured problem.

## When not to use
*   **MFEs**: When one or a few teams own the frontend. A well-organized single app is faster to build.
*   **Global store**: For server data (use a server-cache library) or state used by one part of the page (keep it local).
*   **Signals**: For a mostly static app or simple top-down data flow, plain framework state is easier to debug.

## Interview focus
*   **Routing across MFEs**: The shell router owns top-level paths and mounts each MFE. Each MFE handles paths under its own base path.
*   **When Redux over `useState` + Context**: Many components read and write the same often-changing state, update logic is complex, or you need devtools or middleware.
*   **Signals vs virtual DOM**: Signals track which value each piece of UI reads. Virtual DOM re-runs the component and compares the result.

## Common mistakes
*   **CSS leaking between MFEs**: One team's `button { color: red }` changes another team's buttons. Use CSS Modules, prefixes, or Shadow DOM.
*   **No cleanup on unmount**: Stores, listeners, or subscriptions from a removed MFE keep running.
*   **Missing headers for `SharedArrayBuffer`**: The page must send `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` (or `credentialless`). Without both, `SharedArrayBuffer` is not available.
