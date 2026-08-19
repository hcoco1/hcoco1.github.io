---
title: GDAL/OGR Guide
author: hcoco1
date: 2026-08-17 01:10:00 -0500
categories: [Programming, Linux]
tags: [bash, cli, linux, sql, gdal]
toc: true
description: Inspecting and Loading the Wildlife Spatial Datasets
---

# GDAL/OGR Guide: Inspecting and Loading the Wildlife Spatial Datasets

## 1. Purpose

This guide documents the workflow used to inspect the supplied GIS files, identify their geometry and coordinate reference systems, detect geometry inconsistencies, and prepare the data for loading into PostgreSQL/PostGIS.

The PostgreSQL target used in the workflow was:

```text
Database: courses
Schema:   miller
User:     ivan
Host:     localhost
Port:     5432
```

The main tools used were:

```bash
ogrinfo
ogr2ogr
psql
```

The workflow was:

```text
Source files
    ↓
ogrinfo inspection
    ↓
Check geometry / CRS / attributes
    ↓
Detect geometry inconsistencies when import fails
    ↓
Adjust ogr2ogr options where necessary
    ↓
Import into PostgreSQL/PostGIS
    ↓
Verify imported tables
```

---

# 2. Understanding the Shapefile components

Several datasets are supplied as groups of files rather than as a single file.

For example:

```text
baea_nests.shp
baea_nests.shx
baea_nests.dbf
baea_nests.prj
baea_nests.sbn
baea_nests.sbx
```

These files belong to the same Shapefile dataset.

## 2.1 `.shp`

The `.shp` file contains the actual geometry.

Examples:

```text
Point
LineString
Polygon
```

This is normally the file passed to `ogrinfo` and `ogr2ogr`.

Example:

```bash
ogrinfo -so -al baea_nests.shp
```

## 2.2 `.shx`

The `.shx` file is the Shapefile geometry index.

It supports access to the records in the `.shp` file.

It must normally remain together with the `.shp` file.

## 2.3 `.dbf`

The `.dbf` file contains the attribute table.

For example:

```text
nest_id
species
status
date
```

The attributes reported by `ogrinfo` come from this part of the Shapefile.

## 2.4 `.prj`

The `.prj` file contains the coordinate reference system definition.

For the spatial datasets inspected in this workflow, GDAL reported:

```text
EPSG:4326
WGS 84
```

This is important because PostGIS needs to know the spatial reference associated with the geometry.

## 2.5 `.sbn` and `.sbx`

These are spatial index files used by some ESRI software.

They are not normally required in the `ogr2ogr` command.

For example:

```text
baea_nests.sbn
baea_nests.sbx
```

can remain alongside the Shapefile, but the import command still references:

```text
baea_nests.shp
```

---

# 3. General inspection procedure

Before importing a spatial file, we used:

```bash
ogrinfo -so -al filename.shp
```

The important information to inspect is:

```text
Layer name
Geometry
Feature Count
Extent
Layer SRS
Data fields
```

For example:

```bash
ogrinfo -so -al baea_nests.shp
```

This showed:

```text
Geometry: Point
Feature Count: 70
EPSG:4326
```

The same process should be used for every Shapefile before importing it.

For Excel files, the same command can be used:

```bash
ogrinfo -so -al filename.xlsx
```

The XLSX driver reports the sheet/layer, fields, feature count, and whether geometry exists.

---

# 4. BAEA datasets

## 4.1 `baea_nests`

Files:

```text
baea_nests.dbf
baea_nests.prj
baea_nests.sbn
baea_nests.sbx
baea_nests.shp
baea_nests.shx
```

Inspection command:

```bash
ogrinfo -so -al baea_nests.shp
```

Results:

```text
Layer:        baea_nests
Geometry:     Point
Feature Count: 70
CRS:          EPSG:4326
```

Fields:

```text
postgis_fi   Integer64
lat_y_dd     Real
long_x_dd    Real
status       String
nest_id      Integer64
```

### Interpretation

This is a spatial point dataset.

No geometry-type problem was identified during inspection, so the normal PostGIS import command can be used.

Import:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  baea_nests.shp \
  -nln miller.baea_nests \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

The expected PostGIS geometry is:

```text
Point
SRID 4326
```

---

# 5. BAEA surveys

File:

```text
baea_surveys.xlsx
```

Inspection:

```bash
ogrinfo -so -al baea_surveys.xlsx
```

Results:

```text
Layer:        survey_results
Geometry:     None
Feature Count: 2000
SRS:          unknown
```

Fields:

```text
id       Integer
nest     Integer
user     String
date     Date
result   String
```

### Interpretation

This is not a spatial dataset.

There is no geometry and no CRS.

Therefore it should be imported as a normal PostgreSQL table rather than as a PostGIS geometry table.

Import:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  baea_surveys.xlsx \
  -nln miller.baea_surveys
```

No geometry options are required.

---

# 6. GBH rookery dataset

Files:

```text
gbh_rookeries.dbf
gbh_rookeries.prj
gbh_rookeries.sbn
gbh_rookeries.sbx
gbh_rookeries.shp
gbh_rookeries.shx
```

Initial inspection:

```bash
ogrinfo -so -al gbh_rookeries.shp
```

GDAL reported:

```text
Geometry: Polygon
Feature Count: 55
CRS: EPSG:4326
```

Fields:

```text
postgis_fi   Integer64
species      String
activity     String
```

## 6.1 Import error

The initial import used the default geometry type detected by GDAL:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  gbh_rookeries.shp \
  -nln miller.gbh_rookeries \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

The import failed with:

```text
Geometry to be inserted is of type Multi Polygon,
whereas the layer geometry type is Polygon.
```

and:

```text
ERROR: Geometry type (MultiPolygon)
does not match column type (Polygon)
```

## 6.2 Why this happened

The layer summary reported:

```text
Polygon
```

but at least one actual feature was:

```text
MultiPolygon
```

Therefore the PostGIS column created as:

```text
geometry(Polygon,4326)
```

could not accept a `MultiPolygon`.

## 6.3 Checking geometry types

The geometry types can be investigated with SQLite SQL through GDAL:

```bash
ogrinfo gbh_rookeries.shp \
  -dialect SQLite \
  -sql "SELECT ST_GeometryType(geometry), COUNT(*) FROM gbh_rookeries GROUP BY ST_GeometryType(geometry)"
```

This is preferable to assuming that the layer-level geometry description represents every feature.

## 6.4 Corrected import

Use:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  gbh_rookeries.shp \
  -nln miller.gbh_rookeries \
  -nlt MULTIPOLYGON \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

The important option is:

```text
-nlt MULTIPOLYGON
```

This makes the target geometry compatible with the actual source features.

---

# 7. Linear projects dataset

Files:

```text
linear_projects.dbf
linear_projects.prj
linear_projects.sbn
linear_projects.sbx
linear_projects.shp
linear_projects.shx
```

Inspection:

```bash
ogrinfo -so -al linear_projects.shp
```

GDAL reported:

```text
Geometry: Line String
Feature Count: 1109
CRS: EPSG:4326
```

Fields:

```text
postgis_fi   Integer64
type         String
row_width    Real
Project      Integer64
```

## 7.1 Import error

The initial import was:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  linear_projects.shp \
  -nln miller.linear_projects \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

It failed with:

```text
Geometry to be inserted is of type Multi Line String,
whereas the layer geometry type is Line String.
```

and:

```text
ERROR: Geometry type (MultiLineString)
does not match column type (LineString)
```

## 7.2 Cause

Again, the layer-level geometry was reported as:

```text
LineString
```

but one or more actual features were:

```text
MultiLineString
```

## 7.3 Geometry check

The geometry types can be checked using:

```bash
ogrinfo linear_projects.shp \
  -dialect SQLite \
  -sql "SELECT ST_GeometryType(geometry), COUNT(*) FROM linear_projects GROUP BY ST_GeometryType(geometry)"
```

## 7.4 Corrected import

Use:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  linear_projects.shp \
  -nln miller.linear_projects \
  -nlt MULTILINESTRING \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

The important option is:

```text
-nlt MULTILINESTRING
```

---

# 8. Raptor nests

Files:

```text
raptor_nests.dbf
raptor_nests.prj
raptor_nests.sbn
raptor_nests.shp
raptor_nests.shx
```

Inspection:

```bash
ogrinfo -so -al raptor_nests.shp
```

Results:

```text
Geometry: Point
Feature Count: 876
CRS: EPSG:4326
```

Fields:

```text
postgis_fi   Integer64
lat_y_dd     Real
long_x_dd    Real
lastsurvey   Date
recentspec   String
recentstat   String
Nest_ID      Integer64
longitude    Real
latitude     Real
```

### Import

No geometry conversion option was required.

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  raptor_nests.shp \
  -nln miller.raptor_nests \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

Expected geometry:

```text
Point
SRID 4326
```

The table contains coordinate attributes such as:

```text
latitude
longitude
```

but those are attributes. The actual spatial geometry comes from the Shapefile geometry and is stored in the PostGIS `geom` column.

---

# 9. Raptor surveys

File:

```text
raptor_surveys.xlsx
```

Inspection:

```bash
ogrinfo -so -al raptor_surveys.xlsx
```

Results:

```text
Layer:        survey_results
Geometry:     None
Feature Count: 2000
SRS:          unknown
```

Fields:

```text
id       Integer
nest     Integer
user     String
date     Date
result   String
```

### Interpretation

Like `baea_surveys.xlsx`, this is a non-spatial table.

Import:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  raptor_surveys.xlsx \
  -nln miller.raptor_surveys
```

No geometry options are necessary.

---

# 10. Buowl datasets

Files provided:

```text
buowl_habitat.dbf
buowl_habitat.prj
buowl_habitat.shp
buowl_habitat.shx
buowl_surveys.xlsx
```

These files were identified as part of the dataset collection, but their `ogrinfo` output was not recorded during the workflow documented above.

Therefore their exact geometry type, feature count, CRS, and attribute definitions should be inspected before importing them.

## 10.1 Habitat Shapefile

Run:

```bash
ogrinfo -so -al buowl_habitat.shp
```

Record at least:

```text
Layer name
Geometry
Feature Count
Extent
Layer SRS
Data fields
```

Then determine whether a geometry-type conversion is necessary before running `ogr2ogr`.

The expected import pattern is:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  buowl_habitat.shp \
  -nln miller.buowl_habitat \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

If inspection or an import error reveals mixed geometry, use the appropriate `-nlt` option, for example:

```text
-nlt MULTIPOLYGON
```

or:

```text
-nlt MULTILINESTRING
```

depending on the actual source geometry.

## 10.2 Habitat survey spreadsheet

Inspect:

```bash
ogrinfo -so -al buowl_surveys.xlsx
```

If it reports:

```text
Geometry: None
```

then import it as a regular PostgreSQL table:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  buowl_surveys.xlsx \
  -nln miller.buowl_surveys
```

The exact schema should be confirmed from `ogrinfo` before import.

---

# 11. Important lesson: layer geometry versus feature geometry

One of the most important findings from this workflow was that the geometry reported by:

```bash
ogrinfo -so -al
```

should not always be treated as proof that every feature has exactly that geometry type.

Two examples demonstrated this.

### `gbh_rookeries`

Layer:

```text
Polygon
```

Actual feature:

```text
MultiPolygon
```

Solution:

```text
-nlt MULTIPOLYGON
```

### `linear_projects`

Layer:

```text
LineString
```

Actual feature:

```text
MultiLineString
```

Solution:

```text
-nlt MULTILINESTRING
```

This is why checking geometry types becomes important when an apparently valid `ogr2ogr` import fails.

---

# 12. PostgreSQL/PostGIS import patterns

## Spatial Shapefile

General form:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  source.shp \
  -nln miller.target_table \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

## Spatial Shapefile with multiple geometry types

Use:

```bash
-nlt MULTIPOLYGON
```

for polygon/multipolygon data when required.

Use:

```bash
-nlt MULTILINESTRING
```

for line/multiline data when required.

## Non-spatial Excel table

General form:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  source.xlsx \
  -nln miller.target_table
```

No geometry options are required.

---

# 13. Verification after import

After loading a spatial layer, inspect it from PostgreSQL with `ogrinfo`:

```bash
ogrinfo -so \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  miller.table_name
```

A SQL check is also useful:

```sql
SELECT
    COUNT(*) AS feature_count,
    ST_SRID(geom) AS srid,
    ST_GeometryType(geom) AS geometry_type
FROM miller.table_name
GROUP BY ST_SRID(geom), ST_GeometryType(geom);
```

For a non-spatial table:

```sql
SELECT COUNT(*)
FROM miller.table_name;
```

To inspect tables in `psql`:

```sql
\dt miller.*
```

To inspect columns:

```sql
\d miller.table_name
```

---

# 14. Final dataset classification

| Dataset           | Files                           | Type      | Geometry                     | CRS               | Special handling           |
| ----------------- | ------------------------------- | --------- | ---------------------------- | ----------------- | -------------------------- |
| `baea_nests`      | `.shp/.shx/.dbf/.prj/.sbn/.sbx` | Shapefile | Point                        | EPSG:4326         | Standard import            |
| `baea_surveys`    | `.xlsx`                         | Excel     | None                         | Unknown           | Regular PostgreSQL table   |
| `buowl_habitat`   | `.shp/.shx/.dbf/.prj`           | Shapefile | Not yet inspected            | Not yet confirmed | Inspect before import      |
| `buowl_surveys`   | `.xlsx`                         | Excel     | Not yet inspected            | Not yet confirmed | Inspect before import      |
| `gbh_rookeries`   | `.shp/.shx/.dbf/.prj/.sbn/.sbx` | Shapefile | Polygon / MultiPolygon       | EPSG:4326         | Use `-nlt MULTIPOLYGON`    |
| `linear_projects` | `.shp/.shx/.dbf/.prj/.sbn/.sbx` | Shapefile | LineString / MultiLineString | EPSG:4326         | Use `-nlt MULTILINESTRING` |
| `raptor_nests`    | `.shp/.shx/.dbf/.prj/.sbn`      | Shapefile | Point                        | EPSG:4326         | Standard import            |
| `raptor_surveys`  | `.xlsx`                         | Excel     | None                         | Unknown           | Regular PostgreSQL table   |

---

# 15. Recommended workflow for the remaining files

For every unprocessed file, use this sequence:

### Shapefile

```bash
ogrinfo -so -al filename.shp
```

Then:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  filename.shp \
  -nln miller.table_name \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

If the import reports:

```text
Geometry type (...) does not match column type (...)
```

inspect the actual geometry types before changing the import:

```bash
ogrinfo filename.shp \
  -dialect SQLite \
  -sql "SELECT ST_GeometryType(geometry), COUNT(*) FROM layer_name GROUP BY ST_GeometryType(geometry)"
```

Then select the appropriate `-nlt` option.

### Excel

First inspect:

```bash
ogrinfo -so -al filename.xlsx
```

If:

```text
Geometry: None
```

import as a normal PostgreSQL table:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  filename.xlsx \
  -nln miller.table_name
```

---

# 16. Key commands used in this exercise

Inspect a dataset:

```bash
ogrinfo -so -al filename.shp
```

Inspect Excel:

```bash
ogrinfo -so -al filename.xlsx
```

Inspect geometry types:

```bash
ogrinfo filename.shp \
  -dialect SQLite \
  -sql "SELECT ST_GeometryType(geometry), COUNT(*) FROM layer_name GROUP BY ST_GeometryType(geometry)"
```

Import a spatial layer:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  filename.shp \
  -nln miller.table_name \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid
```

Import a non-spatial table:

```bash
ogr2ogr \
  -f PostgreSQL \
  "PG:host=localhost port=5432 dbname=courses user=ivan password=postgres" \
  filename.xlsx \
  -nln miller.table_name
```

Connect to PostgreSQL:

```bash
psql -h localhost -U ivan -d courses
```

List tables:

```sql
\dt miller.*
```

Inspect a table:

```sql
\d miller.table_name
```

---

# 17. Conclusion

The main practical lessons from these datasets are:

1. A Shapefile is a group of related files. The `.shp` file is the file passed to GDAL, but `.shx`, `.dbf`, and `.prj` are important supporting files.

2. `ogrinfo -so -al` should be used before importing data to determine geometry, feature count, CRS, and fields.

3. `ogr2ogr` uses essentially the same import pattern for points, lines, and polygons.

4. A layer may report `Polygon` or `LineString` while individual features can contain `MultiPolygon` or `MultiLineString` geometries.

5. Geometry mismatch errors during import should be investigated rather than ignored.

6. `-nlt MULTIPOLYGON` and `-nlt MULTILINESTRING` can be used when the target geometry needs to accommodate the actual source features.

7. Excel files with `Geometry: None` are imported as ordinary PostgreSQL tables, not spatial PostGIS tables.

8. The final target for this exercise is the `miller` schema inside the `courses` database.
