# Code smells

The list a smells-only review reads the diff against (after Fowler, _Refactoring_, ch. 3). Each entry is what it looks like, then the fix. All are judgement calls, and a documented repo standard overrides any of them. Skip what the linter or formatter already enforces.

- **Duplicated code**: the same logic in more than one hunk or file of the change. → Extract the shared shape and call it from both.
- **Long function**: a function doing several things, needing comments to mark its parts. → Split it at those parts, each named for what it does.
- **Mysterious name**: a function, variable or type whose name doesn't say what it does or holds. → Rename it; when no honest name comes, the design is unclear.
- **Dead code**: code the change leaves unreachable or unused: branches, parameters, exports, commented-out blocks. → Delete it.
- **Leaky abstraction**: callers that must know a module's internals (its storage, its order of calls, its special cases) to use it. → Move that knowledge behind the interface.
- **Feature envy**: a function that works on another module's data more than its own. → Move it to the data it uses.
- **Data clumps**: the same few values passed around together. → Bundle them into one type.
- **Primitive obsession**: a string or number standing in for a domain concept. → Give the concept a small type of its own.
- **Repeated switches**: the same `switch` or `if` chain on the same value in several places. → One map, or polymorphism, that each place uses.
- **Shotgun surgery**: one logical change forcing edits scattered across many files. → Gather what changes together into one module.
- **Speculative generality**: options, hooks or layers for needs the spec doesn't have. → Inline them away until a real need shows.
- **Middle man**: a function or class that only passes calls on. → Call the real target directly.
