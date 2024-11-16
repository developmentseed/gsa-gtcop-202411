---
title: "Scaling STAC"
subtitle: "Designing and maintaining STAC infrastructure for large‐scale applications"
author: "Henry Rodman"
date: "November 19, 2024"
format:
  revealjs:
    center: false
    center-title-slide: true
    slide-number: true
    chalkboard: 
      buttons: false
    preview-links: auto
    logo: images/logo_no_text.png
    footer: "[developmentseed.org](https://developmentseed.org)"
    theme: white # try black for dark theme
    css: styles.css    
    navigation-mode: vertical
    controls-layout: bottom-right
    controls-tutorial: true
title-slide-attributes:
    data-background-image: images/devseed_background.png
---

# Introduction

## Development Seed

:::: {.columns}
::: {.column width="50%"}
- geospatial engineering and product design consultancy 
- 20 years of operations with offices in Washington DC, and Lisbon, Portugal. With 54+ staff working across the globe.
:::
::: {.column width="50%"}
<iframe src="https://developmentseed.org/" width="500" height="480" style="" frameBorder="0" allowFullScreen></iframe>
:::
::::

# STAC refresher

![](https://stacspec.org/public/images-original/STAC-01.png){.r-stretch}

## SpatioTemporal Asset Catalog

:::: {.columns}
::: {.column width="60%"}
- STAC spec
- STAC API spec
- community-driven
:::

::: {.column width="40%"}
::: {.fragment}
<iframe src="https://giphy.com/embed/hRsayJrDAx8WY" width="371" height="480" style="" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/funny-food-hRsayJrDAx8WY">via GIPHY</a></p>
:::
:::
::::

::: aside
[September 2024 GtCoP presentation](https://github.com/GSA/gtcop-wiki/wiki/September-2024-STAC-presentations-by-development-community)
:::

## Vision

> The goal is for all providers of spatiotemporal assets (Imagery, SAR, Point Clouds, Data Cubes, Full Motion Video, etc) to expose their data as SpatioTemporal Asset Catalogs (STAC), so that new code doesn't need to be written whenever a new data set or API is released.

- Data providers don't need to write new APIs
- Data consumers don't need to learn new APIs 
- Tool builders don't need to build against new APIs

::: {.notes}

:::

## STAC-powered applications

- STAC Browser
- dynamic raster visualizations

## {background-interactive=true background-iframe="https://radiantearth.github.io/stac-browser/#/?.language=en"}

::: footer
:::

## {background-interactive=true background-iframe="https://raster.eoapi.dev/searches/c13976bc1e5dc7f3c2b4a64239871fc0/map?assets=visual&assets_bidx=visual|1,2,3&minzoom=8&maxzoom=22"}

**Maxar imagery from Maui wildfires in August 2023**

::: footer
<https://raster.eoapi.dev/collections/MAXAR_Maui_Hawaii_fires_Aug_23/map?assets=visual&assets_bidx=visual|1,2,3&minzoom=8&maxzoom=22>
:::

## How is it going?

:::: {.columns}
::: {.column width="70%"}
- Providers: producing STAC metadata and hosting STAC APIs
- Developers: building awesome applications
- Consumers: pushing the limits of APIs and STAC spec
- Community: updating the STAC spec to meet needs of providers, consumers, and developers
:::
::: {.column width="30%"}
![](https://icon-library.com/images/cycle-icon/cycle-icon-29.jpg)
:::
::::

# STAC at scale: a journey

## Challenges

- data volume!
- search speed for applications that consume data
- ingestion pipelines

##

### LandsatLook
- user-facing USGS application for visualizaing mosaics of Landsat scenes

### Microsoft Planetary Computer
- massive geospatial data catalog 
- compute platform right next to the data

### Amazon Sustainability Data Initiative
- massive catalog of public data
- event-driven STAC metadata generation pipelines

# LandsatLook

- an early STAC-powered application
- interactive Landsat imagery browser
- MosaicJSON was born

::: {.notes}
- USGS has a STAC API for serving STAC metadata for Landsat missions
- Born out of an experiment to test mosaic visualizations from STAC metadata
- Sending STAC API requests for every single tile request is not efficient
- Spurred the development of MosaicJSON
:::

## {background-interactive=true background-iframe="https://landsatlook.usgs.gov/explore?date=2024-10-17%7C2024-11-15&sat=LANDSAT_9%7CLANDSAT_8"}

::: footer
<https://landsatlook.usgs.gov/explore>
:::

## MosaicJSON

:::: {.columns}
::: {.column width="50%"}
- pre-compute which STAC items are required for each XYZ tile
  - based on search criteria and item spatial extents
- makes dynamic mosaic visualizations possible!
- not very fast
:::
::: {.column width="50%"}
![](https://user-images.githubusercontent.com/10407788/68706772-7539ac00-055e-11ea-8c15-5ee4f30b143e.jpg)
:::
::::

:::{.notes}
- MosaicJSON files can take a while to generate for a large area
:::

# Microsoft Planetary Computer

##
- STAC API with many public datasets
- Continuous flow of new data
- Interactive visualization platform powered by STAC metadata

## {background-interactive=true background-iframe="https://planetarycomputer.microsoft.com/catalog"}

::: footer
<https://planetarycomputer.microsoft.com/catalog>
:::

:::{.notes}
- Show how you can filter by date range, show different visualizations, etc.
:::

## Microsoft Planetary Computer

::::{.columns}
:::{.column width="50%"}
**How can we optimize for lightning fast queries?**
:::
:::{.column width="50%"}
:::{.fragment}
[pgstac](https://github.com/stac-utils/pgstac) was born
![](https://user-images.githubusercontent.com/10407788/174893876-7a3b5b7a-95a5-48c4-9ff2-cc408f1b6af9.png)
:::
:::
::::

## `pgstac`

> PostgreSQL schema and functions for Spatio-Temporal Asset Catalog (STAC)


::::{.columns}
:::{.column width="50%"}
**strengths**

- Postgres!
- built to handle hundreds of millions of items
- optimized for "needle in haystack" queries
- hashed searches
:::
:::{.column width="50%"}
**weaknesses**

- need to host a Postgres database to use it
- aggregation statistics can be slow to calculate
:::
::::

::: {.notes}
- a lot of pgstac's development was funded by work on MPC
- goal was to put as much of the STAC ops in the database so many clients could talk to it without re-writing the core
- putting things in the postgres db gives you a lot for free
- hashed searches: many queries (e.g. for dynamic tiling) are going to yield the same set of items. pgstac has a mechanism for caching search results to enable faster retrieval of matching items
:::

# Amazon Sustainability Data Initiative (ASDI)

## ASDI
- sustainability data situated next to a compute platform
- continuously updated
- NODD: partnership with NOAA

## {background-interactive=true background-iframe="https://radiantearth.github.io/stac-browser/#/external/dev.asdi-catalog.org/?.language=en"}

::: footer
:::

## stactools-packages

> The stactools-packages github organization is a home for datasets and tool packages using the SpatioTemporal Asset Catalog (STAC) specification.

## {background-interactive=true background-iframe="https://stactools-packages.github.io/"}

::: footer
:::


## stactools-pipelines

![](https://github.com/developmentseed/stactools-pipelines/blob/main/docs/aws_asdi_cog.png?raw=true){.r-stretch}

::: footer
[developmentseed/stactools-pipelines](https://github.com/developmentseed/stactools-pipelines)
:::

::: {.notes}
- event-based processing
- SNS notifications for new uploads to a bucket provide a mechanism for keeping the STAC records closely synced with new data
- As soon as a new asset lands, a pipeline to generate STAC metadata is triggered
:::
