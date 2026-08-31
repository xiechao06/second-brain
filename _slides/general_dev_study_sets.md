---
marp: true
theme: gödel
size: 16:9
---

# General Dev Study Sets

---

## Clarify `hostname`, `origin` and `site`

---

- `origin` is the part in a URL between protocol and path
- `hostname` is the part in a URL after protocol, before port or path if there's no port
- `site is the registrable domain plus `localhost` and ip, not the port or subdomain.
For example:

| url | site | host | origin |
| --- | --- | --- | --- |
| <http://localhost:8000/a/b/c> | localhost | localhost | localhost:8000 |
| <http://app.example.com:8000/a/b/c> | example.com | app.example.com | app.example.com:8000 |

---

## What is cookie?

---

## What is 