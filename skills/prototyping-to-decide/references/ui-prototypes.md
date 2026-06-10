# UI Prototypes — the variants-on-a-route shape

Read this when the spike's question is **"what should this look like?"** — layout, information hierarchy, primary affordance. The artifact: several **radically different** UI variants on a single route, switchable from a floating bottom bar. The user flips between them in the browser, picks one (or steals pieces from several), and the rest get deleted.

If the question is about logic or state — wrong shape; use `logic-prototypes.md`.

## Two sub-shapes — strongly prefer A

A variant is much easier to judge **butting up against the real app** — real header, real data, real density. A standalone route is a vacuum where every variant looks fine. Default to A whenever any plausible host page exists.

**A — variants inside an existing page (default).** The route already exists (or the new thing naturally lives inside one — a new dashboard section, a new settings card, a new step in a flow). Variants render on the same route, gated by a `?variant=` search param. Existing data fetching, params, and auth all stay; only the rendered subtree swaps.

**B — a throwaway route (last resort).** Only for genuinely new top-level surfaces with nowhere to embed. Follow the project's routing convention; make `prototype` visible in the path or filename. Sanity-check first: is there *really* no page to host this? An empty route hides exactly the design problems a populated one exposes.

## 1. State the question and pick N

Default **3 variants**; cap at 5 (beyond that is noise, not exploration). One-line plan in a top-of-file comment, mirroring the spike declaration: *"Three variants of the settings page via `?variant=` on the existing `/settings` route."*

## 2. Generate radically different variants

Each variant honors: the page's purpose and available data; the project's component/styling system; a clear exported name (`VariantA`, `VariantB`, `VariantC`).

Variants must be **structurally different** — different layout, different information hierarchy, different primary affordance. Not different colors. Three slightly-tweaked card grids is wallpaper, not a prototype; if two drafts converge, redo one with an explicit constraint ("no card grid").

## 3. Wire the switcher

```tsx
// pseudo-code — adapt to the project's framework/router
const variant = searchParams.get('variant') ?? 'A';
return (
  <>
    {variant === 'A' && <VariantA {...data} />}
    {variant === 'B' && <VariantB {...data} />}
    {variant === 'C' && <VariantC {...data} />}
    <PrototypeSwitcher variants={['A','B','C']} current={variant} />
  </>
);
```

Sub-shape A keeps all existing data fetching above the switcher. Sub-shape B mounts the same switcher on the throwaway route.

## 4. The floating bar

Fixed bottom-center pill: **← arrow / `B — Sidebar layout` label / → arrow**, wrapping cycling.

- Arrows update the URL param via the framework's router (`router.replace` / `navigate`) — shareable, reload-stable.
- `←`/`→` keys also cycle; never intercept when an `<input>`, `<textarea>`, or `[contenteditable]` is focused.
- Visually distinct from the page (high-contrast, shadow) — obviously not part of the design under judgment.
- **Hidden in production builds** (`process.env.NODE_ENV !== 'production'` or equivalent) so a stray merge can't ship it. One shared component, located wherever shared UI lives.

## 5. Hand over, capture, clean up

Surface the URL and variant keys. The valuable feedback is usually composite — "header from B, sidebar from C" — that composite IS the answer; record it in the worklog spike-closed entry (SKILL.md Step 3).

Then: **A** — delete losing variants + switcher, fold the winner into the page. **B** — promote the winner to a real route, delete the throwaway route + switcher. The fold-in is a **rewrite under real discipline** (`testing-by-default`), not a promotion: variant code was written under spike rules. Nothing variant-shaped survives into the final diff — `verifying-done`'s scope check will flag it if it does.

## Anti-patterns

- **Variants differing only in color or copy** — that's a tweak, not a structural alternative.
- **Sharing the layout between variants.** A shared `<Header>` is fine; a shared `<Layout>` defeats the point — each variant must be free to throw the layout away.
- **Wiring variants to real mutations.** Read-only, or stub the writes. The question is "what should this look like," not "does the backend work."
- **Promoting prototype code directly to production.** Rewrite when folding in; the constraints it was written under are the reason it was fast.
