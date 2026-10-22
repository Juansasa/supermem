# supermem

Project logs live in `docs/`. Keep them current in the same turn something is proposed or concluded:

- `docs/PROPOSALS.md`: log every proposal Claude makes as a new `P-NN` with status `open`, in the turn it's made. When the user responds, set `accepted` / `rejected` / `changed`, quote their words, and note where the outcome landed. Never delete a proposal.
- `docs/DECISIONS.md`: agreed decisions only, each with a **Basis** quoting the user and linking its `P-NN` where one exists. Append-only: supersede, never rewrite.
- `docs/DESIGN.md`: agreed design marked ✅ with its `D-NN`. Open parts carry their `P-NN` inline.
- `docs/ARCHITECTURE.md`: Mermaid system map and flows. Update when a decision changes components or flows. Parts resting on open proposals carry their `P-NN`.
- `docs/GOALS.md`: goals, non-goals, anti-goals and success criteria, in the user's words. Open items carry their `P-NN`.

Current stage (D-18): high-level decisions only. Log implementation-level proposals as `deferred`, to be remembered, not decided.

Check `docs/DECISIONS.md` and the open entries in `docs/PROPOSALS.md` before proposing something that may already be decided or pending.
