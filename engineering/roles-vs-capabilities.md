# Roles for staff, capabilities for external parties

A pattern that kept an RBAC system from exploding once the outside world showed up.

**Roles** are for the people who run the product: few, fixed, editable in a matrix
(Super Admin, Admin, Manager). They answer "what may this employee do in the console".

**External parties** (vendors, partners, reps, influencers, ambassadors — N kinds, and the
kind is decided by the business, not by a deploy) must not be roles. Two facts describe
them instead:

- **Capabilities** — what the *organisation* can do (manufacture, design, sell…). Data rows
  in a catalogue with a family, a description, the permission keys they grant and the
  portal section they unlock. Editable from the console; assignable per organisation;
  effective immediately.
- **Membership standing** — a *person's* position inside that organisation (admin / staff).
  It only decides who may manage the organisation's own members and profile.

The server **computes** an external user's permissions per request: a base set ∪ the grants
of the live capabilities their organisation holds ∪ the standing extras. Nothing external is
ever looked up from the roles table, so N capabilities × N kinds never touch the matrix.
Capability gates are enforced server-side (`requireCapability(family)` → 403), not just
hidden in the UI. A staff-only permission key can never be granted through a capability —
validate on write.

**What stays code:** a brand-new portal *module* (new screens) is a drop-in plugin; the
catalogue points at it by id. Everything else — new kinds, what they grant, who has them —
is data.
