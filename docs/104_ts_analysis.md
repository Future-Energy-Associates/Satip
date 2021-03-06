# Extracting Time-Series for Analysis



```python
from satip import io

import shapely
import geopandas as gpd

import seaborn as sns
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
```

    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\google\auth\_default.py:69: UserWarning: Your application has authenticated using end user credentials from Google Cloud SDK without a quota project. You might receive a "quota exceeded" or "API not enabled" error. We recommend you rerun `gcloud auth application-default login` and make sure a quota project is added. Or you can use service accounts instead. For more information about service accounts, see https://cloud.google.com/docs/authentication/
      warnings.warn(_CLOUD_SDK_CREDENTIALS_WARNING)
    Downloading: 100%|â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆ| 1/1 [00:00<00:00,  3.68rows/s]
    

<br>

### User Inputs

We have to specify the bucket where the data is located

```python
zarr_bucket = 'solar-pv-nowcasting-data/satellite/EUMETSAT/SEVIRI_RSS/zarr_full_extent_TM_int16'
```

<br>

### Loading Data

Then the `satip` wrapper for loading data will generate an `xarray` `Dataset`

```python
ds = io.load_from_zarr_bucket(zarr_bucket)

ds
```




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: 'â–º';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: 'â–¼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
Dimensions:                (time: 11662, variable: 12, x: 1870, y: 1831)
Coordinates:
  * time                   (time) datetime64[ns] 2021-03-19T13:20:10 ... 2020...
  * variable               (variable) object &#x27;HRV&#x27; &#x27;IR_016&#x27; ... &#x27;WV_073&#x27;
  * x                      (x) float64 -3.088e+06 -3.084e+06 ... 4.388e+06
  * y                      (y) float64 9.012e+06 9.008e+06 ... 1.692e+06
Data variables:
    stacked_eumetsat_data  (time, x, y, variable) int16 dask.array&lt;chunksize=(36, 1870, 1831, 1), meta=np.ndarray&gt;</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-efd7f4d8-a45e-43ae-9dea-eafb970e8fce' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-efd7f4d8-a45e-43ae-9dea-eafb970e8fce' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 11662</li><li><span class='xr-has-index'>variable</span>: 12</li><li><span class='xr-has-index'>x</span>: 1870</li><li><span class='xr-has-index'>y</span>: 1831</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-410475f6-2243-4ac5-9f1d-6eaeea73b36c' class='xr-section-summary-in' type='checkbox'  checked><label for='section-410475f6-2243-4ac5-9f1d-6eaeea73b36c' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2021-03-19T13:20:10 ... 2020-04-...</div><input id='attrs-37be0dac-ffb7-4923-a74e-762288992f12' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-37be0dac-ffb7-4923-a74e-762288992f12' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-724a9d5a-b476-4695-aa3c-fd68b63ef48c' class='xr-var-data-in' type='checkbox'><label for='data-724a9d5a-b476-4695-aa3c-fd68b63ef48c' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;2021-03-19T13:20:10.000000000&#x27;, &#x27;2021-03-19T13:24:16.000000000&#x27;,
       &#x27;2021-03-19T13:29:17.000000000&#x27;, ..., &#x27;2020-04-01T00:39:20.000000000&#x27;,
       &#x27;2020-04-01T00:44:20.000000000&#x27;, &#x27;2020-04-01T00:49:20.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>variable</span></div><div class='xr-var-dims'>(variable)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>&#x27;HRV&#x27; &#x27;IR_016&#x27; ... &#x27;WV_073&#x27;</div><input id='attrs-c4633a97-418f-4593-9dcb-6381ab181b8d' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-c4633a97-418f-4593-9dcb-6381ab181b8d' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b727afa5-6a3c-4da2-8f41-24e03b5d3293' class='xr-var-data-in' type='checkbox'><label for='data-b727afa5-6a3c-4da2-8f41-24e03b5d3293' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;HRV&#x27;, &#x27;IR_016&#x27;, &#x27;IR_039&#x27;, &#x27;IR_087&#x27;, &#x27;IR_097&#x27;, &#x27;IR_108&#x27;, &#x27;IR_120&#x27;,
       &#x27;IR_134&#x27;, &#x27;VIS006&#x27;, &#x27;VIS008&#x27;, &#x27;WV_062&#x27;, &#x27;WV_073&#x27;], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>-3.088e+06 -3.084e+06 ... 4.388e+06</div><input id='attrs-b0692e09-2525-4dcb-98b0-11b2271f5d6a' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-b0692e09-2525-4dcb-98b0-11b2271f5d6a' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-1bcbf0d7-8972-43a4-8911-350ce21af140' class='xr-var-data-in' type='checkbox'><label for='data-1bcbf0d7-8972-43a4-8911-350ce21af140' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([-3088000., -3084000., -3080000., ...,  4380000.,  4384000.,  4388000.])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>9.012e+06 9.008e+06 ... 1.692e+06</div><input id='attrs-0444bc62-cc94-4ebf-b180-cc02258183d5' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-0444bc62-cc94-4ebf-b180-cc02258183d5' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-dcb4921c-c3e6-4bd7-87b2-703f810d2df4' class='xr-var-data-in' type='checkbox'><label for='data-dcb4921c-c3e6-4bd7-87b2-703f810d2df4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([9012000., 9008000., 9004000., ..., 1700000., 1696000., 1692000.])</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-49d81424-f1cf-490f-93dc-1dcee699a419' class='xr-section-summary-in' type='checkbox'  checked><label for='section-49d81424-f1cf-490f-93dc-1dcee699a419' class='xr-section-summary' >Data variables: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>stacked_eumetsat_data</span></div><div class='xr-var-dims'>(time, x, y, variable)</div><div class='xr-var-dtype'>int16</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(36, 1870, 1831, 1), meta=np.ndarray&gt;</div><input id='attrs-c8918368-c661-48ff-ac4b-50ae3696c9bd' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-c8918368-c661-48ff-ac4b-50ae3696c9bd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-bb5c938e-e46b-44a9-8a3f-c8e8879c9c16' class='xr-var-data-in' type='checkbox'><label for='data-bb5c938e-e46b-44a9-8a3f-c8e8879c9c16' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>meta :</span></dt><dd>{&#x27;orbital_parameters&#x27;: {&#x27;projection_longitude&#x27;: 9.5, &#x27;projection_latitude&#x27;: 0.0, &#x27;projection_altitude&#x27;: 35785831.0}, &#x27;sun_earth_distance_correction_applied&#x27;: True, &#x27;sun_earth_distance_correction_factor&#x27;: 0.9911189780118609, &#x27;units&#x27;: &#x27;%&#x27;, &#x27;wavelength&#x27;: WavelengthRange(min=0.5, central=0.7, max=0.9, unit=&#x27;Âµm&#x27;), &#x27;standard_name&#x27;: &#x27;toa_bidirectional_reflectance&#x27;, &#x27;platform_name&#x27;: &#x27;Meteosat-10&#x27;, &#x27;sensor&#x27;: &#x27;seviri&#x27;, &#x27;start_time&#x27;: datetime.datetime(2021, 3, 19, 13, 15, 9, 278906), &#x27;end_time&#x27;: datetime.datetime(2021, 3, 19, 13, 20, 10, 330158), &#x27;area&#x27;: Area ID: geos_seviri_hrv
Description: SEVIRI high resolution channel area
Projection ID: seviri_hrv
Projection: {&#x27;a&#x27;: &#x27;6378169&#x27;, &#x27;h&#x27;: &#x27;35785831&#x27;, &#x27;lon_0&#x27;: &#x27;9.5&#x27;, &#x27;no_defs&#x27;: &#x27;None&#x27;, &#x27;proj&#x27;: &#x27;geos&#x27;, &#x27;rf&#x27;: &#x27;295.488065897014&#x27;, &#x27;type&#x27;: &#x27;crs&#x27;, &#x27;units&#x27;: &#x27;m&#x27;, &#x27;x_0&#x27;: &#x27;0&#x27;, &#x27;y_0&#x27;: &#x27;0&#x27;}
Number of columns: 5568
Number of rows: 4176
Area extent: (2790874.9005, 5571248.3904, -2777873.154, 1394687.3495), &#x27;name&#x27;: &#x27;HRV&#x27;, &#x27;resolution&#x27;: 1000.134348869, &#x27;calibration&#x27;: &#x27;reflectance&#x27;, &#x27;modifiers&#x27;: (), &#x27;_satpy_id&#x27;: DataID(name=&#x27;HRV&#x27;, wavelength=WavelengthRange(min=0.5, central=0.7, max=0.9, unit=&#x27;Âµm&#x27;), resolution=1000.134348869, calibration=&lt;calibration.reflectance&gt;, modifiers=()), &#x27;ancillary_variables&#x27;: []}</dd></dl></div><div class='xr-var-data'><table>
<tr>
<td>
<table>
  <thead>
    <tr><td> </td><th> Array </th><th> Chunk </th></tr>
  </thead>
  <tbody>
    <tr><th> Bytes </th><td> 958.33 GB </td> <td> 246.53 MB </td></tr>
    <tr><th> Shape </th><td> (11662, 1870, 1831, 12) </td> <td> (36, 1870, 1831, 1) </td></tr>
    <tr><th> Count </th><td> 3889 Tasks </td><td> 3888 Chunks </td></tr>
    <tr><th> Type </th><td> int16 </td><td> numpy.ndarray </td></tr>
  </tbody>
</table>
</td>
<td>
<svg width="479" height="115" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="120" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="3" y1="0" x2="3" y2="25" />
  <line x1="7" y1="0" x2="7" y2="25" />
  <line x1="11" y1="0" x2="11" y2="25" />
  <line x1="14" y1="0" x2="14" y2="25" />
  <line x1="18" y1="0" x2="18" y2="25" />
  <line x1="22" y1="0" x2="22" y2="25" />
  <line x1="25" y1="0" x2="25" y2="25" />
  <line x1="30" y1="0" x2="30" y2="25" />
  <line x1="33" y1="0" x2="33" y2="25" />
  <line x1="37" y1="0" x2="37" y2="25" />
  <line x1="41" y1="0" x2="41" y2="25" />
  <line x1="44" y1="0" x2="44" y2="25" />
  <line x1="48" y1="0" x2="48" y2="25" />
  <line x1="52" y1="0" x2="52" y2="25" />
  <line x1="55" y1="0" x2="55" y2="25" />
  <line x1="60" y1="0" x2="60" y2="25" />
  <line x1="63" y1="0" x2="63" y2="25" />
  <line x1="67" y1="0" x2="67" y2="25" />
  <line x1="71" y1="0" x2="71" y2="25" />
  <line x1="74" y1="0" x2="74" y2="25" />
  <line x1="78" y1="0" x2="78" y2="25" />
  <line x1="82" y1="0" x2="82" y2="25" />
  <line x1="85" y1="0" x2="85" y2="25" />
  <line x1="90" y1="0" x2="90" y2="25" />
  <line x1="93" y1="0" x2="93" y2="25" />
  <line x1="97" y1="0" x2="97" y2="25" />
  <line x1="101" y1="0" x2="101" y2="25" />
  <line x1="104" y1="0" x2="104" y2="25" />
  <line x1="108" y1="0" x2="108" y2="25" />
  <line x1="112" y1="0" x2="112" y2="25" />
  <line x1="115" y1="0" x2="115" y2="25" />
  <line x1="120" y1="0" x2="120" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,25.412616514582485 0.0,25.412616514582485" style="fill:#8B4903A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >11662</text>
  <text x="140.000000" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,140.000000,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="190" y1="0" x2="214" y2="24" style="stroke-width:2" />
  <line x1="190" y1="40" x2="214" y2="65" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="190" y1="0" x2="190" y2="40" style="stroke-width:2" />
  <line x1="214" y1="24" x2="214" y2="65" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="190.0,0.0 214.13178366684946,24.131783666849454 214.13178366684946,65.0564481243637 190.0,40.92466445751424" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="190" y1="0" x2="215" y2="0" style="stroke-width:2" />
  <line x1="214" y1="24" x2="239" y2="24" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="190" y1="0" x2="214" y2="24" style="stroke-width:2" />
  <line x1="192" y1="0" x2="216" y2="24" />
  <line x1="194" y1="0" x2="218" y2="24" />
  <line x1="196" y1="0" x2="220" y2="24" />
  <line x1="198" y1="0" x2="222" y2="24" />
  <line x1="200" y1="0" x2="224" y2="24" />
  <line x1="202" y1="0" x2="226" y2="24" />
  <line x1="204" y1="0" x2="228" y2="24" />
  <line x1="206" y1="0" x2="231" y2="24" />
  <line x1="209" y1="0" x2="233" y2="24" />
  <line x1="211" y1="0" x2="235" y2="24" />
  <line x1="213" y1="0" x2="237" y2="24" />
  <line x1="215" y1="0" x2="239" y2="24" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="190.0,0.0 215.41261651458248,0.0 239.54440018143194,24.131783666849454 214.13178366684946,24.131783666849454" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="214" y1="24" x2="239" y2="24" style="stroke-width:2" />
  <line x1="214" y1="65" x2="239" y2="65" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="214" y1="24" x2="214" y2="65" style="stroke-width:2" />
  <line x1="216" y1="24" x2="216" y2="65" />
  <line x1="218" y1="24" x2="218" y2="65" />
  <line x1="220" y1="24" x2="220" y2="65" />
  <line x1="222" y1="24" x2="222" y2="65" />
  <line x1="224" y1="24" x2="224" y2="65" />
  <line x1="226" y1="24" x2="226" y2="65" />
  <line x1="228" y1="24" x2="228" y2="65" />
  <line x1="231" y1="24" x2="231" y2="65" />
  <line x1="233" y1="24" x2="233" y2="65" />
  <line x1="235" y1="24" x2="235" y2="65" />
  <line x1="237" y1="24" x2="237" y2="65" />
  <line x1="239" y1="24" x2="239" y2="65" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="214.13178366684946,24.131783666849454 239.54440018143194,24.131783666849454 239.54440018143194,65.0564481243637 214.13178366684946,65.0564481243637" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="226.838092" y="85.056448" font-size="1.0rem" font-weight="100" text-anchor="middle" >12</text>
  <text x="259.544400" y="44.594116" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,259.544400,44.594116)">1831</text>
  <text x="192.065892" y="72.990556" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,192.065892,72.990556)">1870</text>
</svg>
</td>
</tr>
</table></div></li></ul></div></li><li class='xr-section-item'><input id='section-58d3333d-8314-48b8-8863-4c645990c2a8' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-58d3333d-8314-48b8-8863-4c645990c2a8' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>



<br>

We can then index this as we would any other `xarray` object

```python
da_HRV_sample = ds['stacked_eumetsat_data'].isel(time=slice(800, 900)).sel(variable='HRV')

da_HRV_sample
```




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: 'â–º';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: 'â–¼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;stacked_eumetsat_data&#x27; (time: 100, x: 1870, y: 1831)&gt;
dask.array&lt;getitem, shape=(100, 1870, 1831), dtype=int16, chunksize=(36, 1870, 1831), chunktype=numpy.ndarray&gt;
Coordinates:
  * time      (time) datetime64[ns] 2020-01-03T12:49:15 ... 2020-01-03T21:04:16
    variable  &lt;U3 &#x27;HRV&#x27;
  * x         (x) float64 -3.088e+06 -3.084e+06 ... 4.384e+06 4.388e+06
  * y         (y) float64 9.012e+06 9.008e+06 9.004e+06 ... 1.696e+06 1.692e+06
Attributes:
    meta:     {&#x27;orbital_parameters&#x27;: {&#x27;projection_longitude&#x27;: 9.5, &#x27;projectio...</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'stacked_eumetsat_data'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 100</li><li><span class='xr-has-index'>x</span>: 1870</li><li><span class='xr-has-index'>y</span>: 1831</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-80584332-8ccf-454b-ac3e-37914dbd464b' class='xr-array-in' type='checkbox' checked><label for='section-80584332-8ccf-454b-ac3e-37914dbd464b' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>dask.array&lt;chunksize=(28, 1870, 1831), meta=np.ndarray&gt;</span></div><div class='xr-array-data'><table>
<tr>
<td>
<table>
  <thead>
    <tr><td> </td><th> Array </th><th> Chunk </th></tr>
  </thead>
  <tbody>
    <tr><th> Bytes </th><td> 684.79 MB </td> <td> 246.53 MB </td></tr>
    <tr><th> Shape </th><td> (100, 1870, 1831) </td> <td> (36, 1870, 1831) </td></tr>
    <tr><th> Count </th><td> 3928 Tasks </td><td> 3 Chunks </td></tr>
    <tr><th> Type </th><td> int16 </td><td> numpy.ndarray </td></tr>
  </tbody>
</table>
</td>
<td>
<svg width="198" height="190" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="30" y2="20" style="stroke-width:2" />
  <line x1="10" y1="120" x2="30" y2="140" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="15" y1="5" x2="15" y2="125" />
  <line x1="23" y1="13" x2="23" y2="133" />
  <line x1="30" y1="20" x2="30" y2="140" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 30.53650492126805,20.53650492126805 30.53650492126805,140.53650492126806 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="127" y2="0" style="stroke-width:2" />
  <line x1="15" y1="5" x2="133" y2="5" />
  <line x1="23" y1="13" x2="140" y2="13" />
  <line x1="30" y1="20" x2="148" y2="20" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="30" y2="20" style="stroke-width:2" />
  <line x1="127" y1="0" x2="148" y2="20" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 127.49732620320856,0.0 148.03383112447662,20.53650492126805 30.53650492126805,20.53650492126805" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="30" y1="20" x2="148" y2="20" style="stroke-width:2" />
  <line x1="30" y1="140" x2="148" y2="140" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="30" y1="20" x2="30" y2="140" style="stroke-width:2" />
  <line x1="148" y1="20" x2="148" y2="140" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="30.53650492126805,20.53650492126805 148.03383112447662,20.53650492126805 148.03383112447662,140.53650492126806 30.53650492126805,140.53650492126806" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="89.285168" y="160.536505" font-size="1.0rem" font-weight="100" text-anchor="middle" >1831</text>
  <text x="168.033831" y="80.536505" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,168.033831,80.536505)">1870</text>
  <text x="10.268252" y="150.268252" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,10.268252,150.268252)">100</text>
</svg>
</td>
</tr>
</table></div></div></li><li class='xr-section-item'><input id='section-597ffa01-dffd-4a8c-bd8a-e196865d482d' class='xr-section-summary-in' type='checkbox'  checked><label for='section-597ffa01-dffd-4a8c-bd8a-e196865d482d' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2020-01-03T12:49:15 ... 2020-01-...</div><input id='attrs-711450ec-1937-4956-90aa-816e5161484b' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-711450ec-1937-4956-90aa-816e5161484b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c553e69d-dd60-488c-92b7-5de25f4f659e' class='xr-var-data-in' type='checkbox'><label for='data-c553e69d-dd60-488c-92b7-5de25f4f659e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;2020-01-03T12:49:15.000000000&#x27;, &#x27;2020-01-03T12:54:16.000000000&#x27;,
       &#x27;2020-01-03T12:59:16.000000000&#x27;, &#x27;2020-01-03T13:04:16.000000000&#x27;,
       &#x27;2020-01-03T13:09:16.000000000&#x27;, &#x27;2020-01-03T13:14:16.000000000&#x27;,
       &#x27;2020-01-03T13:19:16.000000000&#x27;, &#x27;2020-01-03T13:24:16.000000000&#x27;,
       &#x27;2020-01-03T13:29:16.000000000&#x27;, &#x27;2020-01-03T13:34:15.000000000&#x27;,
       &#x27;2020-01-03T13:39:15.000000000&#x27;, &#x27;2020-01-03T13:44:15.000000000&#x27;,
       &#x27;2020-01-03T13:49:15.000000000&#x27;, &#x27;2020-01-03T13:54:15.000000000&#x27;,
       &#x27;2020-01-03T13:59:15.000000000&#x27;, &#x27;2020-01-03T14:04:15.000000000&#x27;,
       &#x27;2020-01-03T14:09:15.000000000&#x27;, &#x27;2020-01-03T14:14:15.000000000&#x27;,
       &#x27;2020-01-03T14:19:15.000000000&#x27;, &#x27;2020-01-03T14:24:16.000000000&#x27;,
       &#x27;2020-01-03T14:29:17.000000000&#x27;, &#x27;2020-01-03T14:34:18.000000000&#x27;,
       &#x27;2020-01-03T14:39:18.000000000&#x27;, &#x27;2020-01-03T14:44:18.000000000&#x27;,
       &#x27;2020-01-03T14:49:18.000000000&#x27;, &#x27;2020-01-03T14:54:16.000000000&#x27;,
       &#x27;2020-01-03T14:59:16.000000000&#x27;, &#x27;2020-01-03T15:04:16.000000000&#x27;,
       &#x27;2020-01-03T15:09:16.000000000&#x27;, &#x27;2020-01-03T15:14:16.000000000&#x27;,
       &#x27;2020-01-03T15:19:16.000000000&#x27;, &#x27;2020-01-03T15:24:16.000000000&#x27;,
       &#x27;2020-01-03T15:29:16.000000000&#x27;, &#x27;2020-01-03T15:34:16.000000000&#x27;,
       &#x27;2020-01-03T15:39:16.000000000&#x27;, &#x27;2020-01-03T15:44:16.000000000&#x27;,
       &#x27;2020-01-03T15:49:15.000000000&#x27;, &#x27;2020-01-03T15:54:15.000000000&#x27;,
       &#x27;2020-01-03T15:59:15.000000000&#x27;, &#x27;2020-01-03T16:04:15.000000000&#x27;,
       &#x27;2020-01-03T16:09:15.000000000&#x27;, &#x27;2020-01-03T16:14:15.000000000&#x27;,
       &#x27;2020-01-03T16:19:15.000000000&#x27;, &#x27;2020-01-03T16:24:16.000000000&#x27;,
       &#x27;2020-01-03T16:29:17.000000000&#x27;, &#x27;2020-01-03T16:34:18.000000000&#x27;,
       &#x27;2020-01-03T16:39:18.000000000&#x27;, &#x27;2020-01-03T16:44:18.000000000&#x27;,
       &#x27;2020-01-03T16:49:18.000000000&#x27;, &#x27;2020-01-03T16:54:17.000000000&#x27;,
       &#x27;2020-01-03T16:59:17.000000000&#x27;, &#x27;2020-01-03T17:04:17.000000000&#x27;,
       &#x27;2020-01-03T17:09:16.000000000&#x27;, &#x27;2020-01-03T17:14:16.000000000&#x27;,
       &#x27;2020-01-03T17:19:16.000000000&#x27;, &#x27;2020-01-03T17:24:16.000000000&#x27;,
       &#x27;2020-01-03T17:29:16.000000000&#x27;, &#x27;2020-01-03T17:34:16.000000000&#x27;,
       &#x27;2020-01-03T17:39:16.000000000&#x27;, &#x27;2020-01-03T17:44:16.000000000&#x27;,
       &#x27;2020-01-03T17:49:16.000000000&#x27;, &#x27;2020-01-03T17:54:16.000000000&#x27;,
       &#x27;2020-01-03T17:59:16.000000000&#x27;, &#x27;2020-01-03T18:04:15.000000000&#x27;,
       &#x27;2020-01-03T18:09:15.000000000&#x27;, &#x27;2020-01-03T18:14:15.000000000&#x27;,
       &#x27;2020-01-03T18:19:15.000000000&#x27;, &#x27;2020-01-03T18:24:15.000000000&#x27;,
       &#x27;2020-01-03T18:29:15.000000000&#x27;, &#x27;2020-01-03T18:34:15.000000000&#x27;,
       &#x27;2020-01-03T18:39:15.000000000&#x27;, &#x27;2020-01-03T18:44:15.000000000&#x27;,
       &#x27;2020-01-03T18:49:15.000000000&#x27;, &#x27;2020-01-03T18:54:16.000000000&#x27;,
       &#x27;2020-01-03T18:59:16.000000000&#x27;, &#x27;2020-01-03T19:04:16.000000000&#x27;,
       &#x27;2020-01-03T19:09:16.000000000&#x27;, &#x27;2020-01-03T19:14:15.000000000&#x27;,
       &#x27;2020-01-03T19:19:15.000000000&#x27;, &#x27;2020-01-03T19:24:15.000000000&#x27;,
       &#x27;2020-01-03T19:29:15.000000000&#x27;, &#x27;2020-01-03T19:34:15.000000000&#x27;,
       &#x27;2020-01-03T19:39:15.000000000&#x27;, &#x27;2020-01-03T19:44:15.000000000&#x27;,
       &#x27;2020-01-03T19:49:15.000000000&#x27;, &#x27;2020-01-03T19:54:16.000000000&#x27;,
       &#x27;2020-01-03T19:59:16.000000000&#x27;, &#x27;2020-01-03T20:04:16.000000000&#x27;,
       &#x27;2020-01-03T20:09:16.000000000&#x27;, &#x27;2020-01-03T20:14:16.000000000&#x27;,
       &#x27;2020-01-03T20:19:15.000000000&#x27;, &#x27;2020-01-03T20:24:15.000000000&#x27;,
       &#x27;2020-01-03T20:29:15.000000000&#x27;, &#x27;2020-01-03T20:34:15.000000000&#x27;,
       &#x27;2020-01-03T20:39:15.000000000&#x27;, &#x27;2020-01-03T20:44:15.000000000&#x27;,
       &#x27;2020-01-03T20:49:15.000000000&#x27;, &#x27;2020-01-03T20:54:16.000000000&#x27;,
       &#x27;2020-01-03T20:59:16.000000000&#x27;, &#x27;2020-01-03T21:04:16.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>variable</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>&lt;U3</div><div class='xr-var-preview xr-preview'>&#x27;HRV&#x27;</div><input id='attrs-b902f2c6-31ce-450a-8116-d025a04e3c42' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-b902f2c6-31ce-450a-8116-d025a04e3c42' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-274d65d3-f692-40db-b0af-380228385854' class='xr-var-data-in' type='checkbox'><label for='data-274d65d3-f692-40db-b0af-380228385854' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(&#x27;HRV&#x27;, dtype=&#x27;&lt;U3&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>-3.088e+06 -3.084e+06 ... 4.388e+06</div><input id='attrs-861f41d0-4a47-41c2-9e67-23b1d4714076' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-861f41d0-4a47-41c2-9e67-23b1d4714076' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-cd303cef-6fc6-460c-805a-68ac08f64e42' class='xr-var-data-in' type='checkbox'><label for='data-cd303cef-6fc6-460c-805a-68ac08f64e42' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([-3088000., -3084000., -3080000., ...,  4380000.,  4384000.,  4388000.])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>9.012e+06 9.008e+06 ... 1.692e+06</div><input id='attrs-d2aa768d-a70d-4f0d-b2c8-6483352d9df2' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-d2aa768d-a70d-4f0d-b2c8-6483352d9df2' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-66ebd74f-0a4d-42df-95f4-26900f099633' class='xr-var-data-in' type='checkbox'><label for='data-66ebd74f-0a4d-42df-95f4-26900f099633' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([9012000., 9008000., 9004000., ..., 1700000., 1696000., 1692000.])</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-8c018f29-73ca-483e-a4f1-2e857b9fc7cf' class='xr-section-summary-in' type='checkbox'  checked><label for='section-8c018f29-73ca-483e-a4f1-2e857b9fc7cf' class='xr-section-summary' >Attributes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>meta :</span></dt><dd>{&#x27;orbital_parameters&#x27;: {&#x27;projection_longitude&#x27;: 9.5, &#x27;projection_latitude&#x27;: 0.0, &#x27;projection_altitude&#x27;: 35785831.0}, &#x27;sun_earth_distance_correction_applied&#x27;: True, &#x27;sun_earth_distance_correction_factor&#x27;: 0.9911189780118609, &#x27;units&#x27;: &#x27;%&#x27;, &#x27;wavelength&#x27;: WavelengthRange(min=0.5, central=0.7, max=0.9, unit=&#x27;Âµm&#x27;), &#x27;standard_name&#x27;: &#x27;toa_bidirectional_reflectance&#x27;, &#x27;platform_name&#x27;: &#x27;Meteosat-10&#x27;, &#x27;sensor&#x27;: &#x27;seviri&#x27;, &#x27;start_time&#x27;: datetime.datetime(2021, 3, 19, 13, 15, 9, 278906), &#x27;end_time&#x27;: datetime.datetime(2021, 3, 19, 13, 20, 10, 330158), &#x27;area&#x27;: Area ID: geos_seviri_hrv
Description: SEVIRI high resolution channel area
Projection ID: seviri_hrv
Projection: {&#x27;a&#x27;: &#x27;6378169&#x27;, &#x27;h&#x27;: &#x27;35785831&#x27;, &#x27;lon_0&#x27;: &#x27;9.5&#x27;, &#x27;no_defs&#x27;: &#x27;None&#x27;, &#x27;proj&#x27;: &#x27;geos&#x27;, &#x27;rf&#x27;: &#x27;295.488065897014&#x27;, &#x27;type&#x27;: &#x27;crs&#x27;, &#x27;units&#x27;: &#x27;m&#x27;, &#x27;x_0&#x27;: &#x27;0&#x27;, &#x27;y_0&#x27;: &#x27;0&#x27;}
Number of columns: 5568
Number of rows: 4176
Area extent: (2790874.9005, 5571248.3904, -2777873.154, 1394687.3495), &#x27;name&#x27;: &#x27;HRV&#x27;, &#x27;resolution&#x27;: 1000.134348869, &#x27;calibration&#x27;: &#x27;reflectance&#x27;, &#x27;modifiers&#x27;: (), &#x27;_satpy_id&#x27;: DataID(name=&#x27;HRV&#x27;, wavelength=WavelengthRange(min=0.5, central=0.7, max=0.9, unit=&#x27;Âµm&#x27;), resolution=1000.134348869, calibration=&lt;calibration.reflectance&gt;, modifiers=()), &#x27;ancillary_variables&#x27;: []}</dd></dl></div></li></ul></div></div>



<br>

### Extracting Time-Series

The coordinates are for a Transverse Mercator projection, we'll create a helper function that converts latitude and longitude into this coordinate system

```python
def convert_lon_lat_to_crs_coords(lon=0.1, lat=51.5):
    new_coords_point = (gpd.GeoDataFrame(geometry=[shapely.geometry.Point(lon, lat)], crs='EPSG:4326')
                        .to_crs('EPSG:3857')
                        .loc[0, 'geometry'])
    
    new_coords = (new_coords_point.x, new_coords_point.y)
    
    return new_coords

x, y = convert_lon_lat_to_crs_coords(lon=1, lat=50)

x, y
```




    (111319.49079327357, 6446275.841017158)



<br>

We'll now interpolate the data at this location and extract a time-series of the HRV intensity

```python
s_HRV = da_HRV_sample.interp(x=x, y=y).sortby('time').to_series()

s_HRV.plot()
```

    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\xarray\core\indexing.py:1369: PerformanceWarning: Slicing is producing a large chunk. To accept the large
    chunk and silence this warning, set the option
        >>> with dask.config.set(**{'array.slicing.split_large_chunks': False}):
        ...     array[indexer]
    
    To avoid creating the large chunks, set the option
        >>> with dask.config.set(**{'array.slicing.split_large_chunks': True}):
        ...     array[indexer]
      return self.array[key]
    




    <AxesSubplot:xlabel='time'>




![png](img/nbs/output_10_2.png)


<br>

We can also visualise the intensity distribution for this series

```python
sns.histplot(s_HRV)
```




    <AxesSubplot:xlabel='stacked_eumetsat_data', ylabel='Count'>




![png](img/nbs/output_12_1.png)

