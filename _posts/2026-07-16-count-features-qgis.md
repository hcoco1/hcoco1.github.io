---
title: How to Count Features by Attribute in QGIS (4 Methods)
date: 2026-07-18 22:30:00 +0000
categories: [QGIS, Tutorial]
tags: [qgis, sql, aggregate, statistics, gis]
description: Four methods to count the total number of airports and summarize them by country in QGIS.
toc: true
pin: true
render_with_liquid: false
---

This tutorial presents four different approaches to count airports in QGIS. The examples use the **OurAirports** dataset, where each feature represents a single airport.

## Dataset

The layer contains one feature per airport and includes fields such as:

- `id`
- `ident`
- `name`
- `type`
- `iso_country`
- `iso_region`

Since each feature corresponds to one airport:

- **Total airports** = Number of features.
- **Airports by country** = Number of features grouped by `iso_country`.

---

## Method 1 – Count the Total Number of Airports

The fastest method is simply checking the feature count.

### Steps

1. Open the attribute table.
2. Read the value shown in the title bar.

Example:

```
Features Total: 73323
```

Since every feature represents one airport, the dataset contains **73,323 airports**.

---

## Method 2 – Statistics by Categories

This is the simplest tool for counting airports by country.

### Steps

1. Open **Processing Toolbox**.
2. Search for **Statistics by Categories**.
3. Set:

| Parameter | Value |
|-----------|-------|
| Input layer | Airports |
| Field to calculate statistics | `id` |
| Categories field | `iso_country` |

4. Click **Run**.

### Result

The output table contains one record per country, including a **count** column.

Example:

| iso_country | count |
|-------------|------:|
| US | 22,665 |
| CA | 1,468 |
| BR | 5,227 |

This method is ideal when you only need summary statistics.

---

## Method 3 – Aggregate Tool

The **Aggregate** algorithm is similar to SQL's `GROUP BY`.

### Step 1

Open

```
Processing Toolbox → Aggregate
```

### Step 2

Choose the input layer.

### Step 3

Group the records by country.

**Group by expression**

```qgis
"iso_country"
```

This creates one group for every country code.

### Step 4

Define the output fields.

#### Country field

| Parameter | Value |
|-----------|-------|
| Name | `country` |
| Type | Text |
| Aggregate | `first_value` |
| Expression | `"iso_country"` |

`first_value` returns the first value found in each group. Since every feature in the group has the same country code, it correctly returns that country's code.

#### Airport count

| Parameter | Value |
|-----------|-------|
| Name | `airports` |
| Type | Integer |
| Aggregate | `count` |
| Expression | `"id"` |

The `count` aggregate counts the number of airport IDs within each country.

### Result

| country | airports |
|---------|----------:|
| US | 22,665 |
| CA | 1,468 |
| BR | 5,227 |

### Understanding Aggregate Functions

Some commonly used aggregate functions include:

| Function | Description |
|----------|-------------|
| `count` | Counts records |
| `sum` | Adds numeric values |
| `mean` | Computes the average |
| `min` | Smallest value |
| `max` | Largest value |
| `median` | Median value |
| `first_value` | Returns the first value in the group |
| `majority` | Most frequent value |
| `minority` | Least frequent value |
| `concatenate` | Joins text values |

---

## Method 4 – SQL

If you are familiar with SQL, this is often the most flexible solution.

Open

```
Layer → Add Layer → Add/Edit Virtual Layer
```

or

```
Processing Toolbox → Execute SQL
```

Then run:

```sql
SELECT
    iso_country,
    COUNT(*) AS airports
FROM "31_airports"
GROUP BY iso_country
ORDER BY airports DESC;
```

> Replace `"31_airports"` with the actual name of your layer if it is different.

### Result

| iso_country | airports |
|-------------|----------:|
| US | 22,665 |
| BR | 5,227 |
| CA | 1,468 |

Using SQL makes it easy to add additional statistics, for example:

```sql
SELECT
    iso_country,
    COUNT(*) AS airports,
    AVG(elevation_ft) AS avg_elevation,
    MAX(elevation_ft) AS highest_airport
FROM "31_airports"
GROUP BY iso_country
ORDER BY airports DESC;
```

---

## Which Method Should You Use?

| Method | Best Use |
|---------|----------|
| Feature count | Total number of airports |
| Statistics by Categories | Quick counts by country |
| Aggregate | Build summary tables inside QGIS |
| SQL | Complex summaries and reusable queries |

For simple summaries, **Statistics by Categories** is usually the fastest option. When more control over the output is required, the **Aggregate** tool or **SQL** provides much greater flexibility.