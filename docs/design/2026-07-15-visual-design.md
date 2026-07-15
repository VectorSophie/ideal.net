# ideal.net — Visual Design Spec

Decided 2026-07-15 in a live design session. This is the surface layer only:
per CLAUDE.md, theme never renames internal concepts (`workspace`, `branch`,
`merge`, `snapshot` stay as-is in code), and every choice below respects the
functional guardrail — nothing ranks, scores, or compares the user's own
workspaces.

## Palette

| Role | Color | Hex |
|---|---|---|
| Background (dark ash) | window/chrome base | `#131317` / panels `#16161a` / wells `#1e1e24` |
| Text (warm ivory) | primary text, leading cube | `#e8e2d0` |
| Muted teal | local/private: active workspace, verified signatures | `#5da9a1` |
| Blue-grey | Tor-routed workspaces, unknown/unverified | `#8a97a8` |
| Brass | shared/inherited, friends' seats, literary accents, numerals | `#b08d57` |
| Vermilion | warnings, outgoing data, the publish act | `#e05b3a` |
| Secondary text | labels, dormant items | `#9a948a` |

Intensity: between "tinted" and "declarative" — the chrome takes a faint tint
of the active workspace's nature (teal local / blue-grey Tor / brass inherited),
accents are visible but not solid-filled, and the vermilion outgoing-data
counter (`▲ n out`) is always present in the toolbar.

## Logo — the slid-cube tesseract

`docs/design/logo.svg` (canonical).

Construction: oblique/cabinet projection following the wetware.engineering
hypercube — square front face, 45° receding depth, thick square-capped strokes.
Two congruent cubes: **ivory cube leads upper-left; teal twin slides down the
descending diagonal**; all eight corner pairs joined by thin ivory ghost lines
(30% opacity). Reads as: the self and its mirror-twin, bound.

- Stroke: 9 units on a 420×440 viewBox, `stroke-linecap: square`, miter joins.
- The logo is the one place square/miter construction lives (see Icons).

## Wordmark

Semibold clean sans (UI stack; Segoe UI on Windows), lowercase:
**`ideal`** in ivory + **`.net`** in teal — the wordmark carries the twin-cube
duality. Sits to the right of the mark, optically centered.
No subtitle in everyday UI.

## Chrome concept — workspace as root, tabs as branches

The approved sidebar/tab visual (session mockup `ui-skin-v6`):

- Each workspace chip is a **root node** (rounded-square chip with ◇, in its
  nature color).
- The **active** workspace grows a tree: a 2px trunk in its color drops from
  the chip and runs beside the tab list; each tab forks off it with a rounded
  git-graph elbow; the **active tab** is boxed (1.5px border, tinted fill) at
  the tip of its branch.
- The trunk **terminates by curving into the last tab** — it never dangles
  past its content.
- **Inactive** workspaces show a short root stub (≈12px, 35% opacity, their
  own color) poking out of the chip — alive but folded.
- Switching workspaces redraws the tree from the new chip in that workspace's
  color. Lines live in chrome only; they never overlay page content.

## Icon language — rounded

Rounded construction (round caps, elbow curves — the tab-tree's DNA), 2px
stroke, ivory default, on a ~28px grid. Approved draft set:

- **branch**: trunk with one elbow fork; origin dot ivory, new tip dot teal
- **merge**: two verticals, one elbows into the other; source dot teal
- **snapshot**: rounded-rect frame with brass serif-italic numeral (Yi Sang's
  numbered poems; versions are numbered, never called "corrections")
- **workspace**: rotated rounded square (the ◇ chip glyph)
- **tor-routed**: small ◇ orbited by two blue-grey arcs
- **publish**: node dot with a short line meeting a large ring (a seat
  approaching the round table)
- **second-opinion**: two mirrored brackets facing each other across a dashed
  centerline

The squared/miter register belongs to the logo alone.

## Relay screen — "the round table" (table + ledger)

Approved layout (mockup `relay-roundtable-v1`, option B):

- **Left: the table sigil.** A brass ring with a faint square grid clipped
  inside it (the network beneath the table). Seats sit on the rim as ◇ chips:
  you (teal, bottom), named friends (brass, petname slugs), unknown publishers
  (blue-grey, dashed outline, "seat ?"). Selecting a seat filters the ledger.
  Seats are all the same size — no standing, no rank, ever.
- **Right: the ledger.** Vertical feed of signed notes. Each card: title,
  then a verification line — `✓ signature verified · <petname> · n sources ·
  date` in teal, or `? unverified — treat as hearsay · unknown seat` in
  blue-grey. Verification is per-note, client-side, always shown.
- **Publish** is a single vermilion outlined pill (`▲ publish…`), bottom-right
  — the one outgoing act on the screen.
- **Empty state**: the bare table fills the screen (option A's atmosphere)
  before any notes exist.
- Header voice: `the round table` in serif italic ivory; status line
  `via tor · <relay>.onion · n seats known` in blue-grey.

## Type system

- **UI**: system sans stack (Segoe UI on Windows), semibold for emphasis.
- **Literary voice**: serif italic (Georgia stack; pair with Noto Serif KR
  for hangul). Reserved for: relay header, snapshot numerals, workspace-
  creation and about/credits copy moments. Never for controls or warnings.
- **Warnings/outgoing data**: always sans, always vermilion — the literary
  voice never delivers safety information.

## Deferred / not designed yet

- Full typography spec (sizes, weights, hangul fallbacks beyond the stack).
- App icon rasterization (16–256px) from logo.svg; needs stroke-weight
  compensation at small sizes.
- Merge/diff UI, snapshot timeline, agent-permission dialogs — designed when
  their features exist (Milestones 2+). The neutral-language rules for them
  are already fixed in CLAUDE.md.
