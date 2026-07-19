---
title: GIS Fundamentals — A Pre-Course Primer for QGIS
author: hcoco1
date: 2026-07-11 10:00:00 +0200
categories: [GIS, Tutorial]
tags: [qgis, gis, projections, crs, spatial-data]
render_with_liquid: false
pin: true
image:
  path: https://thumbs.dreamstime.com/b/gis-geographic-information-system-geospatial-technology-urban-planning-spatial-analysis-digital-map-interface-smart-city-438034315.jpg?w=768
  alt: QGIS 
---

This guide covers the concepts you need before starting the QGIS course: the spatial data model, storage formats, and coordinate reference systems (CRS). CRS and projections are the main source of confusion for new users, so most of the guide is spent there. No software steps are included; the course covers those.

> No prior GIS experience is assumed. Basic familiarity with files and folders is enough.
{: .prompt-info }

## Spatial Data Model

Spatial data combines **geometry** (location) with **attributes** (information about that location). A dataset is spatial only when both are present and linked.

- **Geometry** — where the object is (point, line, polygon; or a pixel grid).
- **Attributes** — what the object is (name, type, population, measurement, date).

In QGIS, attributes are held in the **attribute table** — one row per feature, one column (**field**) per property. Selecting a feature on the map highlights its row, and vice versa. Analysis works by relating geometry to attributes (e.g. "select all polygons where `landuse = residential` within 500 m of a river").

## Attributes: Field Types and NULL

Every field has a **type**, fixed when the layer is created. The type controls what the field can store and how expressions and statistics treat it. Most beginner expression errors come from a type mismatch.

| Type            | Stores                     | Example        | Notes                                              |
| --------------- | -------------------------- | -------------- | -------------------------------------------------- |
| **Integer**     | Whole numbers              | `42`, `-3`     | Counts, codes, year                                |
| **Decimal/Real**| Numbers with fractions     | `3.14`         | Measurements, coordinates                          |
| **Text/String** | Characters                 | `residential`  | Has a length limit; codes with leading zeros must be text |
| **Boolean**     | True / false               | `true`         | Some formats store this as `0`/`1`                 |
| **Date/DateTime**| Calendar dates/timestamps | `2026-07-11`   | Not the same as text; enables date arithmetic      |

**NULL** means "no value recorded." It is **not** `0`, not empty text, and not `false`. Expressions treat NULL specially: it usually propagates (arithmetic on NULL returns NULL), and summaries like count or sum skip it. Test with `field IS NULL`, not `field = ''` or `field = 0`.

**Feature ID (FID / ObjectID)** is an internal row identifier, not analytical data. It can change when you export or re-save a layer, so do not treat it as a stable key unless you have deliberately made one.

> A number stored as text will not sort or calculate as a number, and a date stored as text will not support date math. Check the field type before writing expressions.
{: .prompt-warning }

## Spatial Data Types

### Vector data

Discrete features defined by coordinates.

| Type        | Models              | Examples                          |
| ----------- | ------------------- | --------------------------------- |
| **Point**   | A precise location  | City, sensor, tree, address       |
| **Line**    | A linear feature    | Road, river, pipeline, track      |
| **Polygon** | An enclosed area    | Building, parcel, lake, boundary  |

### Point clouds

Dense **X, Y, Z** observations from lidar or photogrammetry. A small area can hold millions to billions of points, requiring specialized formats and processing separate from ordinary vector points.

### Raster data

A grid of pixels (cells), each holding a value.

- **Imagery** — georeferenced photos from drones, aircraft, or satellites (often multi-**band**: red, green, blue, near-infrared).
- **Continuous grids** — one value per cell representing a continuous surface (elevation, temperature, population density).
- **Mesh** — time-varying values at continuous locations; common for weather/ocean data. A modern alternative to stacked rasters.

### Tiles

Data pre-cut into small tiles for fast web delivery. Both **raster tiles** and **vector tiles** exist; the client fetches only the tiles it currently needs. This is how web basemaps work.

### Vector vs. raster — how to choose

| Use **vector** when                       | Use **raster** when                    |
| ----------------------------------------- | -------------------------------------- |
| Features are discrete with clear edges    | Data is continuous across space        |
| You need attributes per feature           | You need per-cell values               |
| Precise geometry and topology matter      | Imagery, surfaces, or model output     |
| Example: parcels, roads, boundaries       | Example: satellite image, DEM, rainfall|

Raster **resolution** is the ground size of one cell (e.g. 10 m). Vector data is captured at a **scale** it is fit for; data digitized for a national map should not be trusted for street-level measurement.

## Spatial Data Formats

| Category        | Non-spatial analog | Spatial format(s)                          |
| --------------- | ------------------ | ------------------------------------------ |
| Text (vector)   | CSV, JSON          | CSV with lat/long columns, GeoJSON, KML/GPX|
| Binary (vector) | Excel, PDF         | Shapefile, GeoPackage                      |
| Point cloud     | —                  | LAS, LAZ (compressed)                      |
| Imagery/raster  | PNG, JPEG          | GeoTIFF, JPEG 2000                         |
| Database        | PostgreSQL         | PostGIS (vector + raster in-database)      |

### Recommended default: GeoPackage, not Shapefile

Shapefile is common but dated, and its limits cause real problems:

- It is several files (`.shp`{: .filepath}, `.shx`{: .filepath}, `.dbf`{: .filepath}, `.prj`{: .filepath}, and more) that must travel together.
- **2 GB** size limit per file.
- Field names capped at **10 characters** (names get silently truncated).
- Weak field types and text-encoding issues (accented characters can corrupt).
- One geometry type, one layer per file.

`.gpkg`{: .filepath} (GeoPackage) is a single **SQLite** file that holds multiple layers (vector, raster, and saved styles), supports long field names and larger sizes, and is an open OGC standard.

> Use GeoPackage as your working default unless a task specifically requires another format.
{: .prompt-tip }

> Moving a `.shp`{: .filepath} without its sidecar files loses data. Losing the `.prj`{: .filepath} file loses the layer's CRS entirely.
{: .prompt-danger }

### Modern/cloud formats

**Cloud-Optimized GeoTIFF (COG)**, **GeoParquet**, **FlatGeobuf**, and **PMTiles** are designed for large datasets and cloud/streaming access. You will meet them as data volumes grow. Not required for the course.

> Under the hood, QGIS reads and writes these formats through **GDAL/OGR**, the same libraries used by most GIS software. Format compatibility issues are usually GDAL issues.
{: .prompt-info }

## Data Quality: Metadata, Accuracy, Precision, Scale

Every operation you run is only as trustworthy as the data going in. Evaluate a dataset before using it.

### Metadata

**Metadata** is the data about the data. Good metadata records at least: **CRS**, source/creator, date, capture **scale**, positional and attribute accuracy, attribute (field) definitions, and licensing. Without it you cannot judge whether a dataset fits your task, and you may not even know its CRS.

> Before trusting any dataset, inspect its metadata — CRS and capture scale first.
{: .prompt-tip }

### Accuracy vs precision

These are different, and beginners routinely conflate them.

- **Accuracy** — how close a value is to the true value.
- **Precision** — how repeatable a measurement is, or how many digits are reported. High precision does not imply high accuracy: coordinates printed to six decimals can still be in the wrong place.
- **Positional accuracy** (is the location right?) is separate from **attribute accuracy** (is the recorded value right?). A perfectly placed point can carry a wrong population figure, and vice versa.

Storage precision is not accuracy. A GPS reading of `Accuracy: ±3 m` is an **error estimate**, not a guarantee of exactness. A **30 m DEM** stores one elevation per 30 m cell — that is its resolution, not proof that any elevation is correct to 30 m.

### Scale

Vector data has a **capture scale** it was digitized for. `1:250,000` and `1:10,000` datasets of the same area can produce very different results; the coarser one omits detail the finer one captures. Zooming in within QGIS enlarges vertices and pixels but **adds no accuracy** — the data is exactly as detailed as when it was captured.

## What GIS Is (and what QGIS is)

**GIS = Geographic Information System** — software that stores, displays, and analyzes spatial data. This is the everyday meaning.

A related but distinct term, **Geographic Information Science (GIScience)**, is the academic discipline. The *system* is the tooling used within the *science*.

### QGIS specifically

- **Free and open source** (GPL); no license cost, cross-platform (Windows, macOS, Linux).
- Built on established engines: **GDAL/OGR** (data I/O), **PROJ** (projections and datum transforms), **GEOS** (geometry operations). The same math underpins many commercial tools.
- Extensible through **plugins** (Python).
- Connects directly to **PostGIS**, SpatiaLite, and web services (WMS/WMTS/WFS, XYZ tiles).

## What GIS Does — and where it lives in QGIS

| Capability                             | In QGIS you will use            |
| -------------------------------------- | ------------------------------- |
| **View** — display, select, zoom       | **Layers panel**, map canvas    |
| **Browse/connect** to files and DBs    | **Browser panel**               |
| **Create/edit** geometry and attributes| Editing toolbar, digitizing tools|
| **Analyze** spatial relationships      | **Processing Toolbox**          |
| **Make maps** for print                | **Print Layout**                |
| **Transform** formats (ETL)            | Export / Processing / GDAL tools|

The **Layers panel** controls draw order and visibility, the **Browser panel** is how you add data, and the **Processing Toolbox** is where nearly all analysis lives.

## Modeling the Earth's Surface

Analysis needs a mathematical model of the Earth. Three levels, simple to accurate:

1. **Sphere** — a decent approximation. Mean radius ≈ **6,371 km**. Simple math, limited accuracy.
2. **Ellipsoid** — flattened at the poles, so two radii. WGS84: semi-major **6,378.137 km**, semi-minor ≈ **6,356.752 km**. Accurate enough for calculation, and the surface all math is done on.
3. **Geoid** — the true, lumpy surface of equal gravity (historically approximated by mean sea level). Most accurate, but unusable for direct calculation.

> You cannot compute on the geoid, so you compute on an ellipsoid chosen to fit it well. Which ellipsoid, and how it is positioned and oriented, is defined by the **datum**.
{: .prompt-info }

## Datums

A **datum** ties an ellipsoid to the real Earth.

- **Global (geocentric) datums** — ellipsoid centered at Earth's center of mass, best global fit. Used for global work; made practical by GPS.
- **Local datums** — ellipsoid oriented to fit one region tightly (minimum local distortion), with its center offset from Earth's center. Accurate locally, poor elsewhere. Common in older data, still in use.

### Worked example — geoid separation

At Nandi Hills, India:

- Signposted elevation (mean sea level / geoid): **1,478 m**.
- GPS reading (WGS84 ellipsoid): **1,393 m**.

Both are correct against different references. The **~85 m gap is the geoid separation**, and it varies by location. Practical consequence: latitude/longitude values are ambiguous without their datum — the same coordinates on different datums can point to noticeably different places.

### Example datums

| Datum                    | Type            | Notes                                                        |
| ------------------------ | --------------- | ------------------------------------------------------------ |
| **WGS84**                | Global          | Used by GPS; the default assumption for most coordinates     |
| **NAD83 / ETRS89 / GDA2020** | Regional-global | Continental datums (N. America, Europe, Australia)       |
| **Everest 1956**         | Local           | Older Indian datum; required for old Indian maps             |

> Moving data between incompatible datums requires a **datum transformation**. QGIS will prompt you to choose one; grid-based transforms (**NTv2**) are more accurate than simple parameter shifts. For high-precision work, "WGS84" alone is imprecise — it has multiple realizations/epochs, and plate motion means the specific datum and date can matter.
{: .prompt-info }

## Coordinate Reference Systems (CRS)

A CRS = a datum + a coordinate grid + units. Two broad kinds.

### Geographic CRS

Uses **latitude/longitude** (angular units, degrees) on the ellipsoid.

- Requires a datum, an origin (equator = 0° latitude; central meridian = 0° longitude), and a unit (degrees).
- Most common: **WGS84 — EPSG:4326**.
- **Good for locating, poor for measuring.** One degree of longitude is a different ground distance at the equator than near the poles, so distances and areas are unreliable in a geographic CRS.

> EPSG:4326 formally orders coordinates **latitude, longitude**, but many tools and files use **longitude, latitude**. Swapped coordinates put your data in the wrong hemisphere. Watch for this when importing CSVs or GeoJSON.
{: .prompt-warning }

> The same point may be written in **decimal degrees** (`40.7128, -74.0060`) or **degrees-minutes-seconds** (`40°42'46"N, 74°00'21"W`). Both describe one location; know which format your source uses before importing.
{: .prompt-info }

### Projected CRS

A flat plane with a uniform **X/Y** grid, an origin, and **linear units** (meters, feet). Required for reliable distance, angle, and area, and for most analysis.

> **EPSG codes** are short numeric identifiers for coordinate systems (e.g. `4326`), letting you select and share a CRS unambiguously. EPSG originated with the European Petroleum Survey Group and is now maintained by **IOGP**. Look codes up at `epsg.io` or `spatialreference.org`.
{: .prompt-info }

## Map Projections

Flattening an ellipsoid onto a plane is always imperfect. Projections are built by projecting onto a **cylinder, cone, or plane** and unwrapping; the family hints at the distortion pattern, but the practical rule is what matters.

### The trade-off — you cannot keep all three

No projection preserves shape, distance, and area at once.

| Preserves                | Type        | Examples                          | Use for                        |
| ------------------------ | ----------- | --------------------------------- | ------------------------------ |
| **Shape / angles**       | Conformal   | Mercator, Lambert Conformal Conic | Navigation, angle-critical work|
| **Distance (from a point)** | Equidistant | Azimuthal Equidistant          | Range/distance from a center   |
| **Area**                 | Equal-area  | Albers Equal Area, Equal Earth    | Comparing sizes, density maps  |

Over a small enough area, all three can be held within tolerance. Choosing the right projection for the task is the analyst's responsibility. **Tissot's indicatrix** — small circles drawn on a map — is the standard way to visualize where a projection distorts.

> Every common web basemap (OSM, Google, Bing) uses **Web Mercator (EPSG:3857)**. It is fine for display and navigation but grossly inflates area toward the poles (Greenland appears near the size of Africa). Do not measure area or run area-based analysis in EPSG:3857 — reproject to an equal-area or local projected CRS first.
{: .prompt-warning }

## Which Projection to Use

| Scale                 | Recommended                                                                 | EPSG guidance     |
| --------------------- | -------------------------------------------------------------------------- | ----------------- |
| **Global map**        | **Equal Earth** (equal-area, adopted by NASA); do not accept the GIS default| e.g. EPSG:8857    |
| **Country / region**  | **Albers Equal Area** (area) or **Lambert Conformal Conic** (shape); prefer a national grid | Varies |
| **City / regional**   | **UTM** (Universal Transverse Mercator)                                     | See below         |

### National grids

| Country   | Grid                                          |
| --------- | --------------------------------------------- |
| USA       | Albers Equal Area Conic                       |
| India     | Lambert Conformal Conic (Survey of India)     |
| UK        | British National Grid (Ordnance Survey)       |
| Australia | Map Grid of Australia (MGA), by zone          |

### UTM in practice

UTM is a set of 60 zones, each 6° of longitude wide, split into North and South (**120 zones total**). Pick the zone your area falls in, then use its projected CRS to minimize local distortion.

- Find your zone from a UTM reference map or `epsg.io`.
- **EPSG convention (WGS84):** North zones = `326` + zone number (Zone 43N → EPSG:32643); South zones = `327` + zone number (Zone 43S → EPSG:32743).
- Example: Bangalore → **UTM Zone 43N → EPSG:32643**.

## CRS in QGIS — the part that trips people up

This section prevents the most common early mistakes.

### Layer CRS vs Project CRS

- **Layer CRS** — the CRS the layer's coordinates are actually stored in.
- **Project CRS** — the CRS the map canvas displays in (shown in the status bar, bottom-right).

### On-the-fly (OTF) reprojection

QGIS automatically reprojects every layer to the Project CRS for display, so layers in different CRS still overlay correctly. This changes only the **display**, not the files.

> Beginners see layers "line up" and wrongly conclude the files share a CRS. They may not. On-the-fly reprojection is a display convenience only.
{: .prompt-warning }

### Assign CRS vs Reproject — different operations

| Operation                    | What it does                                                        | When to use                                  |
| ---------------------------- | ------------------------------------------------------------------ | -------------------------------------------- |
| **Assign / Set CRS**         | *Declares* the CRS the data is already in. Does not move coordinates | The layer's CRS is missing, undefined, or wrong |
| **Reproject** (Warp)         | *Transforms* coordinates into a new CRS, creating new values/a new file | Converting correctly-defined data into another CRS |

> Rule of thumb: if the data plots in the **wrong place**, its CRS is likely mis-declared → **assign** the correct one. If the data plots correctly but you need it in different units/CRS for analysis → **reproject**.
{: .prompt-tip }

### Practical CRS habits

- Set a sensible Project CRS (usually a projected CRS suited to your area) before measuring or analyzing.
- Keep analysis inputs in the same projected CRS.
- Never trust distance/area readings taken in a geographic CRS (degrees).

## A Typical Geoprocessing Workflow

Most analysis follows the same shape. Having the sequence in mind prevents skipped steps (a mismatched CRS or unclean data invalidates everything downstream).

`Acquire → Inspect metadata → Check & standardize CRS → Clean & validate → Reproject if needed → Analyze → Style & visualize → Compose map → Export`

1. **Acquire** — obtain the dataset.
2. **Inspect metadata** — CRS, capture scale, accuracy, field definitions.
3. **Check & standardize CRS** — confirm each layer's CRS is correctly declared; get inputs into a common projected CRS.
4. **Clean & validate** — fix invalid geometries, remove duplicates, check attributes.
5. **Reproject** — only if a step needs a different CRS or units.
6. **Analyze** — run the operations that answer your question.
7. **Style & visualize** — symbolize outputs to read them.
8. **Compose & export** — build a map layout or export data/results.

> The first three steps happen before any analysis. Most "wrong result" problems trace back to skipping them.
{: .prompt-tip }

## Spatial Relationships and Topology

GIS answers questions by testing how geometries relate in space. These tests (**spatial predicates**) drive **Select by Location**, **Join Attributes by Location**, spatial queries, and overlay tools.

| Predicate            | True when                                                        |
| -------------------- | --------------------------------------------------------------- |
| **intersects**       | Geometries share any point (the most general test)              |
| **within / contains**| One geometry lies fully inside the other (the two are inverses) |
| **touches**          | They share only a boundary, not interior                        |
| **overlaps**         | Interiors partly coincide, same dimension                       |
| **crosses**          | Interiors meet at a lower dimension (a line crossing a polygon) |
| **equals**           | Geometries are identical                                        |

### Topology

**Topology** describes how features connect and share space, independent of exact coordinates: shared boundaries between adjacent polygons, lines that connect at nodes, **no gaps** and **no overlaps (slivers)**. Many processing and overlay tools assume valid topology; invalid geometry produces wrong or empty results. Enable **snapping** while digitizing and validate geometry before analysis.

## Core Vector Operations

Definitions you will apply repeatedly (all in the Processing Toolbox):

- **Buffer** — zone of a set distance around features (e.g. 100 m around rivers).
- **Clip** — cut one layer to the boundary of another (cookie-cutter).
- **Intersection** — keep only overlapping areas; combines attributes of both.
- **Union** — keep all areas from both layers, split where they overlap.
- **Dissolve** — merge features sharing an attribute into one (e.g. districts → states).
- **Spatial join** — attach attributes based on spatial relationship (e.g. tag each point with the polygon it falls in).
- **Centroid** — the geometric center point of a polygon.

> All distance/area-based operations assume a projected CRS in meaningful units.
{: .prompt-warning }

## Core Raster Concepts

- **Bands** — layers of values in one raster (single-band DEM vs multi-band imagery like RGB or RGB+NIR).
- **Resolution** — ground size of one cell (smaller = more detail, larger files).
- **NoData** — a reserved value marking "no measurement"; excluded from calculations.
- **Resampling** — changing cell size/alignment (nearest neighbor for categorical data; bilinear/cubic for continuous).
- **Raster calculator** — per-cell math across bands/rasters (e.g. NDVI from red and NIR bands).
- **Reclassify** — group continuous values into classes (e.g. elevation → low/medium/high).
- **Zonal statistics** — summarize raster values within vector polygons (e.g. mean rainfall per district).
- **Terrain products** — slope, aspect, and hillshade derived from a DEM.

## Common Beginner Pitfalls

- **Mixing CRS in analysis** — inputs in different CRS give wrong or empty results. Standardize first.
- **Assign vs reproject confusion** — see the QGIS CRS section.
- **Measuring in a geographic CRS** — degrees are not distances; reproject first.
- **Using Web Mercator for area** — inflates area toward the poles.
- **Editing Shapefile schemas** — 10-character field limit truncates names; prefer GeoPackage.
- **Broken Shapefiles** — moving a `.shp`{: .filepath} without its sidecar files (especially `.prj`{: .filepath}) loses data or CRS.
- **Digitizing without snapping** — creates gaps/overlaps (slivers); enable snapping and check topology.
- **Comparing NULL as if it were a value** — NULL is not `0` or empty text; test with `IS NULL`.
- **Forgetting to toggle editing / save edits** — changes are not written until you save the edit session.

## Where to Get Data

| Source                   | Content                                                  |
| ------------------------ | -------------------------------------------------------- |
| **Natural Earth**        | Free global vector + raster (small scale)                |
| **OpenStreetMap**        | Global roads/buildings/POIs (QuickOSM plugin, Geofabrik) |
| **Copernicus Data Space**| Sentinel satellite imagery                               |
| **USGS EarthExplorer**   | Landsat imagery, elevation                               |
| **OpenTopography**       | Elevation and lidar                                      |
| **National / open-data portals** | Authoritative local data (boundaries, addresses) |

## Glossary

- **Feature** — one vector object with its attributes.
- **Attribute table** — the table of feature properties; one row per feature.
- **Field type** — the data type of a column (integer, decimal, text, boolean, date).
- **NULL** — "no value recorded"; distinct from `0`, empty text, or `false`.
- **Metadata** — data describing a dataset (CRS, source, date, scale, accuracy, licensing).
- **Accuracy** — closeness of a value to the true value.
- **Precision** — repeatability of a measurement, or number of digits reported.
- **Datum** — definition tying an ellipsoid to the real Earth.
- **Ellipsoid** — the smooth mathematical Earth model used for calculation.
- **Geoid** — the true equal-gravity surface; basis for elevation.
- **Geoid separation** — height difference between geoid and ellipsoid at a location.
- **CRS** — coordinate reference system (datum + grid + units).
- **Geographic CRS** — lat/long in degrees (e.g. EPSG:4326).
- **Projected CRS** — flat X/Y in linear units (meters/feet).
- **EPSG code** — numeric identifier for a CRS.
- **Projection** — method of flattening the ellipsoid onto a plane.
- **UTM** — 120 zoned projected systems for local mapping.
- **On-the-fly reprojection** — QGIS reprojecting layers to the project CRS for display only.
- **Spatial predicate** — a relationship test between geometries (intersects, within, etc.).
- **Topology** — rules about how features connect and share space (no gaps/overlaps).
- **Band** — a value layer within a raster.
- **NoData** — reserved "no value" cell marker.
- **Georeferencing** — assigning real-world coordinates to an image.
- **Geocoding** — converting addresses to coordinates.
- **Spatial index** — a lookup structure for fast location-based queries.

## Appendix: EPSG Codes You Will Use Often

| Code            | System                          | Use                              |
| --------------- | ------------------------------- | -------------------------------- |
| **4326**        | WGS84 geographic (lat/long)     | Storage, GPS, data exchange      |
| **3857**        | Web Mercator                    | Web basemaps/display only         |
| **8857**        | Equal Earth                     | Global equal-area maps           |
| **326xx / 327xx** | UTM North / South (WGS84)     | Local mapping (xx = zone)        |
| **27700**       | British National Grid           | UK mapping                       |

## Appendix: Terms You'll Meet Early in QGIS

- **Georeferencing** — assigning real-world coordinates to an image (a scanned map or aerial photo) so it overlays other layers. Tool: **Georeferencer**.
- **Geocoding** — converting addresses or place names into coordinates; reverse geocoding does the opposite. Performed via plugins.
- **Spatial index** — a lookup structure that lets QGIS find features by location without scanning the whole layer, speeding up joins, Select by Location, and rendering. Built with **Create Spatial Index**; GeoPackage and PostGIS can store one.