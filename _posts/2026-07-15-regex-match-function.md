---
title: Understanding `regexp_match()` in QGIS
author: hcoco1
date: 2026-07-15 10:00:00 +0200
categories: [QGIS, Expressions]
tags: [qgis, expressions, regex, regexp_match]
description: Learn how the QGIS `regexp_match()` function works, how to interpret its return value, and how to use it to filter features with practical examples.
---


The `regexp_match()` function searches a string using a **regular expression (regex)** and returns the **starting position** of the first match. If the pattern is not found, it returns **0**.

This often causes confusion because when the function is used in tools such as **Extract by Expression** or **Select by Expression**, it appears to behave like a Boolean function. In reality, it always returns an integer, but QGIS interprets non-zero values as **TRUE** and `0` as **FALSE** when a Boolean result is expected.

## Syntax

```qgis
regexp_match(input_string, regex)
```

| Argument       | Description                                                                                                        |
| -------------- | ------------------------------------------------------------------------------------------------------------------ |
| `input_string` | The string to search.                                                                                              |
| `regex`        | The regular expression pattern. Backslashes must be escaped in QGIS expressions (for example `\\d`, `\\s`, `\\b`). |

## Return Value

The function returns:

* **1 or greater** — the position where the first match begins.
* **0** — no match was found.

For example,

```qgis
regexp_match('Hello World', 'World')
```

returns

```text
7
```

because **World** starts at the seventh character.

If the text does not exist,

```qgis
regexp_match('Hello World', 'Earth')
```

returns

```text
0
```

## Why Does It Work in Extract by Expression?

Suppose the expression is

```qgis
regexp_match("ref", '^NH')
```

and the attribute values are:

| ref    | Result |
| ------ | -----: |
| NH-01  |      1 |
| NH45   |      1 |
| SH-09  |      0 |
| ABC123 |      0 |

The **Extract by Expression** algorithm expects a Boolean value.

Internally, QGIS converts the results as follows:

|      Returned value | Boolean |
| ------------------: | ------- |
|                   0 | FALSE   |
| Any non-zero number | TRUE    |

Therefore,

```qgis
regexp_match("ref", '^NH')
```

behaves exactly like

```qgis
regexp_match("ref", '^NH') > 0
```

or

```qgis
regexp_match("ref", '^NH') = 1
```

because the pattern `^NH` can only start at the first character.

## Practical Challenge

Assume the **`karnataka_major_roads`** layer contains a field named `"name"` with the following values:

| Road Name             | Should Match? |
| --------------------- | ------------- |
| 1st Main Road         | ✓             |
| 2nd Main Road         | ✓             |
| 23rd Main Road        | ✓             |
| 101st Main Road       | ✓             |
| Banneghatta Main Road | ✗             |
| Old Main Road         | ✗             |
| Main Road             | ✗             |

The objective is to extract only roads that begin with a numbered ordinal followed by **Main Road**.

### Step 1 — Identify the Pattern

Every valid road follows the same structure:

```text
<number><ordinal> Main Road
```

Examples:

```text
1st Main Road
2nd Main Road
23rd Main Road
101st Main Road
```

### Step 2 — Build the Regular Expression

Break the pattern into smaller pieces.

#### One or More Digits

Regex:

```text
\d+
```

In a QGIS expression:

```text
\\d+
```

#### Ordinal Suffix

Possible suffixes are:

```text
st
nd
rd
th
```

Regex:

```text
(st|nd|rd|th)
```

The vertical bar (`|`) means **OR**.

#### A Space

Regex:

```text
\s
```

QGIS expression:

```text
\\s
```

#### Literal Text

```text
Main Road
```

#### Beginning of the String

To ensure the road name starts with the number:

```text
^
```

#### End of the String

To prevent matches such as **1st Main Road Extension**:

```text
$
```

### Complete Regular Expression

Standard regex:

```text
^\d+(st|nd|rd|th)\sMain Road$
```

QGIS expression:

```qgis
'^\\d+(st|nd|rd|th)\\sMain Road$'
```

## Using `regexp_match()`

```qgis
regexp_match(
    "name",
    '^\\d+(st|nd|rd|th)\\sMain Road$'
)
```

### What Does It Return?

| Road Name             | Result |
| --------------------- | -----: |
| 1st Main Road         |      1 |
| 2nd Main Road         |      1 |
| 23rd Main Road        |      1 |
| 101st Main Road       |      1 |
| Banneghatta Main Road |      0 |
| Main Road             |      0 |

Notice that every valid match begins at the **first character**, so the function returns **1**.

## Understanding the Return Value

Consider the road name:

```text
23rd Main Road
```

The regular expression starts matching immediately:

```text
2 3 r d   M a i n   R o a d
^
```

The match begins at the first character.

Result:

```text
1
```

Now consider:

```text
Banneghatta Main Road
```

The regular expression expects

```text
^\d+
```

which means **the string must begin with one or more digits**.

Instead, the first character is **B**, so no match exists.

Result:

```text
0
```

## Another Example

Suppose the regular expression is simply

```qgis
'Main Road'
```

Now the function searches anywhere in the string.

| Road Name             | Result |
| --------------------- | -----: |
| 1st Main Road         |      5 |
| 23rd Main Road        |      6 |
| Banneghatta Main Road |     13 |

For `"1st Main Road"`:

```text
1 s t _ M a i n _ R o a d
1 2 3 4 5
```

The substring **Main Road** begins at character **5**, so

```qgis
regexp_match("name", 'Main Road')
```

returns

```text
5
```

This demonstrates the real purpose of `regexp_match()`: it tells you **where** the first match occurs, not simply whether it exists.

## `regexp_match()` vs `LIKE`

A `LIKE` expression such as

```qgis
"name" LIKE '%Main Road'
```

would also match

* Banneghatta Main Road
* Old Main Road
* East Main Road

It cannot enforce the rule that the name **must start with a numbered ordinal**.

The regular expression

```qgis
regexp_match(
    "name",
    '^\\d+(st|nd|rd|th)\\sMain Road$'
)
```

matches only names that satisfy all of the following conditions:

* begin with one or more digits;
* are immediately followed by `st`, `nd`, `rd`, or `th`;
* contain a single space;
* end with the exact text `Main Road`.

## Key Points

* `regexp_match()` returns the **starting position** of the first regex match.
* If no match exists, it returns **0**.
* In Boolean contexts, **0** is treated as **FALSE** and any non-zero value as **TRUE**.
* Anchors such as `^` and `$` make it possible to validate the entire string instead of searching for a substring.
* Regular expressions are significantly more powerful than `LIKE` when matching structured text such as road names, parcel identifiers, postal codes, or formatted reference numbers.
