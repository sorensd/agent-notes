# Pull the skills when you pick the tech — and scan before you install

Two rules that belong in every plan, not just when someone remembers them.

## Rule 1 — choosing a technology includes finding its skills

When a plan settles on a technology — a framework, an ORM, an auth library, a platform,
a test runner — **do not stop at "we'll use X."** In the same breath, go find the skills the
ecosystem already has for X. A maintained skill encodes the patterns and traps of that tech
so you do not hand-derive them (badly, slowly) each project.

This is not optional polish. The knowledge you skip is exactly the knowledge that costs you a
day later — the cookie-gated CSRF check, the migration that references deleted seed data, the
per-host cookie isolation. Someone has usually already written the skill.

**Where to look (enumerate, do not guess names):**

- Stack-specific collections — e.g. `secondsky/claude-skills` (Cloudflare, Hono, Drizzle,
  Better Auth, Zod, TanStack, Vitest, shadcn, Turnstile), `jezweb/claude-skills` (Cloudflare +
  Vite scaffolders). These beat a general harness for platform depth.
- General workflow harnesses — e.g. `affaan-m/ECC` (plan/tdd/review/e2e, language packs).
- Registries and directories — `skills.sh`, plugin/marketplace hubs.
- The vendor's own docs (many platforms now ship a Claude Code / agent setup page).

Enumerate real skill names, do not invent them:

```
gh api "repos/<owner>/<repo>/git/trees/<branch>?recursive=1" --jq '.tree[].path' \
  | grep -iE 'SKILL\.md$'
```

Then filter to what you actually use, and mark the exact matches — the tech that is genuinely
in your stack, not adjacent.

## Rule 2 — install nothing unsighted; scan first, every time

A skill is executable trust: it runs with your permissions and can carry prompt injection,
data exfiltration, or destructive commands. Research puts ~26% of skills as vulnerable and
~5% as likely malicious. So:

**Scan every candidate with NVIDIA SkillSpector before it goes anywhere near `~/.claude/skills/`.**

```
# build once (Docker path — no Python/uv needed)
git clone https://github.com/NVIDIA/skillspector && cd skillspector && docker build -t skillspector .

# scan a skill directory (also accepts a git URL or a zip)
docker run --rm -v "$PWD/<skill-dir>:/scan" skillspector scan /scan --no-llm --format json
```

Gate on `risk_assessment.score`:

- **≤ 20** — install-able.
- **21–50** — read the HIGH sub-issues; optionally run the LLM pass (`SKILLSPECTOR_PROVIDER`
  + an API key, drop `--no-llm`) to clear false positives, then decide.
- **> 50** — DO NOT INSTALL.

**Read the noise, do not fear it.** The static pass over-flags:

- *MCP Rug Pull* fires on any skill that merely **names** an MCP tool — usually noise for a
  local-markdown install. (It alone pushed a perfectly good Cloudflare-D1 skill to 61.)
- *Agent Snooping* fires on file/context reads — expected for dev skills.
- *Supply Chain / External Script Fetching* fires on a `curl` in the SKILL.md — that one is
  **real**; open the line and see what it fetches.
- *Data Exfiltration* — real enough to read every time.
- `--no-llm` prints "CAUTION" even at score 0; that is not a verdict — use the number.

**Never run a marketplace's `npx … setup` wholesale.** That installs agents *and hooks that
execute code* on your session lifecycle — a large, unvetted surface. Cherry-pick, scan, then
copy the vetted `<skill>/` into `~/.claude/skills/<name>/`.

**Scanning is not theatre.** Real examples caught this way: a `git-workflow` skill scoring 63
for teaching `git push --force` / `git reset --hard` / `.env` access; a stack-perfect
`cloudflare-d1` skill at 61 on MCP-reference noise (safe on review). One you reject, one you
keep — you only know which by scanning.

## Is a big harness (ECC-style) worth it, or cosmetic?

Both, in parts — judge it honestly:

- **Not cosmetic:** the value is the *infrastructure* — hooks that enforce quality gates
  **outside the context window**, subagents, a memory vault, a built-in security scanner.
  That structurally changes agent behaviour; it is not just more prompt text.
- **But the skills themselves are often curated markdown checklists** — worth real time when
  they encode expertise, not magic, and easy to over-value. A general harness also tends to
  be **shallow on your specific stack** (one had 286 skills and *zero* for the platform in
  use). The platform depth lives in stack-specific collections.
- **So split it:** take the harness for the *loop* (plan → test → review → remember → improve),
  enforcement and memory; take stack-specific collections for the *tech*; **author your own**
  only for the traps the ecosystem does not cover (your project's real scars). Scan all of it.

Expect compounding efficiency from enforcement + memory + not re-deriving — not a 10x jump
from skill text alone.
