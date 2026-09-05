# instructions.md

# Universal Coding Agent Instructions

These instructions apply to every coding task in this repository.

The default behavior is:

**Do not build from scratch when an existing library, package, framework feature, component system, template, SDK, CLI, API, starter kit, plugin, or proven implementation already solves the problem.**

The agent must behave as an integrator first and a code author second.

The goal is to produce:

- less custom code
- fewer tokens
- fewer bugs
- faster implementation
- easier maintenance
- higher-quality UI
- production-grade architecture
- predictable deployments

---

# 1. NEVER REINVENT THE WHEEL

Before writing a meaningful function, component, utility, service, parser, exporter, renderer, authentication system, UI element, or infrastructure feature, first determine whether an established solution already exists.

Examples:

If asked to:

- export PDFs → use an established PDF package
- generate QR codes → use a QR library
- create charts → use a charting library
- make tables sortable/filterable → use a data-table package
- create a date picker → use a date-picker package
- send email → use the deployment platform or framework email service
- resize images → use an established image-processing library
- upload files → use the framework/platform upload system
- create authentication → use an established auth provider/framework
- implement OAuth → use the provider's official SDK
- create forms → use the project's form library/framework
- validate input → use a schema/validation library
- parse CSV → use a CSV parser
- create Excel files → use an Excel library
- create ZIP files → use an archive library
- create icons → use an icon library
- create animations → use an animation library
- implement drag-and-drop → use an existing DnD library
- implement payments → use the payment provider's SDK
- make HTTP calls → use the platform/framework HTTP client
- create caching → use the platform/framework cache
- build a queue → use the deployment platform/framework queue
- implement search → use an existing search solution
- implement rich-text editing → use an established editor
- implement markdown → use a markdown parser
- add syntax highlighting → use an existing syntax highlighting library
- build a calendar → use a calendar component/library

Do not write hundreds of lines of custom code for something available through a maintained package in a few lines.

---

# 2. SEARCH BEFORE CODING

For every feature, perform this sequence before implementing custom code.

## Step 1: Inspect the current project

Check:

- package.json
- requirements.txt
- pyproject.toml
- composer.json
- Cargo.toml
- go.mod
- pom.xml
- build.gradle
- lockfiles
- framework configuration
- existing components
- existing utilities
- existing services
- existing design system
- existing SDKs

Determine whether the project already contains a solution.

Reuse what is already installed whenever reasonable.

---

## Step 2: Check the framework

Before adding another dependency, check whether the active framework already supports the requested feature.

Examples:

- Laravel
- Django
- FastAPI
- Rails
- Next.js
- Nuxt
- SvelteKit
- Astro
- React ecosystem
- Vue ecosystem
- Angular
- Cloudflare Workers
- Supabase
- Firebase
- Shopify
- WordPress

Use built-in framework functionality when it is mature and appropriate.

---

## Step 3: Search for a maintained package

Search the relevant ecosystem.

Examples:

- npm
- PyPI
- Packagist
- crates.io
- Maven Central
- RubyGems
- Go modules
- GitHub
- framework plugin marketplaces

Prefer packages that are:

- actively maintained
- widely used
- reasonably documented
- compatible with the project's stack
- compatible with the project's license
- not obviously abandoned
- not unnecessarily large
- not known to introduce major security concerns

---

# 3. PACKAGE-FIRST IMPLEMENTATION

Before implementing substantial functionality, internally determine:

```text
FEATURE:
EXISTING PROJECT SOLUTION:
FRAMEWORK SOLUTION:
BEST EXISTING PACKAGE:
WHY THIS PACKAGE:
CUSTOM CODE REQUIRED:
```

Custom code should primarily be glue code around existing capabilities.

Do not implement a homegrown alternative simply because it is possible.

---

# 4. CUSTOM CODE IS THE LAST OPTION

Custom implementations are acceptable when:

1. no reasonable maintained package exists
2. available packages are incompatible with the project
3. available packages create unacceptable security risks
4. available packages are disproportionately heavy for a tiny requirement
5. licensing prevents their use
6. the requested functionality is genuinely project-specific
7. the user explicitly requests a custom implementation

Even then, custom code should be as small and modular as possible.

---

# 5. FRONTEND QUALITY IS A HARD REQUIREMENT

The frontend must not look AI-generated.

Avoid generic "AI slop."

This is a hard requirement.

Do not automatically generate:

- giant gradient hero sections
- random purple/blue gradients
- excessive rounded cards
- arbitrary glassmorphism
- meaningless floating blobs
- fake analytics
- decorative SVGs generated from scratch
- random geometric illustrations
- excessive shadows
- giant icon grids
- unnecessary dashboards
- generic SaaS landing-page layouts
- excessive pill-shaped UI
- repetitive card-on-card layouts
- oversized headings with vague marketing text
- dozens of hand-coded CSS rules recreating existing components

Do not create decorative SVG artwork manually unless explicitly requested.

Use real component systems, templates, icon libraries, chart libraries, and design systems.

---

# 6. BEFORE BUILDING A FRONTEND, IDENTIFY THE PAGE TYPE

If the frontend type is not already obvious from the task or repository, determine the intended page type before building.

Examples:

- marketing landing page
- SaaS dashboard
- admin panel
- ecommerce storefront
- product detail page
- checkout
- internal business tool
- settings page
- analytics dashboard
- CRM
- portfolio
- directory
- search interface
- mobile web app
- documentation site
- booking interface
- form workflow

The selected design system or template must match the page type.

Do not use the same dashboard-style design for every project.

---

# 7. USE READY-MADE FRONTEND SYSTEMS

Before manually creating UI components, look for a suitable existing component library or template system.

Examples include, depending on the stack:

- shadcn/ui
- Radix UI
- Material UI
- Ant Design
- Mantine
- Chakra UI
- Bootstrap
- Flowbite
- DaisyUI
- Headless UI
- PrimeReact
- PrimeVue
- Vuetify
- Quasar
- HeroUI
- Tremor
- React Aria
- Shopify Polaris

This list is illustrative, not restrictive.

Select the system that best fits the project.

Do not install a React component library into a non-React application merely because it is popular.

---

# 8. LOOK FOR COMPLETE PAGE COMPONENTS BEFORE ASSEMBLING THEM

For common interfaces, search for existing high-quality components or templates first.

Examples:

- admin dashboards
- login screens
- pricing pages
- checkout interfaces
- settings pages
- tables
- CRM views
- sidebars
- navigation
- command palettes
- analytics screens
- file managers
- calendars
- kanban boards

Prefer adapting a proven layout over designing an entire interface from zero.

---

# 9. USE ICON LIBRARIES

Never manually draw ordinary interface icons.

Use established icon libraries such as:

- Lucide
- Heroicons
- Material Symbols
- Font Awesome
- Phosphor
- Tabler Icons

Use one consistent icon family throughout an interface whenever possible.

---

# 10. USE CHARTING LIBRARIES

Do not hand-code charts using SVG or canvas unless there is a specific technical reason.

Use established libraries such as:

- Chart.js
- ECharts
- Recharts
- ApexCharts
- Highcharts
- Plotly

Choose according to the framework and licensing requirements.

---

# 11. USE REAL DATA STRUCTURES

Do not invent an enormous dashboard full of fake metrics merely to make the interface look populated.

If data is unavailable:

- use realistic placeholders sparingly
- clearly separate mock data from real application behavior
- keep the interface structurally ready for actual data

---

# 12. FRONTEND DESIGN PROCESS

Before implementing a significant UI, establish:

```text
PAGE TYPE:
PRIMARY USER:
PRIMARY ACTION:
FRAMEWORK:
DESIGN SYSTEM:
COMPONENT LIBRARY:
ICON LIBRARY:
CHART LIBRARY, IF NEEDED:
EXISTING TEMPLATE OR STARTER:
RESPONSIVE REQUIREMENTS:
```

Then build using these decisions.

---

# 13. MINIMIZE CSS

Avoid writing large custom stylesheets when utilities or components already exist.

Prefer:

1. existing design-system styles
2. framework utilities
3. component variants
4. design tokens
5. small targeted custom CSS

Custom CSS should solve project-specific presentation problems, not recreate a UI framework.

---

# 14. DO NOT BUILD COMPONENT SYSTEMS INSIDE APPLICATIONS

Unless explicitly requested, do not create a homemade:

- modal framework
- button system
- dropdown system
- tooltip system
- toast system
- form framework
- grid system
- icon framework
- dialog manager
- responsive framework
- animation engine
- theme engine

Use established solutions.

---

# 15. BACKEND DISCOVERY BEFORE IMPLEMENTATION

Before beginning a new backend or significant backend feature, determine the following if it is not already available from the repository or task:

```text
DEPLOYMENT PLATFORM:
PROGRAMMING LANGUAGE:
FRAMEWORK:
DATABASE:
AUTHENTICATION TYPE:
FILE STORAGE:
EMAIL PROVIDER:
BACKGROUND JOB REQUIREMENTS:
CACHE REQUIREMENTS:
EXPECTED SCALE:
EXTERNAL SERVICES/APIS:
```

Do not design infrastructure blindly.

---

# 16. ASK ONLY FOR INFORMATION THAT ACTUALLY CHANGES THE IMPLEMENTATION

Do not interrogate the user unnecessarily.

Infer answers from:

- existing repository
- configuration files
- deployment files
- current architecture
- package dependencies
- previous instructions

Ask only when an unknown materially changes the architecture.

Important backend unknowns may include:

- Cloudflare vs traditional server
- serverless vs persistent server
- Node vs Python vs PHP
- SQLite vs PostgreSQL vs MySQL
- JWT vs sessions vs OAuth
- internal-only vs public API
- single-user vs multi-tenant

---

# 17. DEPLOYMENT PLATFORM COMES FIRST

Backend architecture must account for its deployment environment.

Examples:

## Cloudflare Workers

Prefer:

- Workers APIs
- D1
- KV
- R2
- Durable Objects
- Queues
- Cron Triggers

Do not introduce traditional server architecture that conflicts with Workers.

## Vercel

Prefer solutions compatible with:

- serverless functions
- edge functions
- Next.js
- managed databases
- object storage

## Supabase

Prefer:

- Supabase Auth
- PostgreSQL
- Storage
- Realtime
- Row Level Security

Do not build replacements unnecessarily.

## Firebase

Prefer:

- Firebase Authentication
- Firestore
- Cloud Storage
- Cloud Functions

## AWS

Prefer appropriate managed AWS services where they significantly simplify implementation.

Always investigate what the platform already provides before adding custom infrastructure.

---

# 18. AUTHENTICATION MUST NOT BE INVENTED

Do not write authentication from scratch unless explicitly required.

Prefer established solutions such as:

- framework authentication
- Auth.js
- Clerk
- Supabase Auth
- Firebase Auth
- Better Auth
- Auth0
- Cognito
- Shopify authentication
- OAuth provider SDKs

Never invent:

- password hashing algorithms
- session cryptography
- OAuth flows
- token signing
- password-reset security

Use proven libraries and platform services.

---

# 19. DATABASE WORK

Do not create unnecessary database abstractions.

Prefer:

- framework ORM
- established query builder
- platform-native database client
- established migration system

Examples:

- Prisma
- Drizzle
- SQLAlchemy
- Django ORM
- Laravel Eloquent
- ActiveRecord
- Supabase client
- Cloudflare D1 APIs

Use raw SQL when it is simpler or necessary, but do not build a homemade ORM.

---

# 20. VALIDATION

Use established schema/validation libraries.

Examples:

- Zod
- Valibot
- Joi
- Yup
- Pydantic
- Laravel Validation
- Django Forms/Serializers

Do not create hundreds of custom validation branches when a schema handles the requirement cleanly.

---

# 21. PDF GENERATION EXAMPLE

If the user says:

> Add PDF export.

Do NOT immediately implement a custom PDF renderer.

Instead determine:

```text
What needs to become a PDF?
HTML page?
Invoice?
Table?
Report?
Document?
Server-side or browser-side?
```

Then select an appropriate existing solution.

Potential approaches include:

- HTML-to-PDF package
- browser print/PDF
- Puppeteer/Playwright PDF
- jsPDF
- pdf-lib
- PDFKit
- framework-specific PDF package

Implement only the glue required for the chosen library.

---

# 22. EXCEL EXPORT EXAMPLE

If asked for Excel export:

Do not manually construct XLSX internals.

Use packages such as:

- SheetJS
- ExcelJS
- openpyxl
- framework-specific Excel libraries

---

# 23. IMAGE PROCESSING EXAMPLE

If asked for:

- resize
- crop
- thumbnail generation
- compression
- conversion
- metadata handling

Use an established image library or platform service.

Examples:

- Sharp
- Pillow
- ImageMagick
- Cloudinary
- platform-native image optimization

---

# 24. FILE UPLOAD EXAMPLE

Do not implement upload infrastructure unnecessarily.

First check:

- platform storage
- framework upload support
- S3-compatible storage
- Cloudflare R2
- Supabase Storage
- Firebase Storage

Use signed URLs or official SDKs where appropriate.

---

# 25. API INTEGRATIONS

When integrating an external service:

1. check for an official SDK
2. check official API documentation
3. use the SDK if it reduces complexity
4. avoid rebuilding its HTTP client manually
5. use official types when available

Do not manually recreate an SDK unless necessary.

---

# 26. DO NOT CREATE GIANT UTILITY FILES

Avoid files such as:

```text
utils.js
helpers.js
common.js
functions.js
```

containing hundreds or thousands of lines of unrelated handcrafted behavior.

Prefer:

- established packages
- framework utilities
- small domain-specific modules
- clear service boundaries

---

# 27. DO NOT WRITE CODE TO AVOID A SMALL DEPENDENCY WITHOUT REASON

A maintained dependency is often preferable to 300 lines of custom code.

However, do not install a giant framework to replace a five-line function.

Use proportional judgment.

The goal is:

**minimum total complexity**

not:

**minimum dependency count**

---

# 28. PREFER BORING, PROVEN TECHNOLOGY

Production code should generally favor reliable and established solutions.

Do not choose obscure packages merely because they are novel.

Prefer:

- mature
- documented
- maintained
- understandable
- widely adopted

technology unless the task specifically benefits from something newer.

---

# 29. CHECK PACKAGE HEALTH

Before introducing an important dependency, evaluate at least:

- maintenance activity
- recent releases
- compatibility
- documentation
- adoption
- unresolved critical issues
- security advisories
- license

Avoid abandoned libraries when a maintained alternative exists.

---

# 30. DO NOT REIMPLEMENT PACKAGE SOURCE CODE

Never copy the functionality of an existing package into the project merely to avoid installing the package.

If the project can reasonably depend on the package, use the package.

---

# 31. USE SKILLS, MCP TOOLS, AND AVAILABLE AGENT TOOLS

Before manually solving a technical task, check whether the agent environment exposes:

- skills
- MCP servers
- documentation tools
- package search
- browser tools
- code search
- repository search
- framework documentation
- CLI tools

Use available tools before relying solely on model memory.

---

# 32. DOCUMENTATION-FIRST FOR UNKNOWN APIS

Do not guess API signatures.

When using a library or platform:

- check its current documentation
- verify the version used by the project
- follow official examples
- use current APIs

Do not rely on remembered APIs when documentation is available.

---

# 33. CODE GENERATION SHOULD BE THE FINAL STEP

The expected process is:

```text
Understand request
        ↓
Inspect project
        ↓
Identify platform/framework
        ↓
Search existing dependencies
        ↓
Search framework capability
        ↓
Search maintained packages
        ↓
Search available templates/components
        ↓
Choose smallest proven solution
        ↓
Write glue code
        ↓
Test
```

NOT:

```text
Understand request
        ↓
Immediately generate 800 lines of code
```

---

# 34. MODIFY EXISTING ARCHITECTURE INSTEAD OF DUPLICATING IT

Before creating:

- a new service
- a new API client
- a new component
- a new form system
- a new database helper
- another authentication abstraction

search the repository for existing equivalents.

Extend existing patterns whenever reasonable.

---

# 35. KEEP IMPLEMENTATIONS SMALL

For every implementation ask:

> Can an existing package reduce this code substantially?

If yes, investigate it before continuing.

Large amounts of generated code are not a sign of quality.

The ideal implementation often contains:

- configuration
- package initialization
- a small adapter
- application-specific business logic

rather than thousands of lines recreating generic infrastructure.

---

# 36. HIGH-QUALITY FRONTEND REQUIREMENT

Frontend work should appear intentionally designed by a competent product team.

Priorities:

1. information hierarchy
2. spacing
3. typography
4. interaction clarity
5. consistency
6. responsive behavior
7. accessibility
8. appropriate component selection
9. visual restraint
10. usefulness

Decoration comes after usability.

---

# 37. AVOID AI SLOP

The following should trigger reconsideration:

- everything is inside a card
- every section has an icon
- excessive gradients
- giant glowing headings
- random charts
- fake user avatars
- arbitrary metrics
- excessive blur
- random SVG waves
- five different border radii
- excessive explanatory copy
- every button has an icon
- unnecessary badges everywhere
- repetitive marketing copy
- gradients used merely because they look "modern"

Use restraint.

Professional software often looks simpler than generated mockups.

---

# 38. USE REFERENCES FOR IMPORTANT UI

For a substantial new frontend, identify relevant established patterns before designing.

Examples:

- Shopify-style ecommerce
- Stripe-style developer dashboards
- Linear-style productivity tools
- GitHub-style developer interfaces
- Notion-style content tools
- Airbnb-style marketplace search

Do not clone a company blindly.

Use references to understand interaction patterns, density, navigation, hierarchy, and layout.

---

# 39. RESPONSIVENESS IS REQUIRED

Do not treat mobile responsiveness as an afterthought.

Use the responsive capabilities of the selected component/design system.

Avoid manually writing dozens of media queries when the framework provides responsive utilities.

---

# 40. ACCESSIBILITY

Prefer accessible component libraries that already handle:

- keyboard navigation
- focus management
- ARIA attributes
- screen readers
- dialogs
- menus
- focus traps

Do not hand-build accessibility-sensitive widgets unless necessary.

---

# 41. SECURITY

For security-sensitive functionality, existing vetted libraries are strongly preferred.

Never invent cryptography.

Never implement custom:

- encryption algorithms
- password hashing
- token signing algorithms
- cryptographic randomness
- OAuth security
- CSRF mechanisms

when established framework/platform implementations exist.

---

# 42. TEST THE INTEGRATION, NOT THE LIBRARY

Do not write enormous test suites proving that an established dependency works internally.

Test:

- configuration
- integration
- application-specific behavior
- edge cases specific to this project

Trust the dependency's own tests for its internal behavior.

---

# 43. DO NOT ABSTRACT TOO EARLY

Avoid generating large abstraction layers "for future flexibility."

Build around actual requirements.

Prefer:

```text
Library
  ↓
small project adapter
  ↓
application
```

instead of:

```text
Library
  ↓
custom framework
  ↓
abstraction layer
  ↓
provider layer
  ↓
factory
  ↓
manager
  ↓
application
```

unless the complexity is genuinely required.

---

# 44. WHEN A TASK IS ASSIGNED

At the beginning of implementation, silently answer:

```text
What already exists?

What does the current framework provide?

What maintained package solves this?

What component library solves this?

What deployment service solves this?

What is the smallest amount of code required?
```

Only then begin implementation.

---

# 45. WHEN INFORMATION IS MISSING

For a completely new frontend, determine:

```text
PAGE TYPE
TARGET USERS
PRIMARY ACTION
FRONTEND STACK
```

For a completely new backend, determine:

```text
DEPLOYMENT PLATFORM
LANGUAGE
FRAMEWORK
DATABASE
AUTH TYPE
```

Do not ask again when these answers are already available from the repository or conversation.

---

# 46. DEFAULT PHILOSOPHY

The agent is not being paid by the line of code.

Success is not measured by how much code is generated.

Success is:

- solving the requirement
- using reliable existing technology
- minimizing custom code
- maintaining visual quality
- avoiding duplicated functionality
- reducing bugs
- reducing maintenance
- reducing token consumption
- shipping faster

When five lines using a maintained library solve the problem properly, five lines are better than five hundred.

---

# 47. FINAL RULE

Before writing any substantial custom implementation, ask:

> **Am I rebuilding something that already exists?**

If the answer might be yes:

**stop, search, and reuse first.**