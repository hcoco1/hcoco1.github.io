---
title: Writing Expressions in QGIS
author: hcoco1
date: 2026-07-18 14:10:00 +0200
categories: [GIS, QGIS]
tags: [qgis, expressions, filtering, labels]
description: A practical guide to QGIS expressions — filtering, labeling, styling, field calculation, and selection — with copy-ready examples.
toc: true
pin: false
render_with_liquid: false
image:
  path: https://www.osgeo.org/wp-content/uploads/QGIS-Logo.png
  alt: QGIS 
---

Expressions are formulas evaluated once per feature that return a value. QGIS uses them in filtering, labeling, symbology, the field calculator, and selection. The same syntax works everywhere the expression builder appears, so learning it once pays off across the whole application.

This guide is written for **QGIS 3.44 LTR (Solothurn)**. The expression syntax is identical in QGIS 4.x, so it applies there too.

## Quoting rules (read this first)

Most expression errors come from quoting. Three rules:

- Field names use **double quotes**: `"population"`
- Text (string) values use **single quotes**: `'London'`
- Numbers use **no quotes**: `5000`

> `"London"` refers to a field named London. `'London'` is the text value London. Swapping these is the single most common mistake.
{: .prompt-warning }

## The expression dialog

Open it from the **ε** button next to most input fields, or from the attribute table's field calculator. Layout:

- **Left** — function library, grouped (Fields and Values, Operators, String, Math, Conditionals, etc.).
- **Center** — the expression editor.
- **Right** — help for the selected function, plus an output preview showing the result for the current feature.

The preview flags syntax errors and shows the evaluated value. Check it before applying.

## Basic filtering

Filtering limits which features a layer loads and displays. Right-click the layer in the **Layers** panel → **Filter…**, or open **Layer Properties → Source → Query Builder**. Features that evaluate to true stay; the rest are hidden. Clearing the filter restores everything.

> A filter changes what is loaded from the source. It does not select or delete features. The operators below work in the Query Builder; some advanced functions (`concat`, `wordwrap`, `replace`) are only available in labeling, symbology, and the field calculator.
{: .prompt-info }

## Filter using = or IN

Exact match:

```
"country" = 'France'
```

Match any value in a set — shorter than chaining `OR`:

```
"country" IN ('France', 'Germany', 'Spain')
```

## Filter using NOT

Negate a condition:

```
"country" != 'France'
```

or:

```
NOT "country" = 'France'
```

Exclude a set:

```
"country" NOT IN ('France', 'Germany')
```

> `!=` and `<>` both mean "not equal."
{: .prompt-tip }

## Filter using LIKE

`LIKE` matches text patterns using wildcards:

- `%` — any number of characters, including zero
- `_` — exactly one character

```
"name" LIKE 'San%'     -- starts with San
"name" LIKE '%ville'   -- ends with ville
"name" LIKE '%wood%'   -- contains wood
"code" LIKE 'A_'       -- A followed by one character
```

`LIKE` is **case-sensitive**: `'san%'` does not match `San Diego`.

## Combining operators

Use `AND` / `OR` to combine conditions, and parentheses to control grouping:

```
"country" = 'France' AND "population" > 100000
```

```
"country" = 'France' OR "country" = 'Belgium'
```

```
("country" = 'France' OR "country" = 'Belgium') AND "population" > 100000
```

> `AND` binds before `OR`. When you mix both, group explicitly with parentheses or you will get unexpected results.
{: .prompt-warning }

## Filter using numeric data

Numbers are unquoted. Operators: `=  !=  >  <  >=  <=`.

```
"population" > 50000
"elevation" <= 200
"year" >= 2000 AND "year" <= 2010
```

Range shorthand (inclusive of both bounds):

```
"population" BETWEEN 10000 AND 50000
```

> If a numeric column is stored as text, comparisons behave alphabetically. Convert first with `to_int("field")` or `to_real("field")`.
{: .prompt-tip }

## LIKE vs ILIKE

Same wildcards, different case handling:

- `LIKE` — case-sensitive
- `ILIKE` — case-insensitive

```
"name" LIKE 'paris'    -- matches only "paris"
"name" ILIKE 'paris'   -- matches "Paris", "PARIS", "paris"
```

Use `ILIKE` when source data has inconsistent capitalization.

## Vary label size with an expression

Expressions can drive labeling properties. In **Layer Properties → Labels → Text**, click the **data-defined override** button beside **Size**, choose **Edit…**, and enter an expression that returns a number:

```
CASE
  WHEN "population" > 1000000 THEN 14
  WHEN "population" > 100000  THEN 10
  ELSE 7
END
```

Label font size now varies per feature.

## Set symbol size with an expression

The same mechanism applies to symbology. In **Layer Properties → Symbology**, click the data-defined override beside **Size** and enter an expression returning a number:

```
CASE
  WHEN "capacity" > 500 THEN 6
  WHEN "capacity" > 100 THEN 4
  ELSE 2
END
```

To scale proportionally instead of in steps, use the override button's **Assistant**, or compute directly, e.g. `sqrt("capacity") / 5`.

> Watch the size unit (millimeters, points, or map units) set next to the property. It determines how your returned number is interpreted.
{: .prompt-info }

## Set label style (Bold, Italic)

Under **Labels → Text**, the data-defined override beside **Style** accepts a style-name string the font supports:

```
CASE
  WHEN "capital" = 'yes' THEN 'Bold'
  ELSE 'Normal'
END
```

The separate **Bold** and **Italic** overrides instead accept a boolean, so the condition itself is the expression:

```
"capital" = 'yes'
```

This returns true/false to toggle the style.

## Useful label formatting tips

Concatenate fields and text. `||` joins values; `concat()` does the same but skips NULLs:

```
"name" || ' (' || "country" || ')'
```

```
concat("name", ' — pop. ', "population")
```

Add thousands separators:

```
"name" || ': ' || format_number("population", 0)
```

Force a line break inside a label. `char(10)` inserts the line-feed character the labeling engine splits on:

```
"name" || char(10) || "country"
```

Wrap long text automatically at a character count:

```
wordwrap("description", 25)
```

Change case:

```
title("name")   -- Title Case
upper("code")   -- UPPERCASE
lower("email")  -- lowercase
```

> `||` returns NULL if any part is NULL, blanking the entire label. Use `concat()` when a field may be empty.
{: .prompt-tip }

> Do not use `char(13)` (carriage return) for line breaks. It can render as a visible box. Only `char(10)` is needed.
{: .prompt-warning }

## Find and replace with REPLACE and ARRAY

Single replacement — `replace(string, before, after)`:

```
replace("road_type", 'Motorway', 'M')
```

Multiple replacements in one pass — pass two arrays of equal length. Each element in the first is replaced by the element at the same position in the second:

```
replace(
  "road_type",
  array('Motorway', 'A Road', 'B Road'),
  array('M', 'A', 'B')
)
```

`array(...)` builds a list of values. This is far shorter than nesting many two-argument `replace()` calls.

> A `map()` works too: `replace("road_type", map('Motorway','M','A Road','A'))`.
{: .prompt-tip }

## Calculate a new field

Open the attribute table, toggle editing (pencil), then open the **Field Calculator** (abacus icon). To store results permanently, tick **Create a new field**, set a name, type, and length, then enter an expression:

```
round("area_m2" / 10000, 2)
```

This writes a new column — here, area in hectares. Untick **Create a new field** to overwrite an existing column instead. Save edits to commit.

> Field names are capped at 10 characters in Shapefiles. Use GeoPackage to avoid truncation.
{: .prompt-info }

## Select by expression

Selection highlights matching features without hiding the rest. Use **Select Features by Expression** (selection dropdown on the toolbar, or from the attribute table) and enter a condition:

```
"population" > 500000
```

Matching features become selected and can then be exported, edited, or summarized.

The difference from filtering:

| Action | Non-matching features | Purpose |
| ------ | --------------------- | ------- |
| Filter | Hidden                | Restrict the working set |
| Select | Still visible         | Mark a subset to act on |

## Worked example: patterns in British road data

UK roads carry a classification prefix: `M` (motorway), `A` (A-road), `B` (B-road). With a reference field — `ref` in OpenStreetMap data — `LIKE` isolates each class.

All motorways:

```
"ref" LIKE 'M%'
```

A-roads and B-roads together:

```
"ref" LIKE 'A%' OR "ref" LIKE 'B%'
```

Motorways only, excluding A-roads that carry motorway-standard sections such as `A1(M)`:

```
"ref" LIKE 'M%' AND "ref" NOT LIKE 'A%'
```

Label each road with a readable class using `replace` and arrays on the first character:

```
replace(
  left("ref", 1),
  array('M', 'A', 'B'),
  array('Motorway', 'A Road', 'B Road')
)
```

Assign color or size by class with `CASE` in a data-defined override, or filter the layer to a single class for focused mapping.

> Replace `"ref"` with your dataset's actual field name. Inspect the attribute table first to confirm which column holds the road reference.
{: .prompt-tip }

## Reference

Full function list and syntax: QGIS User Manual, **Expressions** chapter at [https://docs.qgis.org](https://docs.qgis.org).