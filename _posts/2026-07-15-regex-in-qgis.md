---
title: Regex in QGIS — A Practical Reference
author: <author_id>
date: 2026-07-15 10:00:00 +0200
categories: [QGIS, Tutorial]
tags: [qgis, regex, expressions, field-calculator, data-cleaning]
description: Where regex works in QGIS, the four expression functions, and worked examples for cleaning, extracting, selecting, and labeling.
toc: true
pin: false
image:
  path: https://www.osgeo.org/wp-content/uploads/QGIS-Logo.png
  alt: QGIS 
---

Regex (regular expressions) is a pattern language for matching and manipulating text. In QGIS it is the fastest way to clean attributes, validate IDs, extract values from messy fields, and build label/filter rules. This post covers where regex works in QGIS, the syntax, and worked examples per task.

> No prior regex experience is assumed. Familiarity with the Field Calculator and Select by Expression is enough.
{: .prompt-info }

## Where Regex Works in QGIS

| Location                                   | Regex support                                          |
| ------------------------------------------ | ------------------------------------------------------ |
| Field Calculator                           | `regexp_match`, `regexp_replace`, `regexp_substr`, `~` |
| Select by Expression                       | Same functions                                         |
| Rule-based symbology / labeling            | Same functions                                         |
| Layer filter (Query Builder)               | `~` works for most local formats (provider-dependent)  |
| Attribute table quick search               | No regex — use Select by Expression                    |
| Python console / PyQGIS                    | Python `re` module                                     |

> In QGIS expressions, every backslash must be doubled: write `\\d`, not `\d`. Single backslashes fail silently or raise an error. Python raw strings (`r'\d'`) keep single backslashes.
{: .prompt-danger }

## Syntax Reference

### Character classes

| Pattern    | Matches                    | Example match |
| ---------- | -------------------------- | ------------- |
| `.`        | Any one character          | `a`, `7`, `-` |
| `\\d`      | One digit                  | `5`           |
| `\\D`      | One non-digit              | `x`           |
| `\\w`      | Letter, digit, underscore  | `b`, `9`, `_` |
| `\\s`      | Whitespace                 | space, tab    |
| `[abc]`    | a, b, or c                 | `b`           |
| `[^0-9]`   | Anything except a digit    | `k`           |
| `[A-Za-z]` | Any letter                 | `Q`           |

### Quantifiers

| Pattern   | Meaning                        | `\\d` + quantifier matches |
| --------- | ------------------------------ | -------------------------- |
| `*`       | 0 or more                      | `""`, `4`, `4567`          |
| `+`       | 1 or more                      | `4`, `4567`                |
| `?`       | 0 or 1                         | `""`, `4`                  |
| `{3}`     | Exactly 3                      | `123`                      |
| `{2,4}`   | 2 to 4                         | `12`, `1234`               |
| `*?` `+?` | Lazy — shortest possible match | See [Pitfalls](#pitfalls)  |

### Anchors, groups, alternation

| Pattern   | Meaning                                                      |
| --------- | ------------------------------------------------------------ |
| `^`       | Start of string                                              |
| `$`       | End of string                                                |
| `\\b`     | Word boundary                                                |
| `(...)`   | Capture group — referenced as `\\1`, `\\2` in replacements   |
| `(?:...)` | Group without capturing                                      |
| `a\|b`    | a OR b                                                       |
| `(?i)`    | Case-insensitive from this point                             |

## The Four QGIS Tools

### `~` — quick match test

Returns true/false.

```sql
"ref" ~ '^A\\d+'
```
{: .nolineno }

True for `A12`, `A4`. False for `N12`, `12A`.

### `regexp_match(string, pattern)` — position of match

Returns an integer: position of the first match, or `0` if none.

```sql
regexp_match("name", 'berg') > 0
```
{: .nolineno }

> `regexp_match` returns a position, not a boolean. Always compare with `> 0`.
{: .prompt-warning }

### `regexp_substr(string, pattern)` — extract text

Returns the matched text; if the pattern contains a capture group, returns the group instead.

```sql
regexp_substr('Kerkstraat 12b', '\\d+')          -- → '12'
regexp_substr('Kerkstraat 12b', '\\d+[a-z]?')    -- → '12b'
regexp_substr('N210; A16', '[AN]\\d+')           -- → 'N210' (first match only)
```
{: .nolineno }

### `regexp_replace(string, pattern, replacement)` — substitute

Replaces **all** matches.

```sql
regexp_replace('a  b   c', '\\s+', ' ')                              -- → 'a b c'
regexp_replace('12/07/2026', '(\\d+)/(\\d+)/(\\d+)', '\\3-\\2-\\1')  -- → '2026-07-12'
```
{: .nolineno }

## Worked Examples by Task

### Cleaning attributes (Field Calculator)

Collapse whitespace and trim:

```sql
regexp_replace(trim("name"), '\\s{2,}', ' ')
```
{: .nolineno }

`'  De   Meent '` → `'De Meent'`

Keep digits only:

```sql
regexp_replace("phone", '\\D', '')
```
{: .nolineno }

`'+31 (0)10-123 45 67'` → `'310101234567'`

Normalize street suffixes:

```sql
regexp_replace("street", '(?i)\\s*str(\\.|aat)?$', 'straat')
```
{: .nolineno }

`'Hoofdstr.'` → `'Hoofdstraat'`

Strip HTML tags from imported data:

```sql
regexp_replace("descr", '<[^>]+>', '')
```
{: .nolineno }

Remove parenthetical notes:

```sql
regexp_replace("name", '\\s*\\([^)]*\\)', '')
```
{: .nolineno }

`'Rotterdam (Zuid)'` → `'Rotterdam'`

### Extracting values

Dutch postcode from a full address:

```sql
regexp_substr("address", '\\d{4}\\s?[A-Z]{2}')
```
{: .nolineno }

`'Coolsingel 40, 3011 AD Rotterdam'` → `'3011 AD'`

Year from mixed text:

```sql
regexp_substr("source", '(19|20)\\d{2}')
```
{: .nolineno }

Numeric value out of a unit string, cast to number:

```sql
to_real(regexp_substr("depth_txt", '\\d+\\.?\\d*'))
```
{: .nolineno }

`'12.5 m'` → `12.5`

### Selecting features (Select by Expression)

Find malformed parcel IDs:

```sql
NOT ("parcel_id" ~ '^[A-Z]{2}-\\d{4}-\\d{2}$')
```
{: .nolineno }

Motorways or national roads:

```sql
"ref" ~ '^(A|N)\\d+$'
```
{: .nolineno }

NULL, empty, or whitespace-only fields:

```sql
regexp_match(coalesce("notes", ''), '^\\s*$') > 0
```
{: .nolineno }

Exact word, not substring — `Park` but not `Parkweg`:

```sql
"name" ~ '\\bPark\\b'
```
{: .nolineno }

### Labeling and symbology (rule-based)

Rule filter — label only A-roads:

```sql
"ref" ~ '^A\\d+'
```
{: .nolineno }

Label expression — drop the prefix:

```sql
regexp_replace("ref", '^[AN]', '')
```
{: .nolineno }

Shorten names for small scales:

```sql
regexp_replace("name", '(?i)^(Gemeente|Municipality of)\\s+', '')
```
{: .nolineno }

### Validation before analysis

Coordinates stored as text — check format before casting:

```sql
"lat_txt" ~ '^-?\\d{1,2}\\.\\d+$'
```
{: .nolineno }

Fix comma decimal separators, then cast:

```sql
to_real(regexp_replace("lat_txt", ',', '.'))
```
{: .nolineno }

Flag non-ISO dates:

```sql
NOT ("date_txt" ~ '^\\d{4}-\\d{2}-\\d{2}$')
```
{: .nolineno }

## Python Console / PyQGIS

Standard `re` module; raw strings keep single backslashes.

```python
import re

layer = iface.activeLayer()

pattern = re.compile(r'^\d{4}\s?[A-Z]{2}$')
for f in layer.getFeatures():
    value = f['postcode'] or ''
    if not pattern.match(value):
        print(f.id(), repr(value))
```
{: file="validate_postcodes.py" }

```python
import re

layer.startEditing()
idx = layer.fields().indexOf('name')
for f in layer.getFeatures():
    cleaned = re.sub(r'\s+', ' ', (f['name'] or '').strip())
    layer.changeAttributeValue(f.id(), idx, cleaned)
layer.commitChanges()
```
{: file="clean_names.py" }

> For large layers, prefer the Field Calculator or `processing.run("native:fieldcalculator", ...)` over per-feature loops.
{: .prompt-tip }

## Quick Pattern Cookbook

| Goal                     | Pattern                  |
| ------------------------ | ------------------------ |
| Whole string is digits   | `^\\d+$`                 |
| Integer or decimal       | `^-?\\d+(\\.\\d+)?$`     |
| Dutch postcode           | `^\\d{4}\\s?[A-Z]{2}$`   |
| ISO date                 | `^\\d{4}-\\d{2}-\\d{2}$` |
| Email (rough)            | `^\\S+@\\S+\\.\\S+$`     |
| URL (rough)              | `^https?://`             |
| EPSG code in text        | `EPSG:?\\s?\\d{4,5}`     |
| Contains no letters      | `^[^A-Za-z]*$`           |
| Trailing whitespace      | `\\s$`                   |

## Pitfalls

- **Single backslashes** in expressions fail; `\\d` in expressions, `\d` only in Python raw strings.
- **`regexp_match` returns an integer** — always write `> 0`.
- **NULL breaks matching** — wrap fields with `coalesce("field", '')`.
- **Case sensitivity is the default** — prefix with `(?i)` when needed.
- **Greedy matching over-captures** — on `'<a><b>'`, pattern `'<.*>'` matches the entire string; `'<.*?>'` matches `'<a>'`.
- **`regexp_replace` replaces all occurrences** — anchor with `^` or `$` to limit it.
- **Numeric fields need casting first** — `to_string("code") ~ '^\\d{3}$'`.
- **Escape special characters** to match them literally: `\\.` `\\(` `\\)` `\\[` `\\+` `\\?` `\\*` `\\$` `\\^`.

> Test patterns in the Expression Builder preview against a sample feature before running on the full layer. Prototype complex patterns at [regex101.com](https://regex101.com) (PCRE flavor), then double the backslashes when pasting into QGIS.
{: .prompt-tip }