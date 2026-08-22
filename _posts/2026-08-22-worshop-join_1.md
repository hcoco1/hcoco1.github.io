---
title: workshop PostGIS Problem
author: hcoco1
date: 2026-08-22 01:20:00 -0500
categories: [Postgis]
tags: [sql, spatial_sql]
toc: true
description: What neighborhood and borough is Atlantic Commons in?
---


![Problem Join Street](/assets/img/posts/problem_join_street.png)

# What neighborhood and borough is Atlantic Commons in?

The query is answering a simple spatial question:

> **Find the street named `Atlantic Commons`, determine which neighborhood it intersects, and return that neighborhood's name and borough.**

The key is to understand the query in terms of **what each table represents, what is being filtered, and what spatial relationship is being tested**.

---

## 1. Start with the question

We have two geographic objects:

* **Atlantic Commons** → a street
* **Neighborhood** → an area represented by a polygon

So the underlying question is:

> **Which neighborhood spatially relates to the street called Atlantic Commons?**

The borough is then an attribute of that neighborhood.

---

## 2. Look at the two datasets

The query uses two tables:

```sql
nyc_streets
nyc_neighborhoods
```

Think about them like this:

```text
nyc_streets
├── name
└── geom

nyc_neighborhoods
├── name
├── boroname
└── geom
```

`geom` is the geometry column.

Conceptually:

```text
street geometry       → LINESTRING
neighborhood geometry → POLYGON
```

So we have a **line** and a **polygon** that we want to compare spatially.

---

## 3. Find the street we are interested in

This part:

```sql
WHERE s.name = 'Atlantic Commons';
```

means:

> From the streets table, only consider the feature(s) whose name is `Atlantic Commons`.

The alias `s` represents `nyc_streets`:

```sql
FROM nyc_streets AS s
```

So:

```sql
s.name
```

means:

> the `name` column from `nyc_streets`.

At this point, we have identified the street we want to investigate.

---

## 4. Find the neighborhood related to that street

This is the important PostGIS part:

```sql
ON ST_Intersects(n.geom, s.geom)
```

`ST_Intersects()` asks:

> **Do these two geometries intersect?**

Here:

```text
n.geom → neighborhood polygon
s.geom → Atlantic Commons street
```

So PostgreSQL is effectively asking:

> Does this neighborhood polygon intersect the geometry of Atlantic Commons?

If yes, that neighborhood is included in the result.

---

## 5. Why do we need a `JOIN`?

The street and neighborhood information are stored in different tables.

We need:

```text
nyc_streets
     │
     │  spatial relationship
     ↓
nyc_neighborhoods
```

The `JOIN` connects the two datasets based on their **location**, rather than on a shared ID.

Normally, a SQL join might look like:

```sql
ON streets.neighborhood_id = neighborhoods.id
```

But here, the relationship is determined spatially:

```sql
ON ST_Intersects(n.geom, s.geom)
```

That's one of the fundamental ideas in PostGIS:

> **The geometry itself can be used to establish the relationship between features.**

---

## 6. Why `ST_Intersects()`?

Because the question is essentially asking:

> Which neighborhood geometry intersects the street geometry?

So the reasoning is:

```text
Question
   ↓
Find Atlantic Commons
   ↓
Compare its geometry with neighborhood geometries
   ↓
Find the neighborhood(s) that intersect it
```

That becomes:

```sql
ST_Intersects(n.geom, s.geom)
```

---

## 7. What does `SELECT` return?

The first line says:

```sql
SELECT n.name, n.boroname
```

We only want two pieces of information from the matching neighborhood:

```text
n.name
    ↓
Neighborhood name

n.boroname
    ↓
Borough name
```

So the final result might look conceptually like:

```text
name                 boroname
-------------------  --------
Some Neighborhood    Brooklyn
```

---

# Read the query from the inside out

A useful way to understand this query is to mentally execute it in stages.

### Stage 1 — Find Atlantic Commons

```sql
WHERE s.name = 'Atlantic Commons'
```

↓

### Stage 2 — Take its geometry

```text
s.geom
```

↓

### Stage 3 — Compare it with neighborhood geometries

```sql
ST_Intersects(n.geom, s.geom)
```

↓

### Stage 4 — Keep the neighborhoods that intersect it

↓

### Stage 5 — Return their names and boroughs

```sql
SELECT n.name, n.boroname
```

---

# Put the whole thing into plain English

```sql
SELECT n.name, n.boroname
FROM nyc_neighborhoods AS n
JOIN nyc_streets AS s
  ON ST_Intersects(n.geom, s.geom)
WHERE s.name = 'Atlantic Commons';
```

Read it as:

> **Look at the neighborhoods and streets. Find the street named Atlantic Commons. Then find the neighborhood geometries that intersect that street. Finally, return the neighborhood name and borough.**

---

## One important detail

`ST_Intersects()` means **any spatial intersection**.

It does **not** necessarily mean:

> “This is the one neighborhood that contains the entire street.”

A street can cross a neighborhood boundary and therefore intersect multiple neighborhoods.

So the query is technically asking:

> **Which neighborhoods intersect Atlantic Commons?**

Whether that is equivalent to:

> **Which neighborhood is Atlantic Commons in?**

depends on the data and the intended interpretation of the exercise.

That distinction is an important part of learning PostGIS: **understand what the spatial predicate actually asks, not just what the English question seems to imply.**
