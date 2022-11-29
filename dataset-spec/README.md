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

A dataset specification refers to GeoJON *feature*. The data cube is fully
specified by the details given in the feature's *properties*. 

In addition, the feature's *geometry* provides the geographic boundaries
or extent of the data cube. According to GeoJSON, the geometry must be 
given in geographic WGS-84 longitude-latitude coordinates
using decimal degree units.

## Cube Generation Recipe

Every data cube generation recipe in the repo `cube-gen` must contain 
exactly one cube specification called `cube-gen/${recipe}/cube.geojson`.

## Property definitions

_Mandatory_ properties are properties that are required inputs to the
data cube generation process and for the dataset's identification.

_Optional_ properties are metadata properties that are not interpreted. 
However, if provided, they should be provided in exactly the form described here.

Additional metadata can be provided using the property 
`metadata`. 

### Dataset properties

* `data_id: string`  -  unique identification string, filename base
  **mandatory**

* `title: string` **mandatory**
* `description: string` **mandatory**
* `license: string` - terms and conditions of the DeepESDL data distribution
   **mandatory**
* `recipe: string`: URL to repo recipe directory
  **mandatory**
* `changes: string`: URL to `CHANGES.md` in recipe directory
  **mandatory**

#### Temporal coverage and resolution

* `time_range: [start, end]` - time coverage of the dataset
  **mandatory**
* `time_period: string | null` - time period of the dataset, e.g. `"8D"`.
  If not provided, the frequency of time steps may be varying.
  **optional**

#### Spatial coverage and resolution

* `spatial_ref`: Spatial coordinate reference system (CRS) name, 
  e.g., `"EPSG:4326"`, `"CRS84"`, **mandatory**

The spatial extent can be given either by 

* `spatial_bbox: [xmin:float, ymin:float, xmax:float, ymax:float]`:
  bounding box in units of the CRS,
  e.g., `[-180, -90, 180, 90]`, **mandatory**
* `spatial_res: res | [xres, yres]`: spatial resolution in units of the CRS,
  e.g., `[0.05, 0.025]`, **mandatory**

or by
        
* `spatial_size: [width:int, height:int]` - the image width and height,
  e.g., `[6000, 3000]`,  **mandatory**
* `spatial_offset: [xpos, ypos]`: the offset coordinates in units of the 
  CRS. These refer to bottom-left corner of the grid cell at position
  (0, 0), **mandatory**


#### Dataset variables
  
* `variables: Variable[]` - array of variable definitions (see below) 
  **mandatory**

#### Dataset sources

* `sources: Source[]` - array of dataset sources

A dataset source `Source` can reference a data store or describe 
the means to refer to a set of files accessible by some URL or set of URLs.

* `variable_names: string[]`: The list of variables provided by theis source,
  **mandatory**
 
* `store_id: string`: The list of variables provided by theis source,
  **mandatory**
* `store_params: string`: Data store parameters.
  **optional**
* `data_open_params: {var_name: open_params}`: Data source open parameters.
  **optional**

  

metadata to be provided in cube.geojson (TODO: align with CF):
   mandatory
            

   optional 
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
            "source": "EO4SIBS, CMEMS"

metadata to be generated:

            "date_modified": "2022-08-19 16:19:15.359970",
            "geospatial_lat_max": 47.9985,
            "geospatial_lat_min": 39.001500000000156,
            "geospatial_lat_resolution": 0.0030000000000001137,
            "geospatial_lon_max": 41.997499999999725,
            "geospatial_lon_min": 26.0015,
            "geospatial_lon_resolution": 0.0030000000000001137,

Variables definition

mandatory properties that we ingest process

            "name": "CHL",
            "long_name": "Chlorophyll Concentration",
            "dtype": "float32",
            "dims": ["time", "lat", "lon"],


        mandatory for numerical, floating point data, use
            "units": "mg/m^3",

            "valid_min": 0.0,
            "valid_max": 1000.0,
        or
            "fill_value"

            "sources":  {           # why a list?
                    "home_url": "https://resources.marine.copernicus.eu/",
                    "data_url": "https://resources.marine.copernicus.eu/",
                    "license_url": "https://marine.copernicus.eu/user-corner/service-commitments-and-licence",
                    "attributions": [],
                    "processing_steps": [
                        "Masking out low quality pixels",
                        "Temporal NaN-mean aggregation",
                        "Spatial bi-cubic interpolation"
                    ],
                    "remarks": ""
                }
            

optional:    
            "time_range": ["2005-05-01", "2022-05-01"],     # defaults to cube "time_range"

            
mandatory metadata            
            "color_bar_name": "YlGn",
            "color_value_min": 
            "color_value_max": 
            
            "color_value_mapping": {
                 0: "red",
                 1: "yellow",
                 ...
            }


optional metadata
                "keywords": ["colour", "marine", "chlorophyll"],
                "processing-level": "L4"
