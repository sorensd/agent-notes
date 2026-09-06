# Framework & library selection

Every framework is optimised for a specific shape of problem. "Most popular" is a
popularity metric, usually skewed by whatever category is most common on a platform — it is
**not** a fitness metric. Pick by fit, then check the ecosystem exists. This note is the
decision aid, with the OMS as the worked example.

---

## 1. The first cut: app or content?

Before naming a framework, answer one question, because it eliminates most of the list:

> **Is this a content surface or an application surface?**

| | Content surface | Application surface |
|---|---|---|
| Examples | Marketing site, blog, docs, landing page | Admin console, dashboard, portal, tool |
| Data | Mostly static / build-time | Live, per-request, behind auth |
| SEO | Critical | Irrelevant (behind login) |
| JavaScript | Ship as little as possible | Needed regardless — the page *is* the app |
| Interactivity | A few islands in a static page | The whole page is interactive |
| Winner | **Astro / static SSG** | **SPA (React+Vite) or app-SSR (Next/Remix)** |

Get this wrong and you fight the framework forever. An SSG framework on an authed dashboard
spends its whole budget optimising things you can't collect (no SEO, nothing static to
pre-render), and you end up shipping a UI framework *inside* it anyway. An SPA on a
content/marketing site ships a blank div to crawlers and a slow first paint to users.

**The OMS is an application surface**, so Astro was ruled out despite being the top
Cloudflare framework — that ranking is dominated by content sites on Pages. If Rael Sports
ever needs a public marketing/docs site, Astro is the right pick *there*, coexisting on the
same API.

---

## 2. Web app / rendering frameworks — what each is for

| Framework | Built for | Reach for it when | Not when |
|---|---|---|---|
| **React + Vite (SPA)** | Interactive apps served as static assets + an API | Authed dashboards, consoles, portals; you own a separate API; multi-client reuse | You need SEO or server-rendered content |
| **Astro** | Content-first, JS-light pages (islands) | Marketing, blogs, docs, landing pages; SEO matters | The whole surface is interactive and behind auth |
| **Next.js** | App-SSR + content in one; React Server Components | You need SSR/SEO *and* rich interactivity in one codebase; want file-based routing | A pure static SPA is enough (extra server cost/complexity for nothing) |
| **Remix / React Router (framework)** | Server-first data loading, web-standards forms | SSR app with heavy form/mutation flows tied to routes | You already separate UI from a versioned API cleanly |
| **SvelteKit** | Small bundles, compiled reactivity | Perf-critical, smaller component ecosystem is fine | You depend on the React component/table ecosystem |
| **SolidStart** | Fine-grained reactive perf | Same as Svelte, React-like JSX | You need ecosystem breadth over raw perf |
| **TanStack Start** | Type-safe full-stack React on TanStack primitives | Deeply invested in TanStack Query/Router/Table | You want a larger, more settled ecosystem |

**Rule of thumb:** an application with its own versioned API (which we have — `/api/v1`,
reused by portals and SSO consumers) wants the *thinnest* front end that can render it. A
static React+Vite SPA on Workers Assets is exactly that: zero server-render cost per
request, one bundle, many clients. Adding SSR would add a per-request render step for
content that is 100% client-driven anyway.

---

## 3. The UI resource layer — pick one per row, project-wide

Choosing a framework is only the first decision. An application also needs a component
library, an icon set, table/form/chart tooling. **Pick one per capability and use it
everywhere** — mixing two icon families or two component libraries is how a product stops
reading as one product.

| Capability | Choice for the OMS | Why / alternatives |
|---|---|---|
| **Component library** | **shadcn/ui** (Radix + Tailwind) | You own the code, themes off CSS variables, a11y from Radix. Alts: Radix Themes, Park UI, Mantine, MUI (heavier, opinionated), Chakra |
| **Styling** | **Tailwind + CSS variables** (tokens in `theme.css`) | shadcn is built on it; one token layer feeds every surface. Alts: vanilla-extract, CSS Modules, Panda CSS |
| **Icons** | **Lucide** (one family, everywhere) | Clean, huge set, tree-shakeable, matches shadcn. Alts: Heroicons, Phosphor, Tabler, Radix Icons. **Never mix families.** |
| **Data tables** | **TanStack Table** (headless) | Sorting/filtering/selection/pagination, you own the markup. Alts: AG Grid (heavier, enterprise), MUI DataGrid |
| **Forms** | **React Hook Form + Zod** (`@hookform/resolvers`) | Zod is already the API contract layer — one schema, both sides. Alts: TanStack Form, Formik (dated) |
| **Data fetching / cache** | **TanStack Query** | Caching, revalidation, mutations against `/api/v1`. Alts: SWR, RTK Query |
| **Charts** | **Recharts** or **visx**, or shadcn Charts | Recharts = fast/declarative; visx = low-level control. Add only when a real metric needs it |
| **Dates** | **date-fns** or **Day.js** | Tree-shakeable; avoid Moment (legacy, heavy) |
| **Tables → CSV/print, etc.** | native + small libs as needed | Integrate, don't author |
| **Illustrations / decorative SVG** | **none** | Operational product — no hero blobs, no hand-drawn SVGs (see design-system.md). Icons carry the visual load |
| **Animation** | **CSS transitions first**, Framer Motion only if a flow needs it | Density over motion; don't animate for its own sake |

**Icon packs specifically:** one family per product. Lucide for the OMS. If a needed glyph
is missing, take it from the *same* family's extended set or draw one in the same stroke
weight/grid — do not import a second pack for one icon.

---

## 4. How to actually decide (the checklist)

1. **App or content?** (§1) — eliminates most of the framework list immediately.
2. **Does the ecosystem you depend on exist there?** shadcn + TanStack are React. Choosing a
   non-React app framework means giving those up or re-finding equivalents. Count that cost.
3. **What already runs the platform?** One Worker serves the SPA *and* the API here; a
   framework that wants its own server model is swimming upstream.
4. **Popularity is a tiebreaker, not a reason.** "#1 on Cloudflare" told us Astro is great
   at content, nothing about its fit for an authed console.
5. **Find its skills.** Once the stack is set, pull the ecosystem's agent skills too — don't
   hand-derive what a maintained skill encodes (see `process/skills-and-tooling.md`).
6. **One per capability, project-wide.** One component library, one icon family, one token
   layer, one form stack. Consistency is a feature.

**Do not introduce a second** router, ORM, validation library, auth system, component
library or icon family without a decision recorded here. That is the rule that keeps four
surfaces reading as one product.
