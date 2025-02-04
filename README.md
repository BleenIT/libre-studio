This project consists on a management source for [Maplibre Martin](https://martin.maplibre.org/), allowing the management of map sources, sprites and font glyphs, all stored in the filesystem and on a Postgresql database.

The frontend is built using ReactJS, Vite, and Bootstrap.

The backend is a Spring Boot application.

![Styles example](https://www.bleen.pt/images/mapbuilder/mapbuilder-styles.gif)

## Features

- Visual frontend for managing [Maplibre Martin](https://martin.maplibre.org/) sources 
- Real-time styles editor, with error validation, feature inspection, and raw JSON editor
- Support importing sources in GeoJSON, Geopackage, KML and SHP
- Provides a ready-to-use, custom maps infrastructure for any project
- Includes 3 sample styles:
  - Bleen ([Qwant](https://github.com/Qwant/qwant-basic-gl-style) derived style)
  - Bleen - Dark
  - [MapTiler Basic](https://github.com/openmaptiles/maptiler-basic-gl-style)

### Prerequisites

- Git
- Docker

## Installation

- Clone this repository
  - `git clone https://github.com/BleenIT/libre-studio .`

- Include 1+ pmtile files under the `pmtiles` folder on the shared volume (defaults to `./mapbuilder-vol/pmtiles/`).
You see [below](#creating-vector-tiles-from-osm-pbfs) how to create pmtiles files from [OSM Geofabrik extracts](https://download.geofabrik.de/).

- Finally:
  - `docker compose up` 

After providing a base map tiles (using the instructions below) running the docker, the Web UI is available at [http://localhost:8081](http://localhost:8081)  

### Notes

- We are not including any example sources on the docker image, as they can be easily loaded through the web interface.

- The exposed ports are (can be changed in `docker-compose.yml`):
  - Postgresql: 54320
  - Martin: 30000
  - Backend: 8181
  - Frontend: 8081

- By default, the backend is configured to run on `localhost`, port `8181` and Martin on `30000`. This can be change by setting two environment variables:
  - BACKEND_URL - the Backend API URL (defaults to http://localhost:8181/api)
      - `export BACKEND_URL=http://localhost:8181/api`
  - `MARTIN_URL` - Martin URL (defaults http://localhost:30000)
      - - `export MARTIN_URL=http://localhost:30000`

- On `docker-compose.yml` file, you can configure the shared docker volume, which defaults to `./mapbuilder-vol/`.

- The frontend is not production ready, currently used for testing purposes.

- The backend has no authentication, only expose the public styles endpoint (`/styles/{id}/raw`), and Martin ones. 

## Creating vector tiles from OSM PBFs

To generate the vector tiles (pmtile file) from the Open Street Maps PBF files, you will need 
[Docker](https://www.docker.com/), 
[Tilemaker](https://tilemaker.org/), 
and a PBF file for the region:

1. Download the map region you need from [OSM Geofabrik extracts](https://download.geofabrik.de/).
    - `Note: start with a small region, as this process can take a lot of time (hours) with bigger regions.`
2. Alternatively, you can use our provided scripts that perform some optimizations on the generated pmtiles file. For that you will need to download the files:
     - [Urban areas](https://naciscdn.org/naturalearth/10m/cultural/ne_10m_urban_areas.zip)
     - [OSM Water polygons](https://osmdata.openstreetmap.de/download/water-polygons-split-4326.zip)
3. Put them in the `./data` folder and unzip them:
     - `mkdir -p ./data/landcover`
     - `unzip ./data/ne_10m_urban_areas.zip -d ./data/landcover/ne_10m_urban_areas`
     - `unzip ./data/water-polygons-split-4326.zip -d ./data/; mv ./data/water-polygons-split-4326 ./data/coastline`
4. Run tilemaker (replace `REGION` by the name of the file you downloaded):
   ```
   docker run -it --rm --pull always -v $(pwd)/data:/data -w /data \
      ghcr.io/systemed/tilemaker:master \
      --input /data/REGION.osm.pbf \
      --output /data/openmaptiles.pmtiles \
      --store /data/store \
      --config /data/config-openmaptiles.json \
      --process /data/process-openmaptiles.lua
   ```
    - Note: this will generate an `openmaptiles.pmtiles` file. Keep the filename as is, so that the provided styles work out-of-the-box.
5. You can now use the generated `openmaptiles.pmtiles` and put it under `./mapbuilder-vol/pmtiles/` folder.

This project uses [tilemaker](https://github.com/systemed/tilemaker) to generate the pmtiles, you can check their github for further instructions and customizations.

## TODO

- Need to create different postgres table sources on martin config, similar to how the sprites are being done;
- Support feature properties on sources (WIP);
- Support duplicate fonts on Martin (if Martin detects a duplicate font, it will shutdown)
- Map sources visual editor
- Pagination on frontend
- Authentication