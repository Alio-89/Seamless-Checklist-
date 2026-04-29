# Really Good Trades — Set File Reference

**Location:** `sets/SET-CREATION-REFERENCE.md` — keep this file in the `sets/` folder alongside the set files it describes. Any future work on the project should be pointed to this document at the start of the session.

---

## File structure

Every set lives in `sets/<set-id>.js` and has three parts:

**1. Style IIFE** — injects badge dot CSS into the page at load time. Must be first.

```js
(function() {
  const s = document.createElement('style');
  s.textContent = `
  .b-<id>-base  { background: #hex; }
  .b-<id>-sub1  { background: #hex; }
  `;
  document.head.appendChild(s);
})();
```

**2. `registerSet()` call** — registers the set metadata. Badge class names and `label_short` values must match the subsets list exactly (same strings, same order).

```js
registerSet('<set-id>', {
  label:   'Human Readable Name',
  year:    '2025',
  series:  'Series Name',
  subsets: ['Base', 'Subset A', 'Subset B'],
  badge: {
    'Base':     'b-<id>-base',
    'Subset A': 'b-<id>-suba',
    'Subset B': 'b-<id>-subb',
  },
  label_short: {
    'Base':     'Base',
    'Subset A': 'Sub A',
    'Subset B': 'Sub B',
  }
});
```

**3. Card array** — passed as the third argument to `registerSet()`, or defined separately as `var CARDS_<SET_ID_UPPERCASE> = [...]`. Both work; inline is simpler for new sets.

Each card entry is a four-element array:
```js
['card_number', 'Full Player Name', 'Subset Name', 'Club Name'],
```

---

## Naming conventions

- **Set ID**: lowercase, hyphenated. e.g. `legacy-ultimate-2024`, `eminence-2025`
- **Badge CSS class prefix**: `b-<set-id>-<subset-abbreviation>`. e.g. `b-em-base`, `b-lu24-hfi`
- **Player names**: always full first name and surname. Never `J. Dawson` — always `Jordan Dawson`
- **Club names**: must match exactly — `Adelaide Crows`, `Brisbane Lions`, `GWS GIANTS`, `Gold Coast SUNS` etc.
- **Card numbers**: strings, not integers. `'1'` not `1`. Prefixed subsets use their own scheme e.g. `'NG1'`, `'HFI17'`, `'CC79S'`

---

## Creating a new set from scratch

**Optimal process:**

1. Read `eminence-2025.js` as the formatting reference (it is the canonical template).
2. Write the entire file in a single `create_file` call directly to `/mnt/user-data/outputs/<set-id>.js`. Never split across multiple calls.
3. Order of sections within the file:
   - Top-of-file comments (uncertain names, known issues)
   - Style IIFE
   - `registerSet()` with metadata
   - Card array, subsets in checklist order, each preceded by a section comment

**What to include at the top of the file before writing:**

- Any uncertain player names flagged as comments, e.g.:
  ```js
  // Uncertain names — verify before finalising:
  //   CC79S  Brett Hart (Adelaide) — could be Ben Hart
  //   S36    Josh Buckley (GWS)    — checklist shows J. Buckley
  ```
- Any known data issues (duplicate codes, pending HoF replacements, etc.)

**Inline uncertain flags on individual cards:**

```js
['S36', 'Josh Buckley', 'Stadia', 'GWS GIANTS'],   // uncertain — checklist shows J. Buckley
```

---

## Editing an existing set file

### Small, targeted changes (1–20 lines)

Use `str_replace`. Read the file first with `view` to get the exact current text, then replace only the specific lines that need changing.

```
view  → confirm exact current text
str_replace(old_str=<exact current text>, new_str=<replacement>)
```

Good for: fixing a single player name, updating one card number, applying a HoF replacement, adding a comment to a specific entry.

**Never use `str_replace` on blocks larger than ~20 lines.** It requires the entire old block verbatim as `old_str`, so you end up writing every unchanged line twice — once to match, once in the replacement. On a 270-entry subset that doubles the work and consumes large amounts of context window.

---

### Large-scale changes (renaming many entries, expanding abbreviated names, restructuring subsets)

Use a **Python script via `bash_tool`**. This is always faster and more reliable than repeated `str_replace` calls.

**Pattern: find-and-replace with a dictionary**

```python
# Run via bash_tool
import re

expansions = {
    "'J. Dawson'":    "'Jordan Dawson'",
    "'H. Andrews'":   "'Harris Andrews'",
    "'C. Cameron'":   "'Charlie Cameron'",
    # ... full dictionary
}

with open('/home/claude/<set-id>.js', 'r') as f:
    content = f.read()

for old, new in expansions.items():
    content = content.replace(old, new)

with open('/home/claude/<set-id>.js', 'w') as f:
    f.write(content)

# Verify nothing abbreviated remains
import re
remaining = re.findall(r"'[A-Z]\. [A-Z]", content)
print(f"Remaining abbreviated: {len(remaining)}")
print(remaining[:20])
```

This handles the same player appearing across multiple subsets (e.g. DPS Copper, Gold, and Platinum) automatically — one dictionary entry covers all occurrences.

**Pattern: targeted multi-line block replacement**

When a whole subset needs rewriting, replace just that subset using Python string operations rather than `str_replace`:

```python
with open('/home/claude/<set-id>.js', 'r') as f:
    content = f.read()

old_block = content[content.index('// ── SUBSET NAME'):content.index('// ── NEXT SUBSET')]
new_block = """  // ── SUBSET NAME
  ['X1', 'Full Name', 'Subset', 'Club'],
  ...
"""
content = content.replace(old_block, new_block)

with open('/home/claude/<set-id>.js', 'w') as f:
    f.write(content)
```

---

## Resolving uncertain entries

Uncertain names are flagged in two places simultaneously — a summary block at the top of the file, and an inline comment on the card entry itself.

**The top-of-file block** gives a quick overview of everything outstanding without scanning the whole file:

```js
// Uncertain names — verify before finalising:
//   CC79S  Brett Hart (Adelaide) — could be Ben Hart
//   S36    Josh Buckley (GWS)    — checklist shows J. Buckley
```

**The inline comment** keeps the flag visible in context and makes entries grep-able:

```js
['S36', 'Josh Buckley', 'Stadia', 'GWS GIANTS'],   // uncertain — checklist shows J. Buckley
```

**To find all outstanding flags in a file:**

```bash
grep "uncertain" sets/<set-id>.js
```

**To resolve an uncertain entry:**

1. Verify the correct name (physical card, official checklist, etc.)
2. Update the player name in the card entry
3. Remove the `// uncertain —` comment from that line
4. Remove the corresponding line from the top-of-file summary block
5. Repeat until all are resolved
6. Once the file is fully clean, remove the summary block entirely

A file with no uncertain block at the top is considered verified. A file that still has one should not be treated as final data.

---

## Verification steps (always run before presenting output)

```bash
# 1. No abbreviated names remain
grep -c "'[A-Z]\. " <file>.js
# Expected: 0

# 2. All uncertain flags are present
grep "uncertain\|// NOTE" <file>.js

# 3. Key structural elements present
grep -n "label:\|document.head\|registerSet" <file>.js

# 4. File size is reasonable (sanity check)
wc -c <file>.js
```

---

## Common mistakes to avoid

- **Using `str_replace` on large blocks** — use Python instead for anything over ~20 lines
- **Abbreviated names** — always expand; the database uses full names for trade matching
- **Mismatched subset strings** — the string in `subsets`, `badge`, `label_short`, and the card array must be identical character-for-character
- **Integer card numbers** — must be strings: `'1'` not `1`
- **Missing style IIFE** — badge dots won't render without it; CSS class names must match the `badge` object exactly
- **Splitting a large file write across multiple `create_file` calls** — always write the whole file in one call
- **Rewriting the same players three times** — when Copper, Gold, and Platinum DPS share the same player list, use a Python loop or a single dictionary replacement pass rather than three manual rewrites
