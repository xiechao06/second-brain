---
marp: true
theme: gödel
size: 16:9
---

# Svelte Study Sets

---

## What is the difference between `ssg` and `spa`?

---

---

## How to enable `spa` mode?

---

## `onMount` vs `$effect`

---

## `+layout.js` vs `+layout.server.js`

---

| File | Where load runs |
| --- | --- |
| +layout.js / +page.js | Universal — server on first SSR, then again in the browser. With ssr = false, client only. |
| +layout.server.js / +page.server.js | Server only (and unused at runtime with adapter-static + ssr = false) |

Official rule: disable SSR and universal load functions always run on the client.

---

## Why use `window.location` in `load` function is unreliable?

---

if you redirect in load function like:

```javascript
await goto('/login')
```

then you will hit `load` before `window.location` changes. So parameter `url` should be used like:

```javascript
export const load = async ({url}) => {
    // use url.pathname
}
```

---
