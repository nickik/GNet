# Contributing to the GNet specification

1. State whether a change is architectural, wire-format, algorithmic, or editorial.
2. Never silently change a FROZEN constraint. Add a decision record that explains the incompatibility and migration effect.
3. Mark new numeric values DRAFT until accepted into a registry.
4. For every packet change, update the packet document, relevant registry, and at least one hexadecimal test vector.
5. Use the words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY only for normative requirements.
6. Keep mechanism at the lowest necessary layer: routers forward GDP; endpoints own sessions and security; directories own names.

A project license has not yet been selected. Contributions should not assume a licensing policy until that open question is resolved.
