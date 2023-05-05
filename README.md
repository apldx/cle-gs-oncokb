OncoKB for GatewaySeq
=====================


Resources
---------

https://github.com/dhslab/cle-gatewayseq

https://www.oncokb.org/swagger-ui/index.html


Configuration
-------------

See template in 

    config/gatewayseq_config.template.json

Add the OncoKB API key here

```json
"oncokb_api_key": "ADD API KEY HERE",
```

This file also configures the tumor types to download from OncoKB 
and the API timeout in seconds


Usage
-----

Read in `CONFIG` and GatewaySeq `JSON` file and add OncoKB annotation under
`.REPORTING.oncokb`, with JSON output to `STDOUT`

```bash
# Production use
py/oncokb_annotate_json.py CONFIG JSON

# Include variant information in each oncokb entry for debugging
py/oncokb_annotate_json.py --include-variant CONFIG JSON
```

Output
------

The script adds a section to the top-level `REPORTING` object, 
creating it if needed

```javascript
"REPORTING": {
  "oncokb": { ... }
}
```

There are two sections under `oncokb`

```javascript
"oncokb": {
  "PASS": [ ... ],
  "Filtered": [ ... ]
}
```

corresponding to the top-level `VARIANTS` section in the input JSON file

```javascript
"VARIANTS": {
  "PASS": [ ... ],
  "Filtered": [ ... ]
}
```

Elements in the `oncokb` section arrays correspond to elements in the
`VARIANTS` arrays, with one object per `VARIANTS` element as follows


```javascript
{
  // The first API call made is a preflight check with no tumor type 
  // specified to see if OncoKB has any data for the variant
  
  // Only if --include-variant was used
  "variant": [ ...variant information... ],

  "apiStatus": "low_vaf|api_failed|not_found|ok"
```

```javascript
  "apiStatus": "low_vaf"
  // The variant is from the Filtered section
  // with VAF below specified minimum (default 1.0%) and is not queried
  // further
```

```javascript
  "apiStatus": "api_failed"
  // Additional debugging data will
  // be stored  under "apiRequests", which can be either an exception
  // or HTTP status information

  // An exception occurs if the network is down or the API is
  // otherwise unreachable or unresponsive
  "apiRequests": { "exception": "exception string" }

  // OR if the server responded, but the status returned was not ok, 
  // eg the API page was not found, a bad request was made, 
  // the call timed out (timeout is set to 60s),
  // or an internal server error occurred
  "apiRequests": { 
    "status_code": "HTTP status code", 
    "reason": "HTTP status reason"
  }
```

```javascript
  "apiStatus": "not_found"
  // The API call succeeded but OncoKB has
  // no data for the variant, and no further queries for the variant 
  // are made
```

```javascript
  "apiStatus": "ok"
  // The variant has data in OncoKB, and a new API
  // call is made for each tumor type, with data stored under a key
  // named with that tumor type containing an object
  // The object contains "apiStatus"/"apiRequests" keys
  // as in the preflight check (in case the preflight is "ok" but a 
  // tumor type call is "not_found" or "api_failed", which should not 
  // normally happen) as well as the OncoKB data (if tumor type "apiStatus" 
  // is "ok"), eg
  "Thyroid": {
    "apiStatus": "ok",
    // ...OncoKB data...
  }
}      
```


Docker
------

Code to build a Docker image based on 

    registry.gsc.wustl.edu/mgi-cle/myeloseqhd:v2

is under `docker/`. The `Dockerfile` is dead simple, just installing 
the Python `requests` package

```bash
cd docker

# Pull registry.gsc.wustl.edu/mgi-cle/myeloseqhd:v2
# I got errors trying to do a FROM directly, but 
# it works if it is present as a locally pulled Docker image
make pull

# Build the Docker image
make build

# Launch an interactive session for testing, with ../ being
# mounted as /host
make interact
```

