# DeepESDL Dataset Specification

A DeepESDL dataset is fully described by a well-defined document
we refer to as _DeepESDL dataset specification_.

A DeepESDL dataset specification defines for each data cube to be generated

* name and description;
* spatio-temporal dimensions;
* spatio-temporal resolution;
* spatio-temporal coverage;
* its data variables;
* arbitrary metadata.
 
And for each of its data variables

* name and description;
* units;
* visualisation info;
* list the data sources;
* list the processing steps;
* arbitrary metadata.

## GeoJSON

Dataset specifications utilize the GeoJSON format with a specific
signature.

A machine-readable form of this signature is given in the JSON schema in 
[template.schema.json](./template.schema.json). 
An example instance that complies to the schema is given in 
[template.json](./template.geojson).

Note that GeoJSON signature can also be provided using the YAML format.

A dataset specification refers to GeoJON *feature*. 

## GeoJSON geometry

The feature's *geometry* provides the geographic boundaries
or extent of the data cube. According to GeoJSON, the geometry must be 
given in geographic WGS-84 longitude-latitude coordinates
using decimal degree units. The geometry is used for visualisation
only. The actual spatial CRS, coverage, and resolution is described 
by dedicated properties.

### GeoJSON properties

The feature's *properties* refer to the data cube's characteristics.
Pure metadata properties are provided in a dedicated property `metadata`,
while other properties are required inputs to the
data cube generation process and resource identification.


#### Dataset - General

| Property    | Type     | Req.? | Description                                            | Example   |
|-------------|----------|:-----:|--------------------------------------------------------|-----------|
| data_id     | string   |   Y   | Unique identification string, filename base            | "TODO"    |
| title       | string   |   Y   | Human-readable title                                   | "TODO"    |
| description | string   |   Y   | Human-readable description                             | "TODO"    |
| version     | string   |   Y   | Version string. Semver format recommended.             | "1.0.3"   |
| recipe      | string   |   N   | URL to repo recipe directory                           | "TODO"    |
| changes     | string   |   N   | URL to `CHANGES.md` in recipe directory                | "TODO"    |
| license     | string   |   Y   | Terms and conditions of the DeepESDL data distribution | "TODO"    |
| variables   | object[] |   Y   | Array of data variable definitions.                    | See below |
| sources     | object[] |   Y   | Array of data source definitions.                      | See below |
| metadata    | object   |   N   | Arbitrary metadata.                                    | See below |

Although `recipe` and `changes` are optional, they should be provided after 
the dataset has been published.


#### Dataset - Temporal coverage and resolution

| Property      | Type               | Req.? | Description                   | Example                      |
|---------------|--------------------|:-----:|-------------------------------|------------------------------|
| time_range    | [string, string]   |   Y   | Time coverage of the dataset  | ["2020-01-01", "2022-12-31"] |
| time_period   | string &#124; null |   N   | Time period of the dataset.   | "8D"                         |

Optional `time_period` may be missing for datasets that comprise timestamps
at varying intervals.


#### Dataset - Spatial coverage and resolution

The spatial extent can be given either by 

| Property        | Type                         | Req.? | Description                                                        | Example                  |
|-----------------|------------------------------|:-----:|--------------------------------------------------------------------|--------------------------|
| spatial_ref     | string                       |   Y   | Spatial coordinate reference system (CRS) name                     | `"EPSG:4326"`            |
| spatial_bbox    | [float, float, float, float] |   N   | Bounding box given as [xmin, ymin, xmax, ymax] in units of the CRS | `[-180, -90, 180, 90]`   |
| spatial_size    | [int, int]                   |   N   | The spatial image size as [width, height] in pixels.               | `[6000, 3000]`           |
| spatial_offset  | [float, float]               |   N   | Offset coordinates in units of the CRS.                            | `[-180, -90]`            |
| spatial_res     | float &#124; [float, float]  |   N   | Spatial resolution in x- and y-direction in units of the CRS.      | `0.025`, `[0.05, 0.025]` |

The `spatial_bbox` coordinates refer to bottom-left and top-right grid cell
corners at grid positions (0, 0) and (width - 1, height - 1).

The `spatial_offset` coordinates refer to bottom-left grid cell corner of the 
grid at position (0, 0).

Although individual properties except `spatial_ref` are optional,
one of the following combinations _must_ be given:

* `spatial_bbox` and `spatial_size`: this is the recommended combination.
* `spatial_bbox` and `spatial_res`: not recommended, 
  because `spatial_size` will be computed and may be unexpected.
* `spatial_offset`, `spatial_size`, and `spatial_res`: not recommended,
  because `spatial_bbox` will be computed and may be unexpected.

  
#### Dataset - Metadata

Example for the `metadata` property that comprises solely uninterpreted 
content:

```json
  "metadata": {
    "Conventions": "CF-1.9",
    "acknowledgment": "EO4SIBS, CMEMS, DeepESDL project",
    "contributor_name": "Brockmann Geomatics Sweden AB",
    "contributor_url": "www.brockmann-geomatics.se",
    "creator_email": "info@brockmann-consult.de",
    "creator_name": "Brockmann Consult GmbH",
    "creator_url": "www.brockmann-consult.de",
    "institution": "Brockmann Consult GmbH",
    "project": "DeepESDL",
    "publisher_email": "info@brockmann-consult.de",
    "publisher_name": "Brockmann Consult GmbH",
    "license_url": "https:// ...",
    "source": "EO4SIBS, CMEMS",
    ...
  }
```

The following metadata fields _should_ be derived from the generated
data cube:

```json
  "metadata": {
    ...,        
    "date_modified": "2022-08-19 16:19:15.359970",
    "geospatial_lon_min": 26.0,
    "geospatial_lon_max": 42.0,
    "geospatial_lat_min": 39.0,
    "geospatial_lat_max": 48.0,
    "geospatial_lon_resolution": 0.003,
    "geospatial_lat_resolution": 0.003,
    "temporal_coverage_start": "2020-01-01 00:00:00",
    "temporal_coverage_end": "2022-12-31 23:59:59",
    "temporal_resolution": "8D",
  }
```

#### Data Variable - General

| Property   | Type               | Req.? | Description                                             | Example                        |
|------------|--------------------|:-----:|---------------------------------------------------------|--------------------------------|
| name       | string             |   Y   | Array of variable definitions.                          | `"CHL"`                        |
| long_name  | string             |   Y   | Array of variable definitions.                          | `"Chlorophyll Concentration"`  |
| dtype      | string             |   Y   | Numpy-compatible data type name.                        | `float32"`                     |
| dims       | string[]           |   Y   | Array of dimension names.                               | `["time", "lat", "lon"]"`      |
| units      | string &#124; null |   Y   | Physical unit.                                          | `"mg/m^3"`, `"n.a."`           |
| fill_value | string &#124; null |   Y   | Unscaled values equal to `fill_value` are undefined.    | `1.0`                          |
| valid_min  | string             |   N   | Values below that number are undefined.                 | `0.0`                          |
| valid_max  | string             |   N   | Values above that number are undefined.                 | `1.0`                          |
| time_range | [string, string]   |   N   | Provided time range. Values out of range are undefined. | `["2005-05-01", "2022-05-01"]` |
| metadata   | object             |   N   | Arbitrary metadata.                                     | See below                      |


`units` and `fill_value` mist be given for floating point data, otherwise
they should be set to `null`.

`time_range` defaults to the dataset's `time_range`.

#### Data Variable - Metadata

The following variable `metadata` properties are **mandatory**, because
we want a defined way to display image tiles: 

```json
  "metadata": {
    ...,          
    "color_bar_name": "YlGn",
    "color_value_min": 
    "color_value_max": 
    
    "color_value_mapping": {
         0: "red",
         1: "yellow",
         ...
    }
```

The following variable `metadata` example comprises 
typical optional properties: 

```json
  "metadata": {
    ...,
    "keywords": ["colour", "marine", "chlorophyll"],
    "processing-level": "L4"
    }
```    

#### Data Source

A data source can be either given as a reference to a xcube data store,
or a set of properties that allows manually accessing individual
data files in a described source, usually by some URL or set of URLs.

The following properties describe access to a xcube data store:

| Property             | Type   | Req.? | Description                                          | Example                |
|----------------------|--------|:-----:|------------------------------------------------------|------------------------|
| name                 | string |   Y   | Human-readable title of this data source.            | `"Sentinel Hub S2L2A"` |
| variable_names       | string |   Y   | Array of variable names provided by this data store. | `["B06", "SCL"]`       |
| store_id             | string |   Y   | xcube data store identifier.                         | `"sentinelhub"`        |
| store_params         | object |   N   | xcube data store parameters.                         | `{"num_retries": 80}`  |
| variable_open_params | object |   N   | xcube data store open parameters.                    |                        |

`variable_open_params` is an object that maps variable names to specific
open parameters passed to the given data store.

The following properties describe access to an arbitrary data source:

| Property            | Type    | Req.? | Description                                                                      | Example                          |
|---------------------|---------|:-----:|----------------------------------------------------------------------------------|----------------------------------|
| name                | string  |   Y   | Human-readable title of this data source.                                        |                                  |
| variable_names      | string  |   Y   | Array of variable names provided by this data store.                             |                                  |
| download_url        | string  |   Y   | Download URL template.                                                           |                                  |
| protocol            | string  |   N   | Transport protocol. Detected from `download_url`, if omitted.                    | `"http"`, `"ftp"`                |
| compressed          | boolean |   N   | Whether data is compressed. Defaults to `false`. Detected from data, if omitted. |                                  |
| compression_format  | string  |   N   | Compression format in use. . Detected from data, if omitted.                     |                                  |
| source_format       | string  |   N   | Source format name. Detected from filename, if omitted.                          | `"netcdf"`, `"hdf"`, `"geotiff"` |
| source_crs          | string  |   N   | Coordinate reference system of source data. Detected from data, if omitted.      |                                  |


## Cube Generation Recipe

Every data cube generation recipe in the repo `cube-gen` must contain 
exactly one cube specification called `cube-gen/${recipe}/cube.geojson`.

