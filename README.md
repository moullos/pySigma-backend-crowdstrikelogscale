# pySigma CrowdStrike LogScale Backend
![Tests](https://github.com/SigmaHQ/pySigma-backend-elasticsearch/actions/workflows/test.yml/badge.svg)
![Coverage
Badge](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/moullos/e3294a47658c1d831f74a0921089ce61/raw/818c4c7d081e9efbbdbddfbf1262e5bd6b9ad20a/moullos-pySigma-backend-crowdstrikelogscale.json)
![Status](https://img.shields.io/badge/Status-pre--release-orange)

This is the CrowdStrike LogScale backend for pySigma. It provides the package `sigma.backends.logscale` with the `LogScaleBackend` class.

Further, it contains the `falcon_pipeline` processing pipeline in `sigma.pipelines.falcon` which adds field and value mappings for logs collected via the CrowdStrike Falcon Agent.

Only a single output format is supported which is the default output format for LogScale queries.

This backend is currently maintained by:

* [Panos Moullotou](https://github.com/moullos/)

## Supported Rules
The following categories and products are supported:
| category | product |
|-|-|
|`process_creation` | `windows`, `linux`|
|`network_connection` | `windows`|
|`dns_query` | `windows`|
|`image_load` | `windows`|
|`driver_load` | `windows`|
|`ps_script` | `windows`|

There's likely more windows categories that can be supported by the falcon pipeline; I will be adding support gradually as availability allows. 

## Backend
The backend transforms rules into [LogScale](https://library.humio.com/data-analysis/syntax.html) queries. Due to the LogScale being [case-sensitive](https://library.humio.com/kb/kb-case-insensitive-searches.html?redirected=true), the backend heavily utilizes [regex delimeters](https://library.humio.com/kb/kb-case-insensitive-searches.html?redirected=true#kb-case-insensitive-searches-work-around). 

## Pipeline
The only currently supported pipeline is `falcon_pipepline` which adds fields and value mappings for events collected through the CrowdStrike agent. 

### Limitations and caveats:
- **Full Paths**: 
Falcon agents do not capture drive names when logging paths. Instead, when drive letters are expected the device path is used. For example, `C:\Windows` results to `\Device\HarddiskVolume3\Windows` in the logs. To account for this, the pipeline replaces any drive letters in fields containing full path with `\Device\HarddiskVolume?\`  (where '?' can be any single character).

- **Parent Name**:
Falcon `process_creation` events do not capture the full path of the parent. Hence, in such cases the transformation is configured to fail.

- **DNS Query Results**:
Falcon `dns_query` events return the IP records of a successful query in [semicolon-seperated](https://github.com/CrowdStrike/logscale-community-content/blob/main/CrowdStrike-Query-Language-Map/CrowdStrike-Query-Language/concatArray.md) string. The pipeline handles this by enforcing a "contains" expression on the `QueryResults` field
- **Unsupported fields**:
Falcon does not always capture the same fields as sysmon for the categories supported. In cases where the rule requires unsupported fields, the transformation fails.

## References
- [LogScale Community Content](https://github.com/CrowdStrike/logscale-community-content)