---

title: Web GIS Developer Learning Plan
author: hcoco1
date: 2026-06-10 16:10:00 +0200
categories: [Career, GIS]
tags: [web-gis, react, fastapi, postgis, python, ai, learning-plan]
render_with_liquid: false
pin: false
description: A practical learning plan to become a Web GIS Developer with React, FastAPI, PostGIS, Python GIS, and applied AI.
---

## Web GIS Developer Learning Plan

The target role is:

> **Web GIS Developer / Geospatial Software Developer**

The goal is not to become a generic GIS analyst, generic web developer, or AI engineer. The goal is to build practical software that handles geospatial data, exposes it through APIs, and displays it in modern web applications.

---

## 1. Define the Career Direction

The main career direction is:

```text
Web GIS Developer
Geospatial Software Developer
GIS Developer
PostGIS Developer
Geospatial Backend Developer
```

This direction combines:

* Web development
* Python backend development
* Spatial databases
* Geospatial data processing
* Interactive maps
* Practical AI integration

---

## 2. React Frontend

Main source:

```text
John Smilga React course on Udemy
```

Focus areas:

* React fundamentals
* Components
* Props and state
* Hooks
* React Router
* Forms
* API calls
* TanStack Query
* TypeScript
* Project structure

Purpose:

> Build the frontend layer of GIS applications.

The goal is not to finish every single lesson blindly. The goal is to become comfortable building real frontend interfaces that communicate with an API.

---

## 3. Python Backend

Main focus:

```text
FastAPI
PostgreSQL
SQLAlchemy
Alembic
Pydantic
JWT authentication
Pytest
Docker
Deployment
```

Purpose:

> Build real APIs that can serve data to a frontend application.

Important backend skills:

| Skill          | Reason                                     |
| -------------- | ------------------------------------------ |
| FastAPI        | Modern Python API framework                |
| SQLAlchemy     | Work with relational databases from Python |
| Alembic        | Manage database migrations                 |
| PostgreSQL     | Main production database                   |
| Authentication | Protect users and private data             |
| Testing        | Prove the API works                        |
| Docker         | Run the app consistently                   |
| Deployment     | Publish the API online                     |

---

## 4. Geospatial Python

Main sources:

```text
Geo-Python
AutoGIS
```

Focus areas:

* Python data handling
* GeoPandas
* Shapely
* Coordinate reference systems
* Spatial joins
* Overlays
* Geocoding
* OpenStreetMap data
* Raster basics
* Reproducible notebooks
* Git and GitHub workflow

Purpose:

> Learn how to clean, process, analyze, and prepare geospatial data before storing it in a database.

This is the data-processing layer of the career path.

---

## 5. PostGIS

Main focus:

```text
PostgreSQL + PostGIS
```

Skills to learn:

* Geometry columns
* Geography columns
* Spatial indexes
* `ST_Intersects`
* `ST_Within`
* `ST_DWithin`
* Bounding box queries
* GeoJSON output
* Spatial joins
* Query performance
* Importing spatial data

Purpose:

> Store and query geospatial data correctly.

PostGIS is one of the most important skills in this plan because it connects GIS knowledge with backend development.

---

## 6. Web Mapping

Start simple, then move to modern tools.

Recommended order:

```text
Leaflet
MapLibre
Vector tiles
PMTiles
deck.gl later
```

Purpose:

> Display spatial data in the browser.

Key skills:

| Tool         | Use                                 |
| ------------ | ----------------------------------- |
| Leaflet      | Simple interactive maps             |
| MapLibre     | Modern WebGL maps                   |
| Vector tiles | Efficient large-scale map rendering |
| PMTiles      | Static map tile delivery            |
| deck.gl      | Advanced visualization later        |

---

## 7. Full-Stack GIS Project

This is the main portfolio project.

Suggested stack:

```text
React + TypeScript
FastAPI
PostgreSQL/PostGIS
MapLibre or Leaflet
Docker
Render/Railway/Vercel
```

The project should include:

* User interface
* Interactive map
* Backend API
* PostGIS database
* Spatial search
* Filters
* Authentication
* Data import
* Tests
* Deployment
* Professional README

Purpose:

> Prove that the learning path produces real software, not only course certificates.

---

## 8. Practical AI Layer

AI should support the career path, not distract from it.

Start with:

* AI-assisted coding
* Code review
* Test generation
* Debugging help
* Documentation
* Resume and job-search support

Later, add AI features to the GIS project:

| AI feature              | Example                                  |
| ----------------------- | ---------------------------------------- |
| Natural-language search | “Show jobs near Delft within 10 km”      |
| Data explanation        | Summarize visible map features           |
| Report generation       | Create a short report from selected data |
| Data cleaning assistant | Detect missing CRS or invalid geometry   |
| Job matching            | Compare job ads with personal skills     |

Purpose:

> Use AI as a practical tool inside the Web GIS workflow.

---

## 9. Portfolio and GitHub

Every serious project should have:

* Clean repository
* Clear README
* Screenshots
* Live demo link
* Setup instructions
* Tech stack explanation
* API documentation
* Short project summary
* Known limitations
* Future improvements

Purpose:

> Make the work understandable to recruiters, developers, and GIS professionals.

---

## 10. Job Search Direction

Target job titles:

```text
Web GIS Developer
GIS Developer
Geospatial Developer
Geospatial Software Developer
PostGIS Developer
Geospatial Backend Developer
Junior Backend Developer with GIS
Geospatial Data Engineer
```

Avoid applying only to:

```text
GIS Technician
GIS Analyst
Mapping Assistant
Cartography Assistant
```

Those roles may be useful, but they are not the main career target.

---

## Basic Order

```text
1. React frontend
2. Python backend
3. Geospatial Python
4. PostGIS
5. Web mapping
6. Full-stack GIS project
7. AI layer
8. Portfolio and job applications
```

---

## Core Principle

Every course, tutorial, and project should support this pipeline:

```text
Raw geospatial data
→ Python processing
→ PostGIS database
→ FastAPI backend
→ React frontend
→ interactive web map
→ deployed portfolio app
→ optional AI feature
```

If a topic does not support this pipeline, it should be delayed or skipped.
