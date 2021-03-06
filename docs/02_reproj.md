# Data Transformation



```python
#exports
import json
import pandas as pd
import xarray as xr
import numpy as np
import numpy.ma as ma

import matplotlib as mpl
import matplotlib.pyplot as plt
from matplotlib import colors
import seaborn as sns

import os
import time
from itertools import product
from collections import OrderedDict
from datetime import datetime
from ipypb import track
import FEAutils as hlp

import satpy
from satpy import Scene
from satpy.readers import seviri_l1b_native
import pyresample
from pyresample.geometry import AreaDefinition

try:
    import pyinterp
    import pyinterp.backends.xarray
except:
    pass
```

<br>

We'll separately install libraries that wont be needed for the `satip` module

```python
import rasterio
from rasterio import Affine as A
from rasterio.warp import reproject, Resampling, calculate_default_transform, transform
from rasterio.control import GroundControlPoint
from rasterio.transform import xy

import geopandas as gpd
from shapely.geometry import Point
import cartopy.crs as ccrs

from IPython.display import JSON
```

<br>

### User Input

```python
data_dir = '../data/raw'
intermediate_data_dir = '../data/intermediate'

calculate_reproj_coords = False
```

<br>

### Exploratory Data Analysis

We'll start by identifying the available files

```python
native_fps = sorted([f'{data_dir}/{f}' for f in os.listdir(data_dir) if '.nat' in f])

native_fps[0]
```




    '../data/raw/MSG2-SEVI-MSG15-0100-NA-20201208090415.301000000Z-NA.nat'



<br>

Then load one of them in as a SatPy scene

```python
native_fp = native_fps[0]

scene = Scene(filenames=[native_fp], reader='seviri_l1b_native')

scene
```




    <satpy.scene.Scene at 0x294fae42e50>



<br>

We can get a list of the available datasets (bands)

```python
scene.all_dataset_names()
```




    ['HRV',
     'IR_016',
     'IR_039',
     'IR_087',
     'IR_097',
     'IR_108',
     'IR_120',
     'IR_134',
     'VIS006',
     'VIS008',
     'WV_062',
     'WV_073']



<br>

Each band contains an XArray DataArray

```python
scene.load(['HRV'])

scene['HRV']
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
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;reshape-3b944f9ca9a40a223ab6382d90bfb37d&#x27; (y: 4176, x: 5568)&gt;
dask.array&lt;mul, shape=(4176, 5568), dtype=float32, chunksize=(1392, 5568), chunktype=numpy.ndarray&gt;
Coordinates:
    crs      object PROJCRS[&quot;unknown&quot;,BASEGEOGCRS[&quot;unknown&quot;,DATUM[&quot;unknown&quot;,E...
  * y        (y) float64 1.395e+06 1.396e+06 1.397e+06 ... 5.57e+06 5.571e+06
  * x        (x) float64 3.164e+06 3.163e+06 3.162e+06 ... -2.402e+06 -2.403e+06
Attributes:
    orbital_parameters:                     {&#x27;projection_longitude&#x27;: 9.5, &#x27;pr...
    sun_earth_distance_correction_applied:  True
    sun_earth_distance_correction_factor:   0.9697642568677852
    units:                                  %
    wavelength:                             0.7â€¯ÂµmÂ (0.5-0.9â€¯Âµm)
    standard_name:                          toa_bidirectional_reflectance
    platform_name:                          Meteosat-9
    sensor:                                 seviri
    start_time:                             2020-12-08 09:00:08.206321
    end_time:                               2020-12-08 09:05:08.329479
    area:                                   Area ID: geos_seviri_hrv\nDescrip...
    name:                                   HRV
    resolution:                             1000.134348869
    calibration:                            reflectance
    modifiers:                              ()
    _satpy_id:                              DataID(name=&#x27;HRV&#x27;, wavelength=Wav...
    ancillary_variables:                    []</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'reshape-3b944f9ca9a40a223ab6382d90bfb37d'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>y</span>: 4176</li><li><span class='xr-has-index'>x</span>: 5568</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-ad262609-2bbd-4fca-8918-0218e4d9c21e' class='xr-array-in' type='checkbox' checked><label for='section-ad262609-2bbd-4fca-8918-0218e4d9c21e' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>dask.array&lt;chunksize=(1392, 5568), meta=np.ndarray&gt;</span></div><div class='xr-array-data'><table>
<tr>
<td>
<table>
  <thead>
    <tr><td> </td><th> Array </th><th> Chunk </th></tr>
  </thead>
  <tbody>
    <tr><th> Bytes </th><td> 93.01 MB </td> <td> 31.00 MB </td></tr>
    <tr><th> Shape </th><td> (4176, 5568) </td> <td> (1392, 5568) </td></tr>
    <tr><th> Count </th><td> 214 Tasks </td><td> 3 Chunks </td></tr>
    <tr><th> Type </th><td> float32 </td><td> numpy.ndarray </td></tr>
  </tbody>
</table>
</td>
<td>
<svg width="170" height="140" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="30" x2="120" y2="30" />
  <line x1="0" y1="60" x2="120" y2="60" />
  <line x1="0" y1="90" x2="120" y2="90" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="90" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="90" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,90.0 0.0,90.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="110.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" >5568</text>
  <text x="140.000000" y="45.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,140.000000,45.000000)">4176</text>
</svg>
</td>
</tr>
</table></div></div></li><li class='xr-section-item'><input id='section-be78b272-a827-49ad-bbd4-e67a22f11f81' class='xr-section-summary-in' type='checkbox'  checked><label for='section-be78b272-a827-49ad-bbd4-e67a22f11f81' class='xr-section-summary' >Coordinates: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>crs</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>PROJCRS[&quot;unknown&quot;,BASEGEOGCRS[&quot;u...</div><input id='attrs-24a4516d-47d6-4b84-a529-9b9a27d0c80c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-24a4516d-47d6-4b84-a529-9b9a27d0c80c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-dfe2da16-3982-4740-a746-8618e52c8eb0' class='xr-var-data-in' type='checkbox'><label for='data-dfe2da16-3982-4740-a746-8618e52c8eb0' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(&lt;Projected CRS: PROJCRS[&quot;unknown&quot;,BASEGEOGCRS[&quot;unknown&quot;,DATUM[&quot;unk ...&gt;
Name: unknown
Axis Info [cartesian]:
- E[east]: Easting (metre)
- N[north]: Northing (metre)
Area of Use:
- undefined
Coordinate Operation:
- name: unknown
- method: Geostationary Satellite (Sweep Y)
Datum: unknown
- Ellipsoid: unknown
- Prime Meridian: Greenwich
, dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>1.395e+06 1.396e+06 ... 5.571e+06</div><input id='attrs-bec74a8d-3724-442d-93c8-52349b020eac' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-bec74a8d-3724-442d-93c8-52349b020eac' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c42e9782-c97e-4139-b747-66f0c2e3e096' class='xr-var-data-in' type='checkbox'><label for='data-c42e9782-c97e-4139-b747-66f0c2e3e096' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>meter</dd></dl></div><div class='xr-var-data'><pre>array([1395187.416673, 1396187.551022, 1397187.68537 , ..., 5568748.054504,
       5569748.188853, 5570748.323202])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>3.164e+06 3.163e+06 ... -2.403e+06</div><input id='attrs-3cb7ed9e-a4f5-443d-8076-214e3b04b819' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-3cb7ed9e-a4f5-443d-8076-214e3b04b819' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-57d04bb2-132c-4a43-bfac-a49dff670d0f' class='xr-var-data-in' type='checkbox'><label for='data-57d04bb2-132c-4a43-bfac-a49dff670d0f' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>meter</dd></dl></div><div class='xr-var-data'><pre>array([ 3164425.079823,  3163424.945474,  3162424.811125, ..., -2401322.571635,
       -2402322.705984, -2403322.840333])</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-294d7518-d732-4a0f-98de-547f0a52450d' class='xr-section-summary-in' type='checkbox'  ><label for='section-294d7518-d732-4a0f-98de-547f0a52450d' class='xr-section-summary' >Attributes: <span>(17)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>orbital_parameters :</span></dt><dd>{&#x27;projection_longitude&#x27;: 9.5, &#x27;projection_latitude&#x27;: 0.0, &#x27;projection_altitude&#x27;: 35785831.0}</dd><dt><span>sun_earth_distance_correction_applied :</span></dt><dd>True</dd><dt><span>sun_earth_distance_correction_factor :</span></dt><dd>0.9697642568677852</dd><dt><span>units :</span></dt><dd>%</dd><dt><span>wavelength :</span></dt><dd>0.7â€¯ÂµmÂ (0.5-0.9â€¯Âµm)</dd><dt><span>standard_name :</span></dt><dd>toa_bidirectional_reflectance</dd><dt><span>platform_name :</span></dt><dd>Meteosat-9</dd><dt><span>sensor :</span></dt><dd>seviri</dd><dt><span>start_time :</span></dt><dd>2020-12-08 09:00:08.206321</dd><dt><span>end_time :</span></dt><dd>2020-12-08 09:05:08.329479</dd><dt><span>area :</span></dt><dd>Area ID: geos_seviri_hrv
Description: SEVIRI high resolution channel area
Projection ID: seviri_hrv
Projection: {&#x27;a&#x27;: &#x27;6378169&#x27;, &#x27;h&#x27;: &#x27;35785831&#x27;, &#x27;lon_0&#x27;: &#x27;9.5&#x27;, &#x27;no_defs&#x27;: &#x27;None&#x27;, &#x27;proj&#x27;: &#x27;geos&#x27;, &#x27;rf&#x27;: &#x27;295.488065897014&#x27;, &#x27;type&#x27;: &#x27;crs&#x27;, &#x27;units&#x27;: &#x27;m&#x27;, &#x27;x_0&#x27;: &#x27;0&#x27;, &#x27;y_0&#x27;: &#x27;0&#x27;}
Number of columns: 5568
Number of rows: 4176
Area extent: (3164925.147, 5571248.3904, -2403822.9075, 1394687.3495)</dd><dt><span>name :</span></dt><dd>HRV</dd><dt><span>resolution :</span></dt><dd>1000.134348869</dd><dt><span>calibration :</span></dt><dd>reflectance</dd><dt><span>modifiers :</span></dt><dd>()</dd><dt><span>_satpy_id :</span></dt><dd>DataID(name=&#x27;HRV&#x27;, wavelength=WavelengthRange(min=0.5, central=0.7, max=0.9, unit=&#x27;Âµm&#x27;), resolution=1000.134348869, calibration=&lt;calibration.reflectance&gt;, modifiers=())</dd><dt><span>ancillary_variables :</span></dt><dd>[]</dd></dl></div></li></ul></div></div>



<br>

We can see that the DataArray contains a crs, however we'll make our own custom area definition that's more accurate. First we'll create a helper function that will create our area definitions.

```python
#exports

def calculate_x_offset(native_fp):
    handler = seviri_l1b_native.NativeMSGFileHandler(native_fp, {}, None)
    lower_east_column_planned = handler.header['15_DATA_HEADER']['ImageDescription']['PlannedCoverageHRV']['LowerEastColumnPlanned']
    x_offset = 32500 + ((2733 - lower_east_column_planned) * 1000)

    return x_offset

def get_seviri_area_def(native_fp, num_x_pixels=5568, num_y_pixels=4176) -> AreaDefinition:
    """
    The HRV channel on Meteosat Second Generation satellites doesn't scan the full number of columns.
    The east boundary of the HRV channel changes (e.g. to maximise the amount of the image which
    is illuminated by sunlight.
    
    Parameters:
        native_fp: Data filepath
        

    """
    
    x_offset = calculate_x_offset(native_fp)
    
    # The EUMETSAT docs say "The distance between spacecraft and centre of earth is 42,164 km. The idealized earth
    # is a perfect ellipsoid with an equator radius of 6378.1690 km and a polar radius of 6356.5838 km." 
    # The projection used by SatPy expresses height as height above the Earth's surface (not distance
    # to the centre of the Earth).
    
    projection = {
        'proj': 'geos',
        'lon_0': 9.5,
        'a': 6378169.0,
        'b': 6356583.8,
        'h': 35785831.00,
        'units': 'm'}

    seviri = AreaDefinition(
        area_id='seviri',
        description='SEVIRI RSS HRV',
        proj_id='seviri',   
        projection=projection,
        width=num_x_pixels,
        height=num_y_pixels,
        area_extent=[
            -2768872.0236 + x_offset, # left
             1394687.3495,            # bottom (from scene['HRV'].area)
             2799876.1893 + x_offset, # right
             5570248.4773]            # top (from scene['HRV'].area)
    )

    return seviri
```

<br>

Then we'll use it to construct the relevant one for Seviri

```python
seviri = get_seviri_area_def(native_fp)
seviri_crs = seviri.to_cartopy_crs()

seviri_crs
```

    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\pyproj\crs\crs.py:543: UserWarning: You will likely lose important projection information when converting to a PROJ string from another format. See: https://proj.org/faq.html#what-is-the-best-format-for-describing-coordinate-reference-systems
      proj_string = self.to_proj4()
    




<?xml version="1.0" encoding="utf-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
  "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<!-- Created with matplotlib (https://matplotlib.org/) -->
<svg height="177.48pt" version="1.1" viewBox="0 0 231.892076 177.48" width="231.892076pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
 <metadata>
  <rdf:RDF xmlns:cc="http://creativecommons.org/ns#" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
   <cc:Work>
    <dc:type rdf:resource="http://purl.org/dc/dcmitype/StillImage"/>
    <dc:date>2020-12-16T23:15:15.552296</dc:date>
    <dc:format>image/svg+xml</dc:format>
    <dc:creator>
     <cc:Agent>
      <dc:title>Matplotlib v3.3.2, https://matplotlib.org/</dc:title>
     </cc:Agent>
    </dc:creator>
   </cc:Work>
  </rdf:RDF>
 </metadata>
 <defs>
  <style type="text/css">*{stroke-linecap:butt;stroke-linejoin:round;}</style>
 </defs>
 <g id="figure_1">
  <g id="patch_1">
   <path d="M 0 177.48 
L 231.892076 177.48 
L 231.892076 0 
L 0 0 
z
" style="fill:none;"/>
  </g>
  <g id="axes_1">
   <g id="patch_2">
    <path d="M 7.2 170.28 
L 7.2 7.2 
L 224.692076 7.2 
L 224.692076 170.28 
L 7.2 170.28 
" style="fill:#ffffff;"/>
   </g>
   <g id="PathCollection_1">
    <path clip-path="url(#p82c2cc087c)" d="M 64.164393 39.917195 
L 63.860271 41.299359 
L 61.225519 43.168999 
L 56.42198 44.573751 
L 53.265485 44.418713 
L 56.334065 42.192808 
L 56.32298 40.246189 
L 60.216677 38.606453 
L 62.375492 37.661085 
L 64.252596 37.505267 
L 66.030268 38.558925 
L 64.164393 39.917195 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 178.497312 88.102676 
L 180.687263 88.252691 
L 183.067069 87.459912 
L 181.530603 88.688229 
L 182.14051 89.547344 
L 179.465711 90.681331 
L 177.827138 90.197552 
L 176.69224 88.893433 
L 178.315951 88.839399 
L 178.497312 88.102676 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 157.709797 87.518371 
L 157.536222 88.435596 
L 152.899609 88.555455 
L 152.814657 88.034185 
L 148.673611 87.302101 
L 148.995578 85.985874 
L 151.015768 87.094096 
L 153.514831 86.992186 
L 155.98371 87.292418 
L 156.03826 87.837852 
L 157.709797 87.518371 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 98.651945 69.893575 
L 100.233914 69.025393 
L 102.12819 71.018707 
L 101.693958 74.818 
L 100.219202 74.634394 
L 98.884833 75.611055 
L 97.665659 74.837943 
L 97.606297 71.367532 
L 96.92111 69.757363 
L 98.651945 69.893575 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 98.836418 65.143792 
L 100.809485 64.118811 
L 101.329036 66.429541 
L 100.298333 68.55288 
L 98.885972 67.991071 
L 98.199426 66.14973 
L 98.836418 65.143792 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 107.607694 35.083532 
L 108.427038 35.979942 
L 107.191366 37.446568 
L 104.693768 36.399189 
L 104.33569 35.65219 
L 107.607694 35.083532 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 72.754099 31.490067 
L 75.266577 31.271515 
L 72.16966 33.108301 
L 74.407436 32.807817 
L 76.695234 32.744369 
L 75.541404 34.132888 
L 72.902983 35.75872 
L 75.127735 35.800927 
L 76.415282 38.084521 
L 77.888538 38.344411 
L 78.65169 40.4978 
L 79.111403 41.262012 
L 81.949221 41.579796 
L 81.340484 42.879685 
L 79.929886 43.508515 
L 80.66893 44.566628 
L 78.120055 45.725556 
L 74.730372 45.786489 
L 70.24021 46.496772 
L 69.210628 46.108196 
L 67.186162 47.175377 
L 64.948758 47.006793 
L 62.855522 47.905584 
L 61.699486 47.517078 
L 66.251631 45.12496 
L 68.628914 44.596226 
L 64.929814 44.377864 
L 64.598337 43.577536 
L 67.374688 42.851824 
L 66.531798 41.810331 
L 67.54585 40.498692 
L 70.946222 40.557627 
L 71.73825 39.421222 
L 70.66914 38.278018 
L 68.081346 38.046151 
L 67.792575 37.56415 
L 68.997568 36.716646 
L 68.5251 36.246655 
L 66.908458 37.148692 
L 67.691557 35.423919 
L 67.11214 34.57832 
L 68.875134 32.801813 
L 71.217762 31.423904 
L 72.754099 31.490067 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 64.672614 21.882372 
L 63.339887 22.610265 
L 63.969739 23.272559 
L 60.839579 24.309241 
L 55.314707 25.487134 
L 53.687054 25.825747 
L 51.973677 25.801664 
L 48.524247 25.765424 
L 50.786452 25.077607 
L 48.757837 24.767798 
L 51.554899 24.298877 
L 52.074351 23.956493 
L 49.756109 23.959037 
L 51.976792 23.124872 
L 54.23233 22.769828 
L 54.999383 23.341266 
L 58.0709 22.5534 
L 59.300687 22.725358 
L 62.464999 21.971341 
L 64.672614 21.882372 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 144.83448 17.718004 
L 144.734018 17.70058 
L 145.600869 17.959836 
L 147.32214 18.462346 
L 148.786615 18.991749 
L 151.191581 19.687427 
L 154.984289 20.658952 
L 154.745129 20.655852 
L 151.62412 20.0972 
L 150.23525 19.681316 
L 147.976097 19.23205 
L 146.486223 18.785398 
L 146.800098 18.741952 
L 145.370521 18.32961 
L 144.667025 17.954324 
L 143.662323 17.767581 
L 143.01024 17.462472 
L 141.59287 17.155346 
L 141.544169 17.08296 
L 141.596102 17.069898 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 112.694123 13.552115 
L 112.351956 13.576087 
L 110.52323 13.513272 
L 108.389053 13.425346 
L 108.077472 13.389779 
L 107.119172 13.363198 
L 106.153493 13.306841 
L 107.893762 13.337133 
L 108.972327 13.381731 
L 109.33171 13.382655 
L 111.097932 13.461639 
L 112.694123 13.552115 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 113.23539 13.867051 
L 111.8521 13.905965 
L 110.245661 13.798058 
L 110.591154 13.751842 
L 109.877564 13.664802 
L 111.269569 13.674945 
L 111.9081 13.768375 
L 113.23539 13.867051 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 104.986253 13.339573 
L 105.105926 13.309139 
L 106.049085 13.323418 
L 107.072572 13.37708 
L 109.870224 13.546978 
L 108.318622 13.564323 
L 108.346492 13.711961 
L 107.750153 13.74195 
L 107.78826 13.963951 
L 106.759399 13.956049 
L 104.70596 13.755735 
L 105.338202 13.675622 
L 104.052715 13.594785 
L 102.411625 13.422589 
L 101.78948 13.313789 
L 103.563137 13.289595 
L 104.022539 13.32833 
L 104.986253 13.339573 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 118.464281 77.958148 
L 120.924009 77.727352 
L 119.972372 80.073174 
L 120.557978 81.018433 
L 120.00305 82.582545 
L 117.346624 81.4024 
L 115.624305 81.057847 
L 110.885486 79.483895 
L 111.269866 77.948197 
L 115.139503 78.250052 
L 118.464281 77.958148 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 179.31973 62.353598 
L 182.705362 63.52938 
L 187.142398 65.979089 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 7.200002 127.351983 
L 8.455117 126.084894 
L 9.722416 125.447506 
L 11.098215 123.272373 
L 11.62635 121.326726 
L 13.362049 119.040928 
L 15.905316 117.6419 
L 18.897975 113.91459 
L 20.982185 112.43983 
L 24.206646 111.933816 
L 27.426311 109.439486 
L 29.337586 108.45061 
L 32.822459 105.463877 
L 32.97423 101.254621 
L 34.938007 98.326516 
L 35.812574 96.56568 
L 38.478142 94.273924 
L 42.136696 92.676027 
L 44.892197 91.261961 
L 47.852847 87.893387 
L 49.327273 85.933858 
L 51.715065 85.876089 
L 53.397551 87.149433 
L 56.577487 86.846787 
L 59.883665 87.455969 
L 61.327977 87.45563 
L 64.829671 85.667402 
L 68.523724 85.051666 
L 70.817102 83.731684 
L 74.150955 82.738404 
L 79.863419 82.109404 
L 85.414552 81.799457 
L 87.072668 82.239216 
L 90.300829 81.011282 
L 93.876643 80.968157 
L 95.217668 81.672424 
L 97.519328 81.4803 
L 101.177166 80.248992 
L 103.51872 80.614589 
L 103.437757 82.158531 
L 106.267665 81.040094 
L 106.521423 81.626887 
L 104.868003 83.127291 
L 104.870132 84.561323 
L 106.067363 85.33992 
L 105.671738 88.069349 
L 103.408515 89.671585 
L 104.092594 91.430971 
L 105.909542 91.489877 
L 106.828662 93.040647 
L 108.186299 93.557203 
L 112.390263 94.709332 
L 113.866419 94.435024 
L 116.858268 95.007387 
L 121.672598 96.533103 
L 123.581683 99.531087 
L 126.860201 100.225913 
L 132.060058 101.718772 
L 136.077785 103.481163 
L 137.717013 102.622213 
L 139.220589 101.086081 
L 138.049014 98.485624 
L 138.923107 96.882192 
L 141.197143 95.380911 
L 143.500658 94.983438 
L 148.25991 95.768954 
L 149.698202 97.280149 
L 150.978057 97.327323 
L 152.175926 97.92437 
L 155.648086 98.410003 
L 156.701988 99.539732 
L 161.212129 99.620652 
L 168.219456 101.72511 
L 169.893212 102.306901 
L 172.188119 101.329304 
L 173.296861 100.416924 
L 176.102641 100.254255 
L 178.521874 100.772293 
L 179.864365 102.479797 
L 180.307491 101.409686 
L 183.105892 102.317299 
L 185.645255 102.619929 
L 186.945996 101.84352 
L 187.518475 100.782355 
L 187.246272 100.58255 
L 187.577957 99.066934 
L 187.391225 96.611906 
L 187.62346 95.777307 
L 187.787915 93.195988 
L 188.46932 90.916843 
L 187.287965 88.489785 
L 187.506234 87.246395 
L 185.890108 85.78271 
L 186.522991 84.691726 
L 184.869249 84.852296 
L 182.164949 84.014116 
L 180.811799 85.673358 
L 176.399348 85.792873 
L 173.386399 84.023725 
L 170.080457 83.77122 
L 169.787375 85.015883 
L 167.768462 85.291282 
L 164.269006 83.528382 
L 160.880186 83.450965 
L 158.169846 80.362164 
L 155.434177 78.620925 
L 156.263165 76.382294 
L 153.922811 74.920408 
L 156.40798 72.305666 
L 160.908052 72.379849 
L 161.360357 70.298297 
L 166.949622 70.923946 
L 169.574785 69.297125 
L 172.434427 68.696043 
L 176.872872 68.899009 
L 182.469216 71.134301 
L 186.788905 72.465673 
L 189.583002 72.246247 
L 191.914997 72.65641 
L 194.067055 71.429953 
L 193.756056 70.281872 
L 192.054728 68.377048 
L 190.05817 67.275089 
L 188.520888 66.86259 
L 187.142398 65.979089 
L 182.705362 63.52938 
L 179.31973 62.353598 
L 176.315061 60.707053 
L 177.873679 60.426796 
L 178.610021 58.482933 
L 176.667327 57.413855 
L 179.496173 56.679867 
L 179.083574 56.147864 
L 177.248277 56.378165 
L 175.487037 56.434073 
L 174.382898 57.107452 
L 172.190576 57.081977 
L 170.599703 57.858045 
L 171.609204 59.41234 
L 173.172542 60.09695 
L 175.627311 60.113723 
L 175.657119 60.983463 
L 173.145742 61.245109 
L 170.433265 62.51736 
L 168.720454 61.907759 
L 168.685558 60.739012 
L 165.482978 59.835299 
L 165.698802 59.378523 
L 167.721085 58.689811 
L 166.685284 58.077833 
L 162.399566 57.229601 
L 161.762849 56.308716 
L 159.539158 56.481623 
L 159.222946 57.784812 
L 158.017047 59.530184 
L 158.365042 60.186039 
L 157.294526 60.67193 
L 156.37259 60.394856 
L 156.870916 63.48052 
L 155.84381 64.510341 
L 155.515509 66.380278 
L 156.995227 67.969218 
L 157.71342 69.040055 
L 160.553963 70.043577 
L 160.274998 70.703535 
L 156.867681 70.712043 
L 155.900046 71.530482 
L 153.89648 72.962599 
L 152.55186 71.605987 
L 152.414358 71.026166 
L 150.574428 70.880706 
L 148.936901 70.561196 
L 145.492912 71.172231 
L 148.024537 72.834123 
L 146.597387 73.250574 
L 144.889799 73.200942 
L 142.90423 71.685882 
L 142.480946 72.292304 
L 143.580552 74.023226 
L 145.461294 75.428968 
L 144.438305 76.03435 
L 146.505588 77.441019 
L 148.269506 78.347705 
L 148.726378 80.038616 
L 145.604399 79.157117 
L 146.893312 80.717871 
L 144.929082 80.978133 
L 146.736261 83.708875 
L 144.582947 83.688781 
L 141.643812 82.284543 
L 139.954092 79.827824 
L 139.001036 77.825661 
L 135.547767 74.733859 
L 135.176954 73.904569 
L 134.604253 73.690591 
L 134.42148 73.056698 
L 132.510633 72.059273 
L 131.991951 70.704496 
L 131.893502 68.803592 
L 132.143752 67.955352 
L 131.547676 67.511237 
L 130.871358 67.285253 
L 129.853722 66.377185 
L 128.450103 65.81277 
L 125.447411 64.763588 
L 123.554518 63.773581 
L 120.706156 62.944642 
L 117.959909 60.989404 
L 118.530339 60.804262 
L 117.058335 59.707764 
L 116.900056 58.849461 
L 114.988462 58.429541 
L 114.20878 59.513411 
L 113.265269 58.655759 
L 113.346638 57.744354 
L 113.966176 57.524312 
L 111.640365 57.140213 
L 109.357547 58.008523 
L 109.599649 59.262915 
L 109.285768 59.987386 
L 110.339798 61.306243 
L 113.239013 62.644145 
L 114.953151 64.854444 
L 118.58042 67.076635 
L 120.986339 67.091848 
L 121.814098 67.708971 
L 121.018706 68.245727 
L 126.331749 70.174391 
L 129.241411 71.695769 
L 129.653747 72.233494 
L 129.232356 73.247488 
L 127.275468 71.880316 
L 124.478534 71.36703 
L 123.392448 73.193174 
L 125.836155 74.299343 
L 125.661566 75.814535 
L 124.337645 75.96768 
L 122.902462 78.483521 
L 121.56046 78.695941 
L 121.476988 77.783639 
L 121.96779 76.205738 
L 122.598637 75.588475 
L 120.017766 72.422561 
L 118.662734 72.050674 
L 117.606746 70.815207 
L 115.535896 70.281545 
L 114.085009 69.14369 
L 111.757845 68.944328 
L 109.248979 67.679154 
L 106.345934 65.888097 
L 104.223522 64.331841 
L 103.235719 61.721254 
L 101.747596 61.417169 
L 99.333119 60.564332 
L 97.9609 60.915633 
L 96.201969 62.127952 
L 94.948903 62.325144 
L 92.140246 63.832119 
L 86.253763 63.167493 
L 81.773728 64.083807 
L 81.213639 65.704274 
L 81.171618 67.282818 
L 77.997487 69.167097 
L 73.936198 69.82683 
L 73.502872 70.776863 
L 71.31669 72.387375 
L 69.694785 74.756814 
L 70.656306 76.402501 
L 68.577787 77.758973 
L 67.547936 79.716929 
L 64.990429 80.371453 
L 62.252883 82.766229 
L 55.026869 82.943383 
L 52.758952 84.088827 
L 51.254794 85.296719 
L 49.731328 85.087526 
L 48.792521 84.079782 
L 48.33098 82.344971 
L 45.507291 81.974517 
L 44.027447 82.810689 
L 42.494287 82.442389 
L 40.786826 82.839186 
L 41.957999 80.44999 
L 42.224328 78.624107 
L 40.945678 78.404887 
L 40.578211 77.315949 
L 41.441175 75.394546 
L 42.975863 74.296891 
L 43.569091 73.130281 
L 44.754562 71.40131 
L 45.092694 70.217176 
L 44.857456 69.244282 
L 45.058844 68.321164 
L 45.902562 66.377407 
L 45.181573 65.262909 
L 49.865959 63.184959 
L 53.156236 63.523675 
L 56.994274 63.369013 
L 59.925735 63.718179 
L 62.373204 63.505503 
L 67.070598 63.461291 
L 68.951151 61.850544 
L 70.707737 56.786759 
L 68.531795 54.309588 
L 66.888728 53.163081 
L 63.108191 52.389131 
L 63.441653 50.710956 
L 66.991947 50.103704 
L 71.222943 50.56198 
L 71.15266 48.037886 
L 73.321488 48.926786 
L 79.814098 47.063642 
L 80.981865 45.282211 
L 85.405626 44.365287 
L 86.806306 43.772523 
L 89.449042 40.763331 
L 92.869795 39.909174 
L 94.870039 39.94894 
L 95.373596 39.535584 
L 97.387952 39.420749 
L 97.814264 39.8456 
L 99.476032 38.892581 
L 98.952705 38.18383 
L 98.879216 37.129163 
L 97.983962 36.121719 
L 98.006412 34.318485 
L 98.398537 33.853429 
L 99.050198 33.342294 
L 100.978777 33.23518 
L 101.740948 32.773959 
L 103.464432 32.310818 
L 103.426244 33.16484 
L 102.795929 33.714124 
L 103.075843 34.194267 
L 104.293338 34.459356 
L 103.776556 35.116561 
L 103.099202 34.924018 
L 101.487798 36.202241 
L 102.124481 37.089444 
L 102.176294 37.805336 
L 104.570723 38.247817 
L 104.577693 38.9204 
L 106.97275 38.57521 
L 108.253407 38.068069 
L 111.010364 38.838417 
L 112.222866 39.460879 
L 113.758141 38.918364 
L 120.009746 37.525783 
L 122.400275 37.890267 
L 122.714494 38.351237 
L 124.955036 38.430136 
L 125.195438 37.621562 
L 128.096385 37.112945 
L 127.007201 35.583916 
L 126.531489 34.272242 
L 127.106102 33.239005 
L 128.805498 32.735834 
L 131.105817 34.049127 
L 132.78022 34.079072 
L 132.143773 31.851774 
L 131.518232 32.026264 
L 129.912666 31.407208 
L 129.22912 30.497665 
L 131.453769 30.163409 
L 133.746956 30.043793 
L 136.010436 30.392024 
L 137.949996 30.441993 
L 139.418263 29.710395 
L 136.904421 28.902837 
L 133.627184 28.858821 
L 130.675203 29.260043 
L 127.730986 29.457351 
L 126.153805 28.608944 
L 124.043591 28.071986 
L 123.695294 26.702162 
L 122.120042 25.468814 
L 122.510607 24.747953 
L 123.55465 24.020432 
L 126.328865 22.880193 
L 127.18119 22.686723 
L 126.577836 22.205782 
L 123.949777 21.60305 
L 121.521035 21.809079 
L 120.531204 22.531194 
L 121.264885 23.246166 
L 119.151865 24.13312 
L 116.275208 25.131818 
L 115.670401 26.988445 
L 117.39953 28.01346 
L 119.581213 28.866631 
L 118.387211 30.538924 
L 116.371142 30.861853 
L 116.333744 33.59188 
L 115.480291 35.189272 
L 112.733143 34.979386 
L 111.699293 36.376335 
L 109.057823 36.432535 
L 108.140304 34.745139 
L 106.101131 32.798548 
L 104.314131 30.509519 
L 102.887727 29.560139 
L 98.783673 31.365122 
L 95.933586 31.75543 
L 93.125026 30.964292 
L 92.671479 29.312253 
L 92.715712 26.035304 
L 94.586513 25.16947 
L 99.464782 24.080133 
L 102.896143 22.845889 
L 105.749547 21.314342 
L 108.930979 19.436644 
L 114.119839 17.839639 
L 116.667937 17.583439 
L 118.822305 17.697481 
L 119.933361 17.155301 
L 122.187626 17.284577 
L 124.094321 17.246646 
L 128.783454 18.005416 
L 127.594138 18.122684 
L 129.817636 18.716412 
L 130.59242 18.507426 
L 133.535319 19.153254 
L 137.298658 19.612624 
L 144.011683 21.075729 
L 145.892526 21.644876 
L 147.291126 22.353532 
L 146.865439 22.793358 
L 145.163287 22.897802 
L 137.716219 21.601004 
L 136.887107 21.664215 
L 140.347445 22.617792 
L 142.939775 24.359487 
L 147.009096 25.29622 
L 146.361016 24.707273 
L 144.732285 24.115815 
L 145.000471 23.739871 
L 149.590423 24.778618 
L 150.318687 24.588451 
L 147.987537 23.634826 
L 149.160591 22.816453 
L 150.455059 22.997797 
L 152.372183 23.513835 
L 151.630602 22.822865 
L 149.344449 22.07566 
L 148.61345 21.507731 
L 146.481288 20.818095 
L 150.353545 21.480554 
L 152.264978 22.128276 
L 151.084596 22.089942 
L 152.321868 22.68315 
L 154.023713 23.165726 
L 155.34532 23.140793 
L 154.138592 22.483715 
L 156.630057 21.893221 
L 157.475197 22.052333 
L 158.106416 22.532716 
L 159.645131 22.843003 
L 159.369349 22.576391 
L 161.020008 22.833388 
L 161.172965 22.629719 
L 163.848283 23.448308 
L 162.892872 22.924022 
L 159.848635 21.918989 
L 164.923946 23.228091 
L 170.944829 25.106494 
L 169.747385 24.626411 
L 158.904345 21.156467 
L 156.846855 20.582777 
L 156.846855 20.582777 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 166.147892 23.314824 
L 166.1594 23.318491 
L 170.477616 24.760358 
L 173.06513 25.660245 
L 174.984524 26.377837 
L 178.324296 27.786326 
L 179.515416 28.199233 
L 178.612788 27.808961 
L 178.434945 27.69124 
L 176.987495 27.116415 
L 175.906884 26.690116 
L 173.749125 25.898378 
L 172.309736 25.381769 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 171.573359 25.123719 
L 171.002002 24.927297 
L 169.064713 24.267387 
L 168.759097 24.165242 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 224.692075 152.998871 
L 223.18181 151.842387 
L 220.830637 148.515801 
L 219.273841 145.332471 
L 216.495564 142.599134 
L 214.926224 141.911224 
L 211.987597 138.192426 
L 211.015154 135.549758 
L 210.627867 133.329744 
L 207.782764 129.120399 
L 205.918352 127.611393 
L 203.986514 126.772325 
L 202.396809 124.623182 
L 202.363494 123.805425 
L 200.982124 121.880092 
L 199.830394 121.029381 
L 197.867229 118.294386 
L 195.134708 115.329932 
L 192.832121 112.829292 
L 191.228103 112.774772 
L 191.205203 110.872808 
L 191.007487 109.659796 
L 191.009553 108.295171 
L 190.759055 107.790589 
L 190.258408 109.139368 
L 190.291174 111.742879 
L 189.89304 113.536809 
L 189.289589 114.122004 
L 187.901823 112.936032 
L 186.014025 111.30526 
L 182.369249 106.232619 
L 182.11657 106.530355 
L 184.432425 110.244536 
L 187.358104 113.862644 
L 191.238647 119.533919 
L 192.939929 121.553011 
L 194.49699 123.655407 
L 198.430524 127.849696 
L 197.916637 128.469457 
L 198.564947 130.893995 
L 203.192276 134.414423 
L 203.943908 135.210066 
L 205.809136 138.974149 
L 205.219977 139.638623 
L 206.496904 143.607889 
L 208.60882 148.279515 
L 210.067299 149.283994 
L 212.16478 150.791671 
L 214.908943 155.405573 
L 216.438525 159.068337 
L 218.576665 161.063259 
L 223.663777 164.974878 
L 224.692075 166.086416 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 224.692074 87.150624 
L 223.877551 86.951087 
L 221.440935 85.278447 
L 218.924223 84.439194 
L 216.920948 82.175743 
L 215.992836 80.714257 
L 216.406875 80.11481 
L 216.165292 79.137436 
L 215.142342 76.948425 
L 216.783981 76.908654 
L 215.619463 76.073361 
L 214.52274 75.834379 
L 212.105806 73.729512 
L 209.986329 72.153202 
L 205.4265 68.71009 
L 204.389509 66.938873 
L 200.657523 64.261995 
L 200.808551 61.844181 
L 202.483931 61.643189 
L 202.22637 60.26636 
L 203.645763 59.95826 
L 204.963096 59.14477 
L 207.075147 59.938308 
L 208.819353 60.054465 
L 210.472702 61.624998 
L 212.208441 64.040695 
L 210.242185 63.453772 
L 208.936245 63.667908 
L 210.348532 65.538341 
L 208.217259 65.067316 
L 208.93269 65.920767 
L 210.549086 66.709534 
L 213.243442 69.193112 
L 216.240251 70.382001 
L 217.316448 71.370298 
L 217.656369 72.448568 
L 218.251182 73.131822 
L 220.154858 75.003341 
L 218.91168 72.997658 
L 219.983462 72.492461 
L 221.695093 74.135864 
L 224.315875 75.985516 
L 223.233607 76.670241 
L 220.864674 75.751974 
L 222.046958 78.084621 
L 223.513367 78.410534 
L 224.304251 80.318185 
L 224.692075 80.519733 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 83.593439 13.939751 
L 84.111904 13.898132 
L 82.119353 14.079338 
L 80.420678 14.266087 
L 78.365471 14.579289 
L 78.044441 14.727484 
L 76.800794 14.841154 
L 74.964697 15.073027 
L 75.362302 15.163602 
L 73.965682 15.532062 
L 72.891788 15.65603 
L 72.393865 16.004209 
L 70.278698 16.220129 
L 70.530669 16.357989 
L 69.550952 16.595344 
L 67.936935 16.812384 
L 66.652355 16.939459 
L 66.437558 17.267512 
L 65.555514 17.573096 
L 64.505722 17.474793 
L 63.450124 17.712106 
L 64.221325 17.756914 
L 64.289884 18.078117 
L 63.088398 18.668196 
L 60.792972 19.010625 
L 60.792773 18.777693 
L 60.781025 18.455739 
L 59.65131 18.959038 
L 57.30547 19.531212 
L 59.992854 19.272068 
L 61.378094 19.170909 
L 56.620348 20.237826 
L 51.881374 21.325163 
L 47.903911 22.06575 
L 46.731009 22.224731 
L 44.694284 22.779469 
L 40.558193 24.161366 
L 36.475989 25.334678 
L 35.651574 25.499894 
L 33.636086 26.028061 
L 31.566346 26.571984 
L 29.006916 27.523758 
L 27.042793 28.463258 
L 24.704724 29.467426 
L 20.815213 30.937774 
L 19.028745 32.024027 
L 16.247361 33.396257 
L 12.978192 35.076003 
L 11.335719 35.482499 
L 12.212905 34.472364 
L 10.380181 34.901766 
L 11.159406 34.235125 
L 13.536631 32.856084 
L 16.005136 31.362953 
L 17.662776 30.528766 
L 20.372956 29.304319 
L 22.392172 28.341722 
L 24.958338 27.321737 
L 25.68611 27.005514 
L 30.009255 25.415664 
L 32.091564 24.723389 
L 33.657561 24.189675 
L 36.260681 23.317509 
L 34.466195 23.883947 
L 33.678747 24.137585 
L 32.718087 24.44843 
L 33.123415 24.300607 
L 33.504486 24.173235 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 37.781934 22.798938 
L 38.002024 22.744913 
L 38.898825 22.456371 
" style="fill:none;stroke:#000000;"/>
   </g>
   <g id="LineCollection_1">
    <path clip-path="url(#p82c2cc087c)" d="M 7.2 52.204341 
L 8.478522 50.836459 
L 9.895499 49.359324 
L 11.323033 47.911493 
L 12.760849 46.493112 
L 14.208671 45.104318 
L 15.666222 43.745243 
L 17.133225 42.416013 
L 18.609403 41.116744 
L 20.094478 39.847547 
L 21.588171 38.608527 
L 23.090205 37.39978 
L 24.6003 36.221398 
L 26.118178 35.073465 
L 27.64356 33.956058 
L 29.176168 32.869248 
L 30.715723 31.8131 
L 32.261948 30.787671 
L 33.814564 29.793013 
L 35.373294 28.829173 
L 36.937861 27.896188 
L 38.507988 26.994093 
L 40.083401 26.122914 
L 41.663823 25.282672 
L 43.248981 24.473382 
L 44.838601 23.695054 
L 46.43241 22.94769 
L 48.030137 22.231287 
L 49.631511 21.545839 
L 51.236263 20.891332 
L 52.844124 20.267745 
L 54.454827 19.675054 
L 56.068106 19.11323 
L 57.683697 18.582237 
L 59.301337 18.082034 
L 60.920763 17.612576 
L 62.541716 17.173812 
L 64.163937 16.765686 
L 65.787168 16.388138 
L 67.411155 16.041102 
L 69.035644 15.724508 
L 70.660383 15.438281 
L 72.285121 15.182341 
L 73.90961 14.956606 
L 74.165827 14.92375 
L 74.165828 14.92375 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 61.32859 170.279998 
L 61.448124 167.654402 
L 61.58057 164.890001 
L 61.719207 162.138232 
L 61.863998 159.399655 
L 62.014905 156.674822 
L 62.171888 153.964281 
L 62.334906 151.268571 
L 62.503914 148.588229 
L 62.67887 145.923782 
L 62.859727 143.275751 
L 63.046437 140.644651 
L 63.238952 138.030989 
L 63.437221 135.435266 
L 63.641194 132.857974 
L 63.850817 130.299599 
L 64.066037 127.760618 
L 64.286797 125.241501 
L 64.513041 122.74271 
L 64.744712 120.264699 
L 64.98175 117.807912 
L 65.224095 115.372788 
L 65.471686 112.959755 
L 65.72446 110.569233 
L 65.982354 108.201634 
L 66.245304 105.85736 
L 66.513243 103.536806 
L 66.786106 101.240356 
L 67.063826 98.968388 
L 67.346332 96.721267 
L 67.633558 94.499353 
L 67.925431 92.302994 
L 68.221882 90.13253 
L 68.52284 87.988293 
L 68.82823 85.870603 
L 69.137981 83.779773 
L 69.452019 81.716106 
L 69.770268 79.679896 
L 70.092655 77.671428 
L 70.419103 75.690976 
L 70.749537 73.738808 
L 71.083878 71.81518 
L 71.422051 69.92034 
L 71.763978 68.054527 
L 72.10958 66.21797 
L 72.458779 64.410888 
L 72.811496 62.633495 
L 73.167652 60.885991 
L 73.527167 59.168571 
L 73.889962 57.481419 
L 74.255956 55.824709 
L 74.62507 54.198609 
L 74.997222 52.603278 
L 75.372334 51.038863 
L 75.750323 49.505507 
L 76.131109 48.003341 
L 76.514611 46.532489 
L 76.900749 45.093066 
L 77.289441 43.685181 
L 77.680607 42.308931 
L 78.074166 40.964409 
L 78.470037 39.651696 
L 78.86814 38.370868 
L 79.268393 37.121992 
L 79.670718 35.905129 
L 80.075034 34.720329 
L 80.48126 33.567638 
L 80.889317 32.447093 
L 81.299125 31.358723 
L 81.710607 30.302551 
L 82.123681 29.278594 
L 82.53827 28.28686 
L 82.954296 27.32735 
L 83.371681 26.400061 
L 83.790347 25.50498 
L 84.210217 24.64209 
L 84.631215 23.811367 
L 85.053263 23.012781 
L 85.476287 22.246294 
L 85.900211 21.511864 
L 86.324961 20.809443 
L 86.750461 20.138976 
L 87.176637 19.500404 
L 87.603418 18.893662 
L 88.03073 18.318677 
L 88.4585 17.775374 
L 88.886658 17.263672 
L 89.315132 16.783484 
L 89.743851 16.334719 
L 90.172747 15.91728 
L 90.601749 15.531067 
L 91.03079 15.175973 
L 91.459801 14.851889 
L 91.888716 14.5587 
L 92.317467 14.296287 
L 92.745989 14.064527 
L 93.174217 13.863293 
L 93.602086 13.692454 
L 94.029532 13.551876 
L 94.456493 13.441419 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 184.780685 170.28 
L 184.582315 168.215309 
L 184.306775 165.475789 
L 184.018332 162.748461 
L 183.717061 160.033865 
L 183.403038 157.332538 
L 183.076343 154.64501 
L 182.73706 151.971808 
L 182.385274 149.313449 
L 182.021075 146.670449 
L 181.644555 144.043313 
L 181.255808 141.432545 
L 180.854933 138.838637 
L 180.44203 136.262079 
L 180.017202 133.703352 
L 179.580556 131.16293 
L 179.132199 128.641279 
L 178.672244 126.138862 
L 178.200805 123.656129 
L 177.717997 121.193527 
L 177.223939 118.751494 
L 176.718752 116.330458 
L 176.202561 113.930844 
L 175.67549 111.553064 
L 175.137668 109.197526 
L 174.589224 106.864627 
L 174.030292 104.554758 
L 173.461004 102.268302 
L 172.881498 100.00563 
L 172.291912 97.767109 
L 171.692385 95.553096 
L 171.08306 93.363939 
L 170.46408 91.199977 
L 169.835589 89.061543 
L 169.197736 86.948958 
L 168.550668 84.862537 
L 167.894535 82.802584 
L 167.229488 80.769398 
L 166.55568 78.763265 
L 165.873265 76.784465 
L 165.182398 74.833269 
L 164.483236 72.909939 
L 163.775934 71.014728 
L 163.060653 69.14788 
L 162.337552 67.309631 
L 161.60679 65.50021 
L 160.86853 63.719833 
L 160.122932 61.968713 
L 159.37016 60.247049 
L 158.610377 58.555035 
L 157.843748 56.892856 
L 157.070435 55.260688 
L 156.290606 53.658699 
L 155.504425 52.087048 
L 154.712057 50.545886 
L 153.91367 49.035357 
L 153.109429 47.555596 
L 152.299501 46.106728 
L 151.484053 44.688874 
L 150.663252 43.302144 
L 149.837264 41.946642 
L 149.006255 40.622462 
L 148.170394 39.329693 
L 147.329847 38.068415 
L 146.484779 36.838699 
L 145.635357 35.640612 
L 144.781747 34.474211 
L 143.924115 33.339547 
L 143.062625 32.236663 
L 142.197443 31.165596 
L 141.328733 30.126375 
L 140.456657 29.119023 
L 139.58138 28.143555 
L 138.703065 27.199981 
L 137.821872 26.288303 
L 136.937963 25.408518 
L 136.051498 24.560616 
L 135.162638 23.744579 
L 134.271541 22.960386 
L 133.378365 22.208008 
L 132.483267 21.48741 
L 131.586404 20.798552 
L 130.687929 20.141388 
L 129.787999 19.515866 
L 128.886765 18.921929 
L 127.984381 18.359515 
L 127.080996 17.828555 
L 126.176761 17.328977 
L 125.271825 16.860703 
L 124.366334 16.423649 
L 123.460437 16.017727 
L 122.554277 15.642846 
L 121.647998 15.298906 
L 120.741744 14.985807 
L 119.835655 14.703443 
L 118.929871 14.451702 
L 118.024532 14.230469 
L 117.119774 14.039626 
L 116.215733 13.87905 
L 115.312543 13.748613 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 224.692076 66.291258 
L 223.201108 64.645367 
L 221.633734 62.957908 
L 220.051012 61.297041 
L 218.453237 59.662975 
L 216.840707 58.055918 
L 215.213719 56.476067 
L 213.572575 54.923618 
L 211.917576 53.398755 
L 210.249027 51.90166 
L 208.567233 50.432507 
L 206.872501 48.991464 
L 205.16514 47.578694 
L 203.445459 46.194351 
L 201.71377 44.838586 
L 199.970383 43.511541 
L 198.215614 42.213354 
L 196.449775 40.944156 
L 194.673181 39.704071 
L 192.886149 38.493219 
L 191.088995 37.311711 
L 189.282036 36.159655 
L 187.46559 35.03715 
L 185.639974 33.944291 
L 183.805508 32.881167 
L 181.96251 31.84786 
L 180.111298 30.844447 
L 178.252193 29.870997 
L 176.385513 28.927577 
L 174.511576 28.014244 
L 172.630703 27.131052 
L 170.743211 26.278048 
L 168.849419 25.455274 
L 166.949646 24.662766 
L 165.044207 23.900554 
L 163.133421 23.168664 
L 161.217604 22.467114 
L 159.297071 21.795919 
L 157.372138 21.155086 
L 155.443118 20.544619 
L 153.510325 19.964516 
L 151.574071 19.414769 
L 149.634666 18.895366 
L 147.692422 18.406288 
L 145.747646 17.947513 
L 143.800647 17.519012 
L 141.85173 17.120754 
L 140.838949 16.925846 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
   </g>
   <g id="LineCollection_2">
    <path clip-path="url(#p82c2cc087c)" d="M 7.200005 142.486079 
L 10.23715 142.392424 
L 18.830049 142.147581 
L 27.637551 141.926061 
L 36.638265 141.728842 
L 45.809764 141.556811 
L 55.128695 141.410753 
L 64.570886 141.291344 
L 74.111473 141.199143 
L 83.725032 141.134583 
L 93.385724 141.097972 
L 103.067436 141.089483 
L 112.743934 141.109157 
L 122.389015 141.1569 
L 131.976657 141.232486 
L 141.481168 141.335555 
L 150.87733 141.465625 
L 160.140539 141.622088 
L 169.246934 141.804226 
L 178.173516 142.011212 
L 186.89826 142.24212 
L 195.400208 142.495937 
L 203.659559 142.771572 
L 211.65774 143.067865 
L 219.377459 143.383599 
L 224.692071 143.619406 
L 224.692075 143.619406 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 7.200003 106.312356 
L 10.629499 106.14393 
L 18.275394 105.796859 
L 26.128946 105.479012 
L 34.17201 105.191687 
L 42.385514 104.936079 
L 50.749534 104.713269 
L 59.243378 104.52421 
L 67.845685 104.369722 
L 76.534525 104.250481 
L 85.287513 104.167013 
L 94.08192 104.119687 
L 102.894795 104.108715 
L 111.70309 104.134145 
L 120.48378 104.195864 
L 129.213987 104.293597 
L 137.871106 104.426913 
L 146.432918 104.595225 
L 154.877707 104.797798 
L 163.184365 105.03376 
L 171.332495 105.302105 
L 179.302501 105.601705 
L 187.075671 105.931324 
L 194.63425 106.289623 
L 201.961502 106.675179 
L 209.041765 107.086492 
L 215.860489 107.522 
L 222.404269 107.980091 
L 224.692075 108.150347 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 7.200002 75.64741 
L 9.410948 75.484078 
L 15.715234 75.048044 
L 22.221633 74.641226 
L 28.916305 74.265101 
L 35.784601 73.921062 
L 42.811112 73.610402 
L 49.979719 73.334308 
L 57.273657 73.09385 
L 64.675581 72.889971 
L 72.167642 72.723477 
L 79.731562 72.595033 
L 87.348721 72.505156 
L 95.00024 72.454209 
L 102.667078 72.442399 
L 110.330117 72.469772 
L 117.970258 72.536219 
L 125.568513 72.64147 
L 133.106097 72.785101 
L 140.564514 72.966536 
L 147.925647 73.185052 
L 155.171838 73.439787 
L 162.285965 73.729749 
L 169.25151 74.053818 
L 176.052631 74.410765 
L 182.674214 74.799256 
L 189.101926 75.217862 
L 195.32226 75.665076 
L 201.322573 76.13932 
L 207.091109 76.638956 
L 212.617028 77.1623 
L 217.890415 77.707632 
L 222.902291 78.273209 
L 224.692072 78.487653 
L 224.692075 78.487654 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 7.200003 51.792354 
L 10.77296 51.389978 
L 15.475295 50.900569 
L 20.366358 50.435262 
L 25.436835 49.995506 
L 30.676761 49.582699 
L 36.075552 49.198173 
L 41.622021 48.843194 
L 47.304418 48.518946 
L 53.110462 48.226527 
L 59.027378 47.966939 
L 65.041947 47.741082 
L 71.140552 47.549746 
L 77.309227 47.393608 
L 83.533717 47.273223 
L 89.799531 47.18902 
L 96.092005 47.141302 
L 102.39636 47.130242 
L 108.697769 47.155878 
L 114.981415 47.218119 
L 121.232559 47.31674 
L 127.436598 47.451388 
L 133.579127 47.621583 
L 139.645999 47.826721 
L 145.623381 48.06608 
L 151.497807 48.338825 
L 157.25623 48.644015 
L 162.886067 48.980608 
L 168.375241 49.347472 
L 173.712223 49.743389 
L 178.886062 50.167067 
L 183.886417 50.617144 
L 188.703579 51.092203 
L 193.328495 51.590775 
L 197.752778 52.111353 
L 201.968723 52.652395 
L 205.96931 53.212337 
L 209.748208 53.789598 
L 213.299774 54.382592 
L 216.619044 54.98973 
L 219.701732 55.60943 
L 222.54421 56.240123 
L 224.692076 56.763995 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 7.200001 36.48091 
L 8.570684 36.14369 
L 10.910432 35.61243 
L 13.428227 35.092401 
L 16.120554 34.584852 
L 18.983441 34.091028 
L 22.012456 33.612165 
L 25.202701 33.149487 
L 28.548825 32.704199 
L 32.045018 32.277485 
L 35.685024 31.870495 
L 39.462151 31.484349 
L 43.369279 31.120124 
L 47.398874 30.778851 
L 51.543012 30.46151 
L 55.79339 30.169025 
L 60.141351 29.902256 
L 64.577911 29.661997 
L 69.093782 29.44897 
L 73.679401 29.263821 
L 78.324964 29.107114 
L 83.020451 28.979331 
L 87.755671 28.880868 
L 92.520287 28.812029 
L 97.303858 28.77303 
L 102.095874 28.763991 
L 106.885796 28.784942 
L 111.663091 28.835816 
L 116.417273 28.916454 
L 121.137938 29.026608 
L 125.814802 29.165934 
L 130.437738 29.334003 
L 134.996808 29.5303 
L 139.4823 29.754228 
L 143.884757 30.005109 
L 148.195007 30.282194 
L 152.404194 30.58466 
L 156.503799 30.911623 
L 160.485667 31.262135 
L 164.342025 31.635195 
L 168.065505 32.029754 
L 171.649153 32.444716 
L 175.086449 32.878949 
L 178.371315 33.331288 
L 181.49812 33.800542 
L 184.461695 34.285496 
L 187.257325 34.784922 
L 189.880763 35.297578 
L 192.328216 35.822218 
L 194.596355 36.357595 
L 196.682301 36.902462 
L 198.583622 37.455583 
L 200.298329 38.015731 
L 201.824861 38.581694 
L 203.162076 39.15228 
L 204.309243 39.726316 
L 204.361241 39.754887 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p82c2cc087c)" d="M 34.574558 23.820094 
L 35.67366 23.477574 
L 37.07238 23.083168 
L 38.591614 22.696153 
L 40.229193 22.317356 
L 41.982681 21.947602 
L 43.84937 21.587705 
L 45.82628 21.238471 
L 47.91017 20.900693 
L 50.097532 20.575149 
L 52.384597 20.2626 
L 54.767346 19.963782 
L 57.241508 19.679411 
L 59.802575 19.410175 
L 62.445807 19.156732 
L 65.166239 18.919708 
L 67.958698 18.699694 
L 70.817811 18.497243 
L 73.738019 18.312868 
L 76.713588 18.147041 
L 79.738631 18.000189 
L 82.807115 17.87269 
L 85.912883 17.764876 
L 89.04967 17.677029 
L 92.211118 17.60938 
L 95.390801 17.562105 
L 98.582234 17.535331 
L 101.778903 17.529126 
L 104.974273 17.543508 
L 108.161817 17.578438 
L 111.335028 17.633826 
L 114.487445 17.709523 
L 117.612665 17.805333 
L 120.704365 17.921004 
L 123.75632 18.056235 
L 126.762423 18.210674 
L 129.716694 18.383923 
L 132.613307 18.575538 
L 135.446595 18.78503 
L 138.211071 19.011871 
L 140.90144 19.255491 
L 143.51261 19.515285 
L 146.039706 19.790614 
L 148.478077 20.080809 
L 150.82331 20.385169 
L 153.071231 20.702971 
L 155.217921 21.033467 
L 157.259715 21.37589 
L 159.193209 21.729456 
L 161.015265 22.093366 
L 162.723013 22.466811 
L 164.313852 22.848972 
L 165.785452 23.239023 
L 167.13575 23.636135 
L 167.713037 23.820094 
L 167.713037 23.820094 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
   </g>
   <g id="patch_3">
    <path d="M 7.2 170.28 
L 7.2 7.2 
L 224.692076 7.2 
L 224.692076 170.28 
L 7.2 170.28 
" style="fill:none;stroke:#000000;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
  </g>
 </g>
 <defs>
  <clipPath id="p82c2cc087c">
   <path d="M 7.2 170.28 
L 7.2 7.2 
L 224.692076 7.2 
L 224.692076 170.28 
L 7.2 170.28 
"/>
  </clipPath>
 </defs>
</svg>
<pre>_PROJ4Projection(+ellps=WGS84 +a=6378169.0 +rf=295.488065897001 +h=35785831.0 +lon_0=9.5 +no_defs=True +proj=geos +type=crs +units=m +x_0=0.0 +y_0=0.0 +no_defs)</pre>



<br>

We'll create a loader function that will extract the relevant data for `lower_east_column_planned` automatically

```python
#exports
def load_scene(native_fp):
    # Reading scene and loading HRV
    scene = Scene(filenames=[native_fp], reader='seviri_l1b_native')

    # Identifying and recording lower_east_column_planned
    handler = seviri_l1b_native.NativeMSGFileHandler(native_fp, {}, None)
    scene.attrs['lower_east_column_planned'] = handler.header['15_DATA_HEADER']['ImageDescription']['PlannedCoverageHRV']['LowerEastColumnPlanned']
    
    return scene
```

<br>

We'll see how quickly this loads

```python
%%time

scene = load_scene(native_fp)
scene.load(['HRV'])
```

    Wall time: 1.18 s
    

    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\pyproj\crs\crs.py:543: UserWarning: You will likely lose important projection information when converting to a PROJ string from another format. See: https://proj.org/faq.html#what-is-the-best-format-for-describing-coordinate-reference-systems
      proj_string = self.to_proj4()
    

<br>

We can visualise what a specific band looks like

```python
fig = plt.figure(dpi=250, figsize=(10, 10))
ax = plt.axes(projection=seviri_crs)

scene['HRV'].plot.imshow(ax=ax, add_colorbar=False, cmap='magma', vmin=0, vmax=50)

ax.set_title('')
ax.coastlines(resolution='50m', alpha=0.8, color='white')
```




    <cartopy.mpl.feature_artist.FeatureArtist at 0x2948c808e50>




![png](img/nbs/output_22_1.png)


<br>

One of the benefits of having access to the underlying XArray object is that we can more easily start to do some analysis with the data, for example defining a reflectance threshold

```python
reflectance_threshold = 35

cmap = colors.ListedColormap([
    (0, 0, 0, 0), # transparent
    (251/255, 242/255, 180/255, 1) # yellow
#     (0.533, 0.808, 0.922, 1) # grey-like blue
])

# Plotting
fig = plt.figure(dpi=250, figsize=(10, 10))
ax = plt.axes(projection=seviri_crs)

scene['HRV'].plot.imshow(ax=ax, vmin=0, vmax=50, cmap='magma', add_colorbar=False)
(scene['HRV']>reflectance_threshold).plot.imshow(ax=ax, cmap=cmap, add_colorbar=False)

ax.set_title('')
ax.coastlines(resolution='50m', alpha=0.8, color='white')
```




    <cartopy.mpl.feature_artist.FeatureArtist at 0x2948c809f40>




![png](img/nbs/output_24_1.png)


<br>

We'll extract the values from the XArray object, then mask all NaN values to enable us to carry out statistical analysis

```python
HRV = scene["HRV"].values
HRV_masked = ma.masked_array(HRV, mask=xr.ufuncs.isnan(scene["HRV"]).values)

np.mean(HRV_masked)
```




    12.372444717553362



<br>

We can also visualise the full distribution.

N.b. to reduce the time it takes to calculate the best KDE fit we'll take only a sample of the data.

```python
HRV_sample = np.random.choice(HRV_masked.flatten(), 1_000_000)

# Plotting
fig, ax = plt.subplots(dpi=250)

sns.kdeplot(HRV_sample, ax=ax, fill=True)

ax.set_yticks([])
ax.set_ylabel('')
ax.set_xlabel('HRV Reflectance')
hlp.hide_spines(ax, positions=['top', 'left', 'right'])
```


![png](img/nbs/output_28_0.png)


<br>

### Evaluating Reprojection to Tranverse Mercator

Before we can resample we need to define the area we're resampling to, we'll write a constructor to help us do this

```python
#exports
def construct_area_def(scene, area_id, description,
                       proj_id, projection,
                       west, south, east, north,
                       pixel_size=None):

    # If None then will use same number of x and y points
    # HRV's resolution will be more like 4km for Europe
    if pixel_size is not None:
        width = int((east - west) / pixel_size)
        height = int((north - south) / pixel_size)
    else:
        width = scene[list(scene.keys())[0]['name']].x.values.shape[0]
        height = scene[list(scene.keys())[0]['name']].y.values.shape[0]

    area_extent = (west, south, east, north)

    area_def = AreaDefinition(area_id, description,
                              proj_id, projection,
                              width, height, area_extent)

    return area_def

def construct_TM_area_def(scene):
    meters_per_pixel = 4000 
    west, south, east, north = (-3090000, 1690000, 4390000, 9014000)

    area_id = 'TM'
    description = 'Transverse Mercator' 
    proj_id = 'TM'

    projection = {
        'ellps': 'WGS84',
        'proj': 'tmerc',  # Transverse Mercator
        'units': 'm'  # meters
    }

    tm_area_def = construct_area_def(scene, area_id, description,
                                     proj_id, projection,
                                     west, south, east, north,
                                     meters_per_pixel)
    
    return tm_area_def
```

```python
tm_area_def = construct_TM_area_def(scene)

tm_area_def.to_cartopy_crs()
```

    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\pyproj\crs\crs.py:543: UserWarning: You will likely lose important projection information when converting to a PROJ string from another format. See: https://proj.org/faq.html#what-is-the-best-format-for-describing-coordinate-reference-systems
      proj_string = self.to_proj4()
    




<?xml version="1.0" encoding="utf-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
  "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<!-- Created with matplotlib (https://matplotlib.org/) -->
<svg height="177.48pt" version="1.1" viewBox="0 0 180.953577 177.48" width="180.953577pt" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
 <metadata>
  <rdf:RDF xmlns:cc="http://creativecommons.org/ns#" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
   <cc:Work>
    <dc:type rdf:resource="http://purl.org/dc/dcmitype/StillImage"/>
    <dc:date>2020-12-16T23:15:54.380829</dc:date>
    <dc:format>image/svg+xml</dc:format>
    <dc:creator>
     <cc:Agent>
      <dc:title>Matplotlib v3.3.2, https://matplotlib.org/</dc:title>
     </cc:Agent>
    </dc:creator>
   </cc:Work>
  </rdf:RDF>
 </metadata>
 <defs>
  <style type="text/css">*{stroke-linecap:butt;stroke-linejoin:round;}</style>
 </defs>
 <g id="figure_1">
  <g id="patch_1">
   <path d="M 0 177.48 
L 180.953577 177.48 
L 180.953577 0 
L 0 0 
z
" style="fill:none;"/>
  </g>
  <g id="axes_1">
   <g id="patch_2">
    <path d="M 7.2 170.28 
L 7.2 7.2 
L 173.753577 7.2 
L 173.753577 170.28 
L 7.2 170.28 
" style="fill:#ffffff;"/>
   </g>
   <g id="PathCollection_1">
    <path clip-path="url(#p6af88d2914)" d="M 66.930454 74.556112 
L 67.021396 76.344429 
L 65.688257 78.452078 
L 62.826144 79.625229 
L 60.703977 78.97381 
L 62.274018 76.559724 
L 61.837582 73.947479 
L 64.052022 72.267211 
L 65.260018 71.236164 
L 66.457284 71.256451 
L 67.850921 72.921704 
L 66.930454 74.556112 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 25.333635 7.200001 
L 25.182856 7.920256 
L 25.352537 9.591068 
L 23.879482 12.017921 
L 24.528806 13.530687 
L 22.752753 14.179189 
L 19.098542 13.997614 
L 19.470164 12.238588 
L 21.026591 9.563451 
L 20.1458 8.436854 
L 18.684312 8.92989 
L 17.746363 10.648859 
L 17.305142 12.587796 
L 16.880676 13.33788 
L 14.860925 15.261478 
L 12.931954 15.796639 
L 12.87667 14.057505 
L 13.567428 10.35271 
L 12.135252 12.900751 
L 11.215705 14.803476 
L 10.450685 15.418129 
L 9.9653 11.789727 
L 10.409136 8.667682 
L 10.972934 7.200001 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 143.752045 108.614297 
L 145.28896 108.068825 
L 146.906046 106.540445 
L 145.911758 108.2067 
L 146.402152 108.814214 
L 144.606726 110.696584 
L 143.435193 110.743796 
L 142.56053 109.886245 
L 143.67871 109.351548 
L 143.752045 108.614297 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 129.835083 113.450056 
L 129.78681 114.276401 
L 126.826599 115.316719 
L 126.736659 114.891604 
L 124.066349 115.053099 
L 124.176293 113.879008 
L 125.529929 114.440323 
L 127.109786 113.865519 
L 128.709216 113.619626 
L 128.782224 114.076747 
L 129.835083 113.450056 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 92.354392 106.237849 
L 93.212362 105.372919 
L 94.533721 107.008227 
L 94.625862 110.315947 
L 93.732597 110.251738 
L 93.027396 111.157846 
L 92.233047 110.570537 
L 91.873883 107.589327 
L 91.309279 106.214328 
L 92.354392 106.237849 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 91.983574 101.951716 
L 93.054738 100.878968 
L 93.605623 102.967068 
L 93.204125 104.947853 
L 92.305611 104.532344 
L 91.707928 102.90965 
L 91.983574 101.951716 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 93.089423 67.850203 
L 93.756866 69.000854 
L 93.272611 71.143936 
L 91.562027 70.001829 
L 91.203765 69.010932 
L 93.089423 67.850203 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 70.546001 63.170746 
L 72.118015 63.046221 
L 70.574877 65.653612 
L 71.947203 65.385429 
L 73.401386 65.452198 
L 72.989333 67.461315 
L 71.668281 69.634459 
L 73.100419 69.846698 
L 74.400019 73.063141 
L 75.381998 73.472052 
L 76.278 76.297314 
L 76.707258 77.274217 
L 78.532297 77.718969 
L 78.383724 79.307487 
L 77.616977 80.047934 
L 78.255593 81.318936 
L 76.868075 82.635198 
L 74.766678 82.608917 
L 72.070096 83.230621 
L 71.353086 82.716103 
L 70.256693 83.829935 
L 68.804768 83.468482 
L 67.62539 84.340133 
L 66.815128 83.783353 
L 69.298601 81.341198 
L 70.715662 80.879563 
L 68.314717 80.325609 
L 67.949896 79.310071 
L 69.591561 78.647083 
L 68.848702 77.261924 
L 69.237522 75.658845 
L 71.425114 76.001612 
L 71.70245 74.572628 
L 70.782801 72.965676 
L 69.0679 72.424649 
L 68.775689 71.73053 
L 69.36716 70.660399 
L 68.954834 69.945814 
L 68.109478 71.058177 
L 68.221745 68.666238 
L 67.640001 67.3449 
L 68.351339 64.829876 
L 69.527018 62.903619 
L 70.546001 63.170746 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 61.703077 42.052639 
L 61.101053 43.569604 
L 61.862249 45.491022 
L 60.084729 47.03567 
L 56.587766 47.885771 
L 55.535615 48.052029 
L 54.24141 47.258554 
L 51.57848 45.480065 
L 52.977034 44.810229 
L 51.22035 42.823357 
L 53.156298 43.047612 
L 53.370357 42.339585 
L 51.541319 40.995012 
L 52.801948 39.737178 
L 54.363848 39.942541 
L 55.286808 42.080761 
L 57.18656 41.269188 
L 58.200606 42.318209 
L 60.141998 41.477378 
L 61.703077 42.052639 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 110.280962 7.200002 
L 110.949073 8.235193 
L 112.287848 10.619936 
L 114.113276 11.738376 
L 116.76246 11.638386 
L 116.664901 12.184206 
L 114.744783 14.235849 
L 113.70697 13.707614 
L 112.235171 14.431225 
L 111.098184 13.632989 
L 111.184424 12.662797 
L 110.081532 11.807957 
L 109.25688 9.315598 
L 108.610076 9.652521 
L 107.867097 7.2 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 84.293788 7.2 
L 84.919785 7.477486 
L 84.95672 7.200001 
L 84.95672 7.2 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 87.352741 7.2 
L 87.352952 7.200073 
L 87.360109 8.694803 
L 86.272415 9.503243 
L 84.871488 9.601517 
L 84.565656 8.982002 
L 83.955646 9.144758 
L 83.16199 8.19815 
L 84.087698 7.200001 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 88.574216 12.707719 
L 87.878113 14.114062 
L 86.785188 13.916274 
L 86.898687 13.186233 
L 86.326352 12.550821 
L 87.114755 11.704756 
L 87.661212 12.491269 
L 88.574216 12.707719 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 82.679838 10.005644 
L 82.619085 9.140911 
L 83.203275 8.883068 
L 83.989492 9.547933 
L 86.042626 10.788388 
L 85.227452 12.126683 
L 85.53671 13.955926 
L 85.236558 14.544779 
L 85.587322 16.595655 
L 84.954964 16.879212 
L 83.415423 15.708638 
L 83.675355 14.733159 
L 82.744266 14.218007 
L 81.359027 12.364029 
L 80.644687 10.516118 
L 81.638156 9.402451 
L 82.071867 10.195239 
L 82.679838 10.005644 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 7.479339 7.2 
L 7.200001 7.781751 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 7.200001 19.293135 
L 7.527792 19.331336 
L 7.200001 19.753975 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 104.889934 111.541174 
L 106.345909 111.073992 
L 105.964373 113.155621 
L 106.39003 113.878984 
L 106.178625 115.230381 
L 104.497563 114.532637 
L 103.441683 114.417323 
L 100.490793 113.544775 
L 100.592813 112.231073 
L 102.925668 112.126746 
L 104.889934 111.541174 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 142.026347 81.375988 
L 144.500757 81.201979 
L 147.893517 81.978365 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 37.502367 159.053493 
L 37.333377 157.760119 
L 37.699855 156.515346 
L 37.203415 155.242752 
L 36.059013 154.007665 
L 36.526652 151.775114 
L 37.515923 151.185134 
L 38.437179 149.948903 
L 38.383035 149.073145 
L 39.392969 147.377355 
L 40.870038 145.917537 
L 41.694124 145.598587 
L 42.468257 144.196085 
L 42.674678 142.860034 
L 43.673761 141.397934 
L 45.283056 140.650783 
L 46.984486 138.2453 
L 48.250287 137.372867 
L 50.338873 137.305886 
L 52.237502 135.766628 
L 53.396123 135.203023 
L 55.392467 133.283465 
L 55.123378 130.144271 
L 56.12125 128.078368 
L 56.519379 126.795604 
L 58.009974 125.234519 
L 60.184401 124.266626 
L 61.78942 123.351664 
L 63.32492 120.87774 
L 64.055537 119.397146 
L 65.539478 119.493523 
L 66.705934 120.604062 
L 68.639048 120.513315 
L 70.720799 121.125104 
L 71.60153 121.172702 
L 73.568358 119.855755 
L 75.747309 119.44116 
L 77.011291 118.419262 
L 78.925992 117.647509 
L 82.288573 117.125454 
L 85.568308 116.787397 
L 86.591304 117.101495 
L 88.403091 116.012182 
L 90.520465 115.840682 
L 91.374295 116.352695 
L 92.722152 116.086218 
L 94.788139 114.879715 
L 96.206998 115.031278 
L 96.2844 116.293294 
L 97.872748 115.189321 
L 98.070655 115.64976 
L 97.209364 116.981175 
L 97.322713 118.131398 
L 98.091909 118.670661 
L 98.062121 120.851219 
L 96.840536 122.242403 
L 97.370648 123.556784 
L 98.448647 123.487349 
L 99.099659 124.613605 
L 99.937777 124.915628 
L 102.504808 125.486082 
L 103.362685 125.161786 
L 105.179143 125.349315 
L 108.15031 126.074363 
L 109.479203 128.157715 
L 111.490714 128.353844 
L 114.722337 128.917803 
L 117.271163 129.778094 
L 118.226512 128.92969 
L 119.0621 127.568522 
L 118.187911 125.721447 
L 118.627717 124.358564 
L 119.937302 122.860072 
L 121.340364 122.203911 
L 124.363829 122.081566 
L 125.362414 123.053706 
L 126.174016 122.877692 
L 126.969583 123.151753 
L 129.21424 122.930097 
L 129.958291 123.647251 
L 132.88411 122.859665 
L 138.787354 123.24739 
L 140.280233 121.917828 
L 140.979959 120.892076 
L 142.891638 120.065455 
L 144.597054 119.887549 
L 145.634588 120.995777 
L 145.882085 119.963041 
L 147.910359 119.984244 
L 149.742244 119.529179 
L 150.633638 118.467157 
L 150.983898 117.355375 
L 150.774535 117.260822 
L 150.921076 115.804069 
L 150.629613 113.641349 
L 150.743311 112.802518 
L 150.689687 110.356817 
L 151.02539 107.968803 
L 149.995865 106.061632 
L 150.06147 104.775866 
L 148.78968 103.934492 
L 149.159682 102.62723 
L 147.989688 103.391894 
L 146.010796 103.529089 
L 145.187088 105.588605 
L 142.135589 107.103692 
L 139.945509 106.353125 
L 137.695372 107.069259 
L 137.591221 108.296109 
L 136.262906 109.089047 
L 133.817912 108.393285 
L 131.59561 109.155542 
L 129.605387 107.009792 
L 127.710871 106.060436 
L 128.06128 103.801585 
L 126.443659 102.991432 
L 127.808334 99.902051 
L 130.718851 98.783365 
L 130.826308 96.59449 
L 134.547792 95.547261 
L 136.1402 93.008125 
L 137.998763 91.370648 
L 141.030335 89.91904 
L 145.1192 90.095263 
L 148.300434 89.688189 
L 150.286435 88.143594 
L 152.021438 87.475658 
L 153.479714 84.929033 
L 153.132921 83.719911 
L 151.68719 82.327303 
L 150.121909 82.047244 
L 148.972693 82.345708 
L 147.893517 81.978365 
L 144.500757 81.201979 
L 142.026347 81.375988 
L 139.787799 80.749866 
L 140.816884 79.701529 
L 141.090068 76.928249 
L 139.637802 76.507066 
L 141.472666 74.155311 
L 141.122966 73.66606 
L 139.903165 74.893911 
L 138.718615 75.818322 
L 138.057865 77.187372 
L 136.585571 78.136381 
L 135.618099 79.767101 
L 136.47135 81.217065 
L 137.596345 81.380377 
L 139.253531 80.337623 
L 139.372813 81.372746 
L 137.707381 82.751284 
L 136.030828 85.302251 
L 134.826986 85.258508 
L 134.676417 83.934864 
L 132.466702 84.089523 
L 132.557191 83.485401 
L 133.808809 81.918743 
L 133.056615 81.596608 
L 130.160354 82.188282 
L 129.640009 81.336467 
L 128.224842 82.316717 
L 128.17146 83.922789 
L 127.591103 86.274802 
L 127.886095 86.890151 
L 127.251249 87.753947 
L 126.63121 87.731393 
L 127.27274 90.889627 
L 126.718898 92.25927 
L 126.692377 94.270514 
L 127.791448 95.47597 
L 128.352801 96.354144 
L 130.279509 96.569085 
L 130.158645 97.305172 
L 127.961861 98.234027 
L 127.414537 99.281327 
L 126.260164 101.151155 
L 125.286508 100.177938 
L 125.147834 99.65406 
L 123.969881 99.947775 
L 122.908815 100.016311 
L 120.80409 101.343836 
L 122.534991 102.36514 
L 121.675852 103.054854 
L 120.60445 103.358646 
L 119.236921 102.350664 
L 119.027083 102.995923 
L 119.858976 104.373277 
L 121.149099 105.285221 
L 120.560852 106.033167 
L 121.966459 106.895311 
L 123.145435 107.356234 
L 123.565399 108.770278 
L 121.539213 108.597625 
L 122.46671 109.726177 
L 121.258607 110.320983 
L 122.592801 112.350627 
L 121.245898 112.720802 
L 119.315406 112.028068 
L 118.082709 110.196815 
L 117.336651 108.610866 
L 114.960736 106.451561 
L 114.663405 105.770935 
L 114.294581 105.672686 
L 114.128318 105.132561 
L 112.874322 104.539689 
L 112.437867 103.390442 
L 112.204888 101.655584 
L 112.278642 100.823929 
L 111.873497 100.506024 
L 111.440141 100.404273 
L 110.734234 99.713286 
L 109.826783 99.401714 
L 107.903839 98.862444 
L 106.66027 98.193094 
L 104.857184 97.792418 
L 102.9984 96.261485 
L 103.321932 96.008122 
L 102.317327 95.120721 
L 102.127465 94.291933 
L 100.930696 94.109379 
L 100.582801 95.272619 
L 99.920475 94.534465 
L 99.866203 93.618278 
L 100.213468 93.326117 
L 98.77152 93.203662 
L 97.500808 94.309236 
L 97.788294 95.522165 
L 97.680819 96.260284 
L 98.457041 97.430672 
L 100.33776 98.407655 
L 101.593303 100.302766 
L 103.990138 101.942224 
L 105.438675 101.657117 
L 105.996267 102.118915 
L 105.568192 102.713287 
L 108.952905 103.758319 
L 110.852901 104.713222 
L 111.150717 105.135846 
L 110.983216 106.104706 
L 109.677221 105.164037 
L 107.940622 105.088257 
L 107.447056 106.852908 
L 109.017119 107.507884 
L 109.039845 108.853323 
L 108.253842 109.155871 
L 107.596439 111.485534 
L 106.806943 111.821805 
L 106.68261 111.058371 
L 106.84697 109.651624 
L 107.174565 109.043339 
L 105.348951 106.586893 
L 104.502312 106.415934 
L 103.757097 105.439884 
L 102.466985 105.191518 
L 101.491914 104.325437 
L 100.080956 104.379888 
L 98.459439 103.474384 
L 96.546657 102.094696 
L 95.118016 100.829412 
L 94.247721 98.452668 
L 93.323365 98.273221 
L 91.781712 97.62165 
L 90.999014 98.043045 
L 90.080907 99.294143 
L 89.351915 99.544747 
L 87.83513 101.080468 
L 84.223991 100.647733 
L 81.626078 101.57125 
L 81.471354 103.06848 
L 81.620535 104.501924 
L 79.909096 106.201574 
L 77.521654 106.764174 
L 77.361444 107.597505 
L 76.206049 108.973687 
L 75.468306 110.985272 
L 76.219003 112.398589 
L 75.093622 113.499177 
L 74.660556 115.103083 
L 73.169253 115.578851 
L 71.730949 117.452756 
L 67.305648 117.293138 
L 66.012462 118.105614 
L 65.195993 118.999347 
L 64.223974 118.737145 
L 63.533599 117.854456 
L 63.063218 116.39281 
L 61.237978 115.872534 
L 60.387184 116.450839 
L 59.367962 116.012525 
L 58.316022 116.191607 
L 58.800989 114.280163 
L 58.761745 112.738595 
L 57.910466 112.42352 
L 57.541959 111.436024 
L 57.866645 109.82949 
L 58.724566 109.009171 
L 58.961867 108.021472 
L 59.507598 106.571881 
L 59.57226 105.518279 
L 59.291165 104.5903 
L 59.297579 103.746826 
L 59.578254 101.993183 
L 58.951774 100.837181 
L 61.693139 99.31872 
L 63.852016 99.9615 
L 66.268818 100.123291 
L 68.160879 100.65852 
L 69.664202 100.599506 
L 72.575121 100.776206 
L 73.532375 99.310665 
L 73.945001 94.385964 
L 72.238876 91.750275 
L 71.039081 90.456498 
L 68.534461 89.381444 
L 68.475157 87.566745 
L 70.623582 87.138269 
L 73.351146 87.863889 
L 72.902692 85.048998 
L 74.40268 86.140798 
L 78.135091 84.200949 
L 78.568271 82.161906 
L 81.145805 81.085344 
L 81.909294 80.366045 
L 83.021472 76.644912 
L 84.97154 75.455456 
L 86.20352 75.413243 
L 86.439066 74.861202 
L 87.650109 74.596456 
L 87.984343 75.11036 
L 88.831706 73.77345 
L 88.386073 72.888809 
L 88.148643 71.496487 
L 87.411121 70.194749 
L 87.070975 67.674867 
L 87.217454 66.981065 
L 87.512638 66.192496 
L 88.673904 65.882466 
L 89.045437 65.135767 
L 90.004137 64.27978 
L 90.158232 65.552861 
L 89.884374 66.416177 
L 90.151698 67.081255 
L 90.947818 67.338322 
L 90.760239 68.320184 
L 90.309383 68.115788 
L 89.569684 70.036393 
L 90.123073 71.182779 
L 90.284586 72.132181 
L 91.821197 72.488945 
L 91.942945 73.365799 
L 93.338557 72.661972 
L 94.027993 71.845865 
L 95.834654 72.509744 
L 96.675231 73.154906 
L 97.514748 72.228292 
L 101.068043 69.320613 
L 102.582627 69.356288 
L 102.852392 69.920125 
L 104.22716 69.564289 
L 104.234629 68.402904 
L 105.910678 67.028466 
L 104.972525 65.093522 
L 104.436648 63.265233 
L 104.584496 61.544903 
L 105.517355 60.31212 
L 107.17802 61.732242 
L 108.204481 61.301191 
L 107.366832 57.926861 
L 107.022533 58.401673 
L 105.915325 57.853753 
L 105.303096 56.527957 
L 106.58104 55.28284 
L 107.948272 54.329074 
L 109.404009 54.163783 
L 110.596513 53.547141 
L 111.322212 51.645208 
L 109.601214 51.11645 
L 107.600268 52.231668 
L 105.902122 53.93767 
L 104.159759 55.170669 
L 103.005095 54.121231 
L 101.596194 53.735007 
L 101.044206 51.298873 
L 99.76301 49.337885 
L 99.799685 47.773932 
L 100.221269 45.940373 
L 101.544557 42.482328 
L 101.994699 41.718491 
L 101.471998 40.781655 
L 99.684618 40.23748 
L 98.291327 41.54238 
L 97.931916 43.536707 
L 98.601604 44.932405 
L 97.586055 47.406688 
L 96.119724 50.107648 
L 96.240367 53.737863 
L 97.545426 55.211311 
L 99.073931 56.22116 
L 98.724455 59.268923 
L 97.568522 60.185851 
L 98.107802 64.394429 
L 97.895954 66.86407 
L 96.187372 67.003361 
L 95.818738 69.111404 
L 94.223876 69.544461 
L 93.348845 67.305503 
L 91.717926 64.722232 
L 90.133846 61.415392 
L 89.041898 60.043374 
L 86.925848 63.249431 
L 85.253765 64.02859 
L 83.335471 62.939662 
L 82.661123 60.31633 
L 81.829354 54.632829 
L 82.756016 52.950438 
L 85.482981 50.54196 
L 87.234174 47.680114 
L 88.491534 43.919299 
L 89.751444 38.708612 
L 90.785144 36.500336 
L 92.221993 32.797484 
L 93.634572 31.14778 
L 94.984568 30.771745 
L 95.358058 28.367587 
L 96.762233 27.857156 
L 97.858036 26.766087 
L 101.011293 27.157896 
L 100.384135 28.280304 
L 101.985072 29.20496 
L 102.327986 27.992171 
L 104.360958 28.546405 
L 106.757453 27.692035 
L 111.314339 27.914166 
L 112.65745 28.402491 
L 113.779753 29.693632 
L 113.719194 31.458655 
L 112.766984 33.023174 
L 107.885245 33.955553 
L 107.421536 34.616623 
L 109.826391 35.325013 
L 111.975297 38.462125 
L 113.60621 38.341465 
L 114.711185 38.40632 
L 114.130072 37.292691 
L 112.959843 36.751365 
L 112.986394 35.558131 
L 116.072783 35.316074 
L 116.43532 34.251187 
L 114.707885 33.194022 
L 115.058683 29.698137 
L 115.882889 29.198747 
L 117.20981 29.242849 
L 116.476582 27.515267 
L 114.82599 26.927336 
L 114.130859 25.445962 
L 112.57611 24.734922 
L 115.080847 23.541323 
L 116.487324 24.187501 
L 115.811636 25.281696 
L 116.801618 26.324129 
L 117.993777 26.413685 
L 118.722326 24.901146 
L 117.710531 23.569815 
L 118.646905 17.244733 
L 119.182092 16.763868 
L 119.846565 18.476359 
L 120.837764 17.689479 
L 120.504661 16.605904 
L 121.491187 15.101837 
L 121.37663 13.326946 
L 123.291114 13.25311 
L 122.372698 11.424116 
L 120.575673 11.473681 
L 120.085896 10.624666 
L 122.091478 8.684648 
L 123.404635 7.925733 
L 125.130045 7.200001 
L 125.130045 7.200001 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 173.753576 143.64529 
L 171.91722 142.079185 
L 170.992484 140.146299 
L 170.576812 138.389811 
L 168.039047 135.68071 
L 166.452738 134.961281 
L 164.860514 134.821737 
L 163.49891 133.475226 
L 163.435046 132.795052 
L 162.257205 131.572306 
L 161.317026 131.189553 
L 159.665738 129.445254 
L 157.433107 127.721005 
L 155.573282 126.255914 
L 154.377471 126.663243 
L 154.260305 125.052713 
L 154.048735 124.070583 
L 153.975446 122.893715 
L 153.762433 122.53072 
L 153.467762 123.839791 
L 153.631393 126.050541 
L 153.4314 127.671262 
L 153.01799 128.321833 
L 151.943318 127.699048 
L 150.490788 126.83204 
L 147.612192 123.524875 
L 147.449851 123.841173 
L 149.298773 126.355568 
L 151.595181 128.610386 
L 154.720377 132.288904 
L 156.082297 133.507852 
L 157.348984 134.827922 
L 160.539041 137.238784 
L 160.168725 137.875444 
L 160.770767 139.675415 
L 164.547224 141.354856 
L 165.179163 141.808054 
L 166.82695 144.38515 
L 166.374469 145.068964 
L 167.553715 147.942771 
L 169.44725 151.171382 
L 170.694153 151.632924 
L 172.511408 152.346104 
L 173.753575 153.861494 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 33.563398 170.28 
L 33.147973 169.921188 
L 34.285166 169.521901 
L 35.63328 167.82141 
L 36.327661 166.557505 
L 36.229434 165.16408 
L 37.022707 163.965174 
L 37.53737 161.593663 
L 37.502367 159.053493 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 173.753576 70.041126 
L 173.281026 69.37755 
L 173.753577 68.374763 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 173.753577 84.648152 
L 172.608263 83.820144 
L 171.67443 82.625301 
L 171.963411 81.533351 
L 171.657597 80.429277 
L 170.564479 78.291199 
L 171.939541 76.921952 
L 170.86718 76.71558 
L 169.925729 77.263087 
L 167.699686 76.269577 
L 165.803651 75.730222 
L 161.794353 74.381284 
L 160.776765 72.675275 
L 157.595096 71.6116 
L 157.395863 68.003925 
L 158.633483 66.4012 
L 158.246447 64.488092 
L 159.275878 62.809935 
L 160.155445 60.311408 
L 161.8947 59.679026 
L 163.261942 58.178776 
L 164.799338 59.241441 
L 166.539213 61.60443 
L 164.890734 62.531481 
L 163.892307 64.082682 
L 165.267941 65.764947 
L 163.520012 66.898608 
L 164.197256 67.589679 
L 165.58505 67.397428 
L 168.081366 68.802079 
L 170.700986 67.886804 
L 171.727177 68.39645 
L 172.149724 69.722132 
L 172.737398 70.201022 
L 173.753576 70.766558 
L 173.753577 70.766559 
" style="fill:none;stroke:#000000;"/>
    <path clip-path="url(#p6af88d2914)" d="M 70.26554 7.200001 
L 69.459458 7.703061 
L 69.074811 8.186937 
L 67.661748 8.175397 
L 68.558278 8.603632 
L 67.503777 10.169551 
L 66.61882 11.57405 
L 65.717366 14.196199 
L 65.820235 15.948178 
L 64.955352 15.762624 
L 63.801725 16.17625 
L 64.363678 17.79154 
L 63.805636 19.84122 
L 63.08601 19.831077 
L 63.177914 22.138689 
L 61.700974 21.798067 
L 62.097403 23.02711 
L 61.581519 23.777588 
L 60.502117 23.784178 
L 59.570754 23.415769 
L 59.786547 25.330663 
L 59.390544 26.371458 
L 58.396734 24.844502 
L 57.772902 25.304104 
L 58.478392 26.285498 
L 58.864633 28.086616 
L 58.434916 30.079768 
L 56.874897 29.984649 
L 56.657244 28.845904 
L 56.319232 27.123579 
L 55.876172 28.79091 
L 54.442074 29.58853 
L 56.451328 30.588334 
L 57.490207 31.151155 
L 54.49991 32.281956 
L 51.426979 33.045204 
L 48.598199 32.558225 
L 47.685444 32.05706 
L 46.340686 32.401401 
L 43.772147 33.977727 
L 40.947632 34.278917 
L 40.291913 33.960199 
L 38.784618 33.667641 
L 37.197265 33.240648 
L 35.487995 33.93959 
L 34.364638 35.362639 
L 32.861499 36.350009 
L 30.055697 36.769117 
L 29.145357 38.662706 
L 27.343188 39.993643 
L 25.203253 41.566524 
L 23.672674 40.492109 
L 23.66128 37.590509 
L 21.78806 35.800458 
L 21.936111 33.884645 
L 23.221932 31.479378 
L 24.241779 27.71308 
L 25.15224 26.135692 
L 26.850428 24.477064 
L 27.760098 21.771241 
L 29.500204 20.850992 
L 29.793974 19.742017 
L 32.78234 18.601056 
L 34.438693 19.082302 
L 35.49909 18.655899 
L 37.147757 17.479545 
L 35.790301 17.29311 
L 35.179751 17.157591 
L 34.377753 16.749713 
L 34.362854 15.348814 
L 35.731199 14.244081 
L 37.029718 13.685277 
L 37.539163 14.284197 
L 38.106839 16.063322 
L 38.451825 13.907536 
L 38.694156 12.786739 
L 37.915645 12.353512 
L 38.031352 11.427577 
L 40.483955 10.731169 
L 40.980604 9.795195 
L 42.082791 8.29467 
L 42.934371 7.200001 
" style="fill:none;stroke:#000000;"/>
   </g>
   <g id="LineCollection_1">
    <path clip-path="url(#p6af88d2914)" d="M 7.200001 27.085971 
L 7.389138 26.958394 
L 8.989152 25.88351 
L 10.58498 24.819033 
L 12.176683 23.764619 
L 13.764329 22.719938 
L 15.347987 21.684662 
L 16.927732 20.658476 
L 18.503638 19.641067 
L 20.075784 18.632132 
L 21.64425 17.631373 
L 23.209121 16.6385 
L 24.77048 15.653228 
L 26.328415 14.675277 
L 27.883015 13.704374 
L 29.434371 12.740251 
L 30.982573 11.782646 
L 32.527715 10.8313 
L 34.069892 9.885961 
L 35.6092 8.946381 
L 37.145734 8.012314 
L 38.486914 7.2 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 7.200001 76.559062 
L 8.126852 75.024721 
L 9.128558 73.384858 
L 10.135489 71.754941 
L 11.14745 70.134828 
L 12.164249 68.524373 
L 13.185704 66.923428 
L 14.211634 65.331842 
L 15.241867 63.749463 
L 16.276234 62.176136 
L 17.314572 60.611706 
L 18.356723 59.056015 
L 19.402534 57.508904 
L 20.451858 55.970212 
L 21.504552 54.43978 
L 22.560477 52.917443 
L 23.619499 51.40304 
L 24.681489 49.896405 
L 25.746322 48.397375 
L 26.813878 46.905784 
L 27.884039 45.421467 
L 28.956694 43.944258 
L 30.031734 42.473989 
L 31.109054 41.010496 
L 32.188553 39.55361 
L 33.270134 38.103166 
L 34.353702 36.658995 
L 35.439167 35.220933 
L 36.526441 33.78881 
L 37.615441 32.362462 
L 38.706085 30.941721 
L 39.798295 29.526421 
L 40.891996 28.116396 
L 41.987114 26.711479 
L 43.08358 25.311506 
L 44.181327 23.91631 
L 45.280288 22.525727 
L 46.380403 21.139592 
L 47.481609 19.75774 
L 48.58385 18.380007 
L 49.687068 17.006229 
L 50.791209 15.636243 
L 51.896221 14.269886 
L 53.002053 12.906995 
L 54.108656 11.547407 
L 55.215983 10.190961 
L 56.323988 8.837495 
L 57.432627 7.486847 
L 57.668439 7.200001 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 27.094125 170.279999 
L 27.234125 168.763753 
L 27.408396 166.953344 
L 27.590245 165.144332 
L 27.779625 163.336772 
L 27.976492 161.530715 
L 28.180798 159.726212 
L 28.392492 157.92331 
L 28.611525 156.12206 
L 28.837843 154.322507 
L 29.071394 152.524696 
L 29.312122 150.728672 
L 29.559971 148.934477 
L 29.814883 147.142153 
L 30.076799 145.35174 
L 30.345659 143.563275 
L 30.621401 141.776797 
L 30.903964 139.99234 
L 31.193282 138.209938 
L 31.489292 136.429625 
L 31.791928 134.65143 
L 32.101124 132.875384 
L 32.416811 131.101515 
L 32.738921 129.329849 
L 33.067385 127.560411 
L 33.402133 125.793224 
L 33.743093 124.02831 
L 34.090195 122.265689 
L 34.443366 120.505381 
L 34.802533 118.747401 
L 35.167623 116.991766 
L 35.538561 115.23849 
L 35.915273 113.487586 
L 36.297684 111.739063 
L 36.685718 109.992932 
L 37.0793 108.249201 
L 37.478352 106.507875 
L 37.882798 104.76896 
L 38.292562 103.032459 
L 38.707565 101.298374 
L 39.127731 99.566705 
L 39.552981 97.837451 
L 39.983237 96.110611 
L 40.418421 94.386179 
L 40.858456 92.66415 
L 41.303262 90.944519 
L 41.75276 89.227276 
L 42.206873 87.512412 
L 42.665522 85.799917 
L 43.128628 84.089778 
L 43.596112 82.381981 
L 44.067896 80.676513 
L 44.543901 78.973357 
L 45.024049 77.272495 
L 45.50826 75.57391 
L 45.996458 73.877582 
L 46.488563 72.18349 
L 46.984497 70.491611 
L 47.484183 68.801923 
L 47.987542 67.114402 
L 48.494497 65.429022 
L 49.004971 63.745756 
L 49.518885 62.064578 
L 50.036164 60.385459 
L 50.55673 58.708369 
L 51.080505 57.033279 
L 51.607415 55.360155 
L 52.137382 53.688967 
L 52.670331 52.019681 
L 53.206185 50.352263 
L 53.74487 48.686678 
L 54.28631 47.02289 
L 54.830429 45.360863 
L 55.377153 43.700558 
L 55.926408 42.041939 
L 56.478118 40.384965 
L 57.032211 38.729598 
L 57.588612 37.075797 
L 58.147247 35.423521 
L 58.708043 33.772729 
L 59.270928 32.123377 
L 59.835828 30.475424 
L 60.402671 28.828825 
L 60.971383 27.183537 
L 61.541894 25.539515 
L 62.114132 23.896713 
L 62.688023 22.255087 
L 63.263498 20.61459 
L 63.840484 18.975176 
L 64.418911 17.336798 
L 64.998707 15.699407 
L 65.579803 14.062957 
L 66.162127 12.427399 
L 66.745609 10.792685 
L 67.330179 9.158766 
L 67.915766 7.525593 
L 68.032656 7.200001 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 76.00355 170.28 
L 76.00355 169.314551 
L 76.00355 167.598104 
L 76.00355 165.881543 
L 76.00355 164.164865 
L 76.00355 162.448065 
L 76.00355 160.73114 
L 76.00355 159.014084 
L 76.00355 157.296894 
L 76.00355 155.579566 
L 76.00355 153.862097 
L 76.00355 152.144482 
L 76.00355 150.426717 
L 76.00355 148.708801 
L 76.00355 146.990727 
L 76.00355 145.272495 
L 76.00355 143.554099 
L 76.00355 141.835538 
L 76.00355 140.116807 
L 76.00355 138.397904 
L 76.00355 136.678826 
L 76.00355 134.95957 
L 76.00355 133.240133 
L 76.00355 131.520513 
L 76.00355 129.800708 
L 76.00355 128.080714 
L 76.00355 126.360531 
L 76.00355 124.640154 
L 76.00355 122.919583 
L 76.00355 121.198816 
L 76.00355 119.47785 
L 76.00355 117.756684 
L 76.00355 116.035317 
L 76.00355 114.313747 
L 76.00355 112.591972 
L 76.00355 110.869991 
L 76.00355 109.147804 
L 76.00355 107.42541 
L 76.00355 105.702806 
L 76.00355 103.979993 
L 76.00355 102.256971 
L 76.00355 100.533738 
L 76.00355 98.810294 
L 76.00355 97.086639 
L 76.00355 95.362773 
L 76.00355 93.638696 
L 76.00355 91.914408 
L 76.00355 90.189909 
L 76.00355 88.4652 
L 76.00355 86.740281 
L 76.00355 85.015152 
L 76.00355 83.289816 
L 76.00355 81.564271 
L 76.00355 79.83852 
L 76.00355 78.112564 
L 76.00355 76.386403 
L 76.00355 74.660039 
L 76.00355 72.933475 
L 76.00355 71.20671 
L 76.00355 69.479747 
L 76.00355 67.752588 
L 76.00355 66.025235 
L 76.00355 64.29769 
L 76.00355 62.569955 
L 76.00355 60.842032 
L 76.00355 59.113924 
L 76.00355 57.385633 
L 76.00355 55.657162 
L 76.00355 53.928514 
L 76.00355 52.199692 
L 76.00355 50.470698 
L 76.00355 48.741535 
L 76.00355 47.012207 
L 76.00355 45.282718 
L 76.00355 43.553069 
L 76.00355 41.823266 
L 76.00355 40.093311 
L 76.00355 38.363207 
L 76.00355 36.63296 
L 76.00355 34.902572 
L 76.00355 33.172048 
L 76.00355 31.441391 
L 76.00355 29.710605 
L 76.00355 27.979695 
L 76.00355 26.248665 
L 76.00355 24.517519 
L 76.00355 22.786262 
L 76.00355 21.054897 
L 76.00355 19.32343 
L 76.00355 17.591865 
L 76.00355 15.860206 
L 76.00355 14.128459 
L 76.00355 12.396627 
L 76.00355 10.664716 
L 76.00355 8.932731 
L 76.00355 7.200676 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 124.912975 170.279999 
L 124.772975 168.763753 
L 124.598704 166.953344 
L 124.416855 165.144332 
L 124.227474 163.336772 
L 124.030608 161.530715 
L 123.826302 159.726212 
L 123.614608 157.92331 
L 123.395575 156.12206 
L 123.169257 154.322507 
L 122.935706 152.524696 
L 122.694978 150.728672 
L 122.447129 148.934477 
L 122.192217 147.142153 
L 121.930301 145.35174 
L 121.661441 143.563275 
L 121.385699 141.776797 
L 121.103136 139.99234 
L 120.813818 138.209938 
L 120.517808 136.429625 
L 120.215172 134.65143 
L 119.905976 132.875384 
L 119.590289 131.101515 
L 119.268179 129.329849 
L 118.939715 127.560411 
L 118.604967 125.793224 
L 118.264007 124.02831 
L 117.916905 122.265689 
L 117.563734 120.505381 
L 117.204567 118.747401 
L 116.839477 116.991766 
L 116.468539 115.23849 
L 116.091826 113.487586 
L 115.709416 111.739063 
L 115.321381 109.992932 
L 114.9278 108.249201 
L 114.528748 106.507875 
L 114.124301 104.76896 
L 113.714538 103.032459 
L 113.299535 101.298374 
L 112.879369 99.566705 
L 112.454119 97.837451 
L 112.023863 96.110611 
L 111.588678 94.386179 
L 111.148644 92.66415 
L 110.703838 90.944519 
L 110.25434 89.227276 
L 109.800226 87.512412 
L 109.341578 85.799917 
L 108.878472 84.089778 
L 108.410988 82.381981 
L 107.939204 80.676513 
L 107.463199 78.973357 
L 106.983051 77.272495 
L 106.49884 75.57391 
L 106.010642 73.877582 
L 105.518537 72.18349 
L 105.022603 70.491611 
L 104.522917 68.801923 
L 104.019558 67.114402 
L 103.512603 65.429022 
L 103.002129 63.745756 
L 102.488214 62.064578 
L 101.970936 60.385459 
L 101.45037 58.708369 
L 100.926595 57.033279 
L 100.399685 55.360155 
L 99.869718 53.688967 
L 99.336769 52.019681 
L 98.800914 50.352263 
L 98.26223 48.686678 
L 97.72079 47.02289 
L 97.176671 45.360863 
L 96.629947 43.700558 
L 96.080692 42.041939 
L 95.528982 40.384965 
L 94.974889 38.729598 
L 94.418488 37.075797 
L 93.859853 35.423521 
L 93.299057 33.772729 
L 92.736172 32.123377 
L 92.171272 30.475424 
L 91.604429 28.828825 
L 91.035716 27.183537 
L 90.465205 25.539515 
L 89.892968 23.896713 
L 89.319077 22.255087 
L 88.743602 20.61459 
L 88.166616 18.975176 
L 87.588189 17.336798 
L 87.008392 15.699407 
L 86.427297 14.062957 
L 85.844973 12.427399 
L 85.261491 10.792685 
L 84.676921 9.158766 
L 84.091334 7.525593 
L 83.974444 7.200001 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 173.753577 139.646066 
L 173.693663 139.455323 
L 173.042996 137.417344 
L 172.376792 135.390497 
L 171.695407 133.374915 
L 170.999198 131.370717 
L 170.28852 129.378013 
L 169.563729 127.396895 
L 168.825177 125.427446 
L 168.073217 123.469737 
L 167.308197 121.523824 
L 166.530465 119.589753 
L 165.740364 117.667561 
L 164.938234 115.75727 
L 164.124411 113.858894 
L 163.299228 111.972435 
L 162.463013 110.097887 
L 161.616088 108.235234 
L 160.758773 106.384449 
L 159.891381 104.545498 
L 159.014219 102.718339 
L 158.127591 100.902919 
L 157.231794 99.09918 
L 156.327118 97.307055 
L 155.413851 95.526472 
L 154.492271 93.757349 
L 153.562652 91.999601 
L 152.625263 90.253135 
L 151.680364 88.517852 
L 150.728212 86.793649 
L 149.769057 85.080416 
L 148.80314 83.37804 
L 147.830701 81.686402 
L 146.85197 80.00538 
L 145.867172 78.334846 
L 144.876527 76.674672 
L 143.880248 75.024721 
L 142.878542 73.384858 
L 141.871611 71.754941 
L 140.85965 70.134828 
L 139.842851 68.524373 
L 138.821396 66.923428 
L 137.795466 65.331842 
L 136.765233 63.749463 
L 135.730866 62.176136 
L 134.692528 60.611706 
L 133.650377 59.056015 
L 132.604566 57.508904 
L 131.555241 55.970212 
L 130.502548 54.43978 
L 129.446623 52.917443 
L 128.387601 51.40304 
L 127.325611 49.896405 
L 126.260778 48.397375 
L 125.193222 46.905784 
L 124.123061 45.421467 
L 123.050406 43.944258 
L 121.975366 42.473989 
L 120.898046 41.010496 
L 119.818547 39.55361 
L 118.736966 38.103166 
L 117.653398 36.658995 
L 116.567933 35.220933 
L 115.480659 33.78881 
L 114.391659 32.362462 
L 113.301015 30.941721 
L 112.208805 29.526421 
L 111.115104 28.116396 
L 110.019986 26.711479 
L 108.92352 25.311506 
L 107.825773 23.91631 
L 106.726812 22.525727 
L 105.626697 21.139592 
L 104.525491 19.75774 
L 103.42325 18.380007 
L 102.320032 17.006229 
L 101.215891 15.636243 
L 100.110879 14.269886 
L 99.005047 12.906995 
L 97.898444 11.547407 
L 96.791117 10.190961 
L 95.683112 8.837495 
L 94.574473 7.486847 
L 94.338661 7.200001 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 173.753577 48.192521 
L 172.496086 47.197599 
L 170.820338 45.884666 
L 169.148953 44.589415 
L 167.48199 43.311329 
L 165.819496 42.0499 
L 164.161508 40.804635 
L 162.50805 39.575052 
L 160.85914 38.360683 
L 159.214784 37.161069 
L 157.574981 35.975764 
L 155.939722 34.804332 
L 154.308991 33.64635 
L 152.682764 32.501402 
L 151.061011 31.369086 
L 149.443699 30.249007 
L 147.830784 29.14078 
L 146.222222 28.044031 
L 144.617962 26.958394 
L 143.017948 25.88351 
L 141.42212 24.819033 
L 139.830417 23.764619 
L 138.242771 22.719938 
L 136.659113 21.684662 
L 135.079368 20.658476 
L 133.503462 19.641067 
L 131.931316 18.632132 
L 130.36285 17.631373 
L 128.797979 16.6385 
L 127.23662 15.653228 
L 125.678685 14.675277 
L 124.124085 13.704374 
L 122.572729 12.740251 
L 121.024527 11.782646 
L 119.479385 10.8313 
L 117.937208 9.885961 
L 116.3979 8.946381 
L 114.861366 8.012314 
L 113.520186 7.2 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
   </g>
   <g id="LineCollection_2">
    <path clip-path="url(#p6af88d2914)" d="M 7.200001 152.409865 
L 10.901714 153.080061 
L 14.766017 153.736179 
L 18.594701 154.343155 
L 22.390198 154.903208 
L 26.154886 155.418348 
L 29.891092 155.890389 
L 33.601095 156.320964 
L 37.287124 156.71154 
L 40.95137 157.063426 
L 44.595979 157.377787 
L 48.223066 157.655649 
L 51.83471 157.897911 
L 55.432961 158.105348 
L 59.019846 158.278617 
L 62.59737 158.418264 
L 66.167517 158.524725 
L 69.73226 158.598332 
L 73.293562 158.639313 
L 76.853378 158.647796 
L 80.41366 158.623805 
L 83.976361 158.567268 
L 87.543439 158.478009 
L 91.116861 158.35575 
L 94.698606 158.200111 
L 98.290667 158.010603 
L 101.895059 157.786627 
L 105.513818 157.527471 
L 109.149011 157.232301 
L 112.80273 156.900159 
L 116.477106 156.529954 
L 120.174307 156.120452 
L 123.89654 155.670271 
L 127.646059 155.177866 
L 131.425164 154.641519 
L 135.236207 154.059325 
L 139.081591 153.429175 
L 142.963775 152.748743 
L 146.885274 152.015459 
L 150.848658 151.226493 
L 154.856554 150.378726 
L 158.911642 149.468725 
L 163.016654 148.492707 
L 167.174362 147.44651 
L 171.387579 146.325545 
L 173.753575 145.668557 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 7.200002 123.962176 
L 9.665944 124.695946 
L 13.157883 125.681961 
L 16.629653 126.601971 
L 20.082047 127.458564 
L 23.515929 128.254123 
L 26.932217 128.99084 
L 30.331881 129.670726 
L 33.715925 130.295617 
L 37.085392 130.867189 
L 40.441348 131.386958 
L 43.784886 131.856292 
L 47.117116 132.27642 
L 50.439161 132.648431 
L 53.752161 132.973285 
L 57.057262 133.251815 
L 60.355622 133.484731 
L 63.648401 133.672625 
L 66.936767 133.815973 
L 70.22189 133.915137 
L 73.504944 133.970366 
L 76.787104 133.981799 
L 80.069544 133.949465 
L 83.353439 133.873282 
L 86.639965 133.75306 
L 89.930292 133.588494 
L 93.22559 133.37917 
L 96.527024 133.124555 
L 99.835753 132.824003 
L 103.152929 132.476745 
L 106.479696 132.081887 
L 109.817185 131.638408 
L 113.166516 131.145154 
L 116.528788 130.60083 
L 119.905084 130.003999 
L 123.296458 129.353067 
L 126.703936 128.646286 
L 130.128507 127.881735 
L 133.571116 127.057319 
L 137.032658 126.170756 
L 140.513963 125.219566 
L 144.01579 124.20106 
L 147.538812 123.112332 
L 151.083596 121.95024 
L 154.650594 120.7114 
L 158.240113 119.392167 
L 161.852296 117.988625 
L 165.487096 116.496575 
L 169.144239 114.911518 
L 172.823191 113.228645 
L 173.753575 112.788559 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 7.200001 94.349903 
L 9.387851 95.348066 
L 12.33763 96.628619 
L 15.287618 97.836798 
L 18.236964 98.974698 
L 21.184963 100.044311 
L 24.13104 101.047523 
L 27.074741 101.986114 
L 30.015718 102.861759 
L 32.953722 103.676025 
L 35.888588 104.430375 
L 38.820232 105.126168 
L 41.748637 105.764658 
L 44.673849 106.346999 
L 47.595969 106.874244 
L 50.515147 107.347346 
L 53.431575 107.767161 
L 56.345482 108.134448 
L 59.257129 108.449873 
L 62.166804 108.714006 
L 65.074817 108.927324 
L 67.981497 109.090213 
L 70.887188 109.202968 
L 73.792243 109.265793 
L 76.697022 109.278801 
L 79.601887 109.242015 
L 82.507201 109.15537 
L 85.413321 109.018708 
L 88.320596 108.831782 
L 91.229362 108.594255 
L 94.139941 108.305697 
L 97.052633 107.965586 
L 99.967713 107.573307 
L 102.885427 107.12815 
L 105.805989 106.629309 
L 108.72957 106.075882 
L 111.656297 105.466869 
L 114.586246 104.801168 
L 117.519434 104.077579 
L 120.455814 103.294796 
L 123.395263 102.451412 
L 126.337578 101.545914 
L 129.282463 100.576684 
L 132.229521 99.541996 
L 135.178243 98.440019 
L 138.127992 97.268818 
L 141.077996 96.02635 
L 144.027329 94.710471 
L 146.974898 93.318941 
L 149.919427 91.84942 
L 152.859437 90.299482 
L 155.793232 88.666621 
L 158.718875 86.948257 
L 161.634169 85.141754 
L 164.53664 83.24443 
L 167.423507 81.253578 
L 170.29167 79.166486 
L 173.13768 76.980463 
L 173.753576 76.491495 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 7.2 62.125541 
L 7.739508 62.526689 
L 9.944244 64.112404 
L 12.173119 65.631821 
L 14.424109 67.085746 
L 16.695321 68.47502 
L 18.98499 69.80051 
L 21.291473 71.063104 
L 23.613246 72.263693 
L 25.948895 73.40317 
L 28.297112 74.482424 
L 30.65669 75.502326 
L 33.026516 76.463732 
L 35.405566 77.367472 
L 37.792898 78.214351 
L 40.187648 79.005138 
L 42.589022 79.740569 
L 44.996294 80.421343 
L 47.408797 81.048116 
L 49.82592 81.621503 
L 52.247104 82.142073 
L 54.671833 82.610349 
L 57.099635 83.026808 
L 59.530071 83.391874 
L 61.962737 83.705927 
L 64.397254 83.969292 
L 66.833268 84.182244 
L 69.270443 84.345007 
L 71.708459 84.457752 
L 74.147005 84.520599 
L 76.58578 84.533614 
L 79.024483 84.49681 
L 81.462815 84.41015 
L 83.900468 84.27354 
L 86.33713 84.086838 
L 88.772472 83.849847 
L 91.20615 83.562317 
L 93.6378 83.223951 
L 96.067031 82.834396 
L 98.493422 82.393252 
L 100.91652 81.900069 
L 103.335832 81.354349 
L 105.750822 80.755545 
L 108.160907 80.103067 
L 110.565449 79.39628 
L 112.963755 78.634507 
L 115.355066 77.817032 
L 117.738556 76.943102 
L 120.113325 76.01193 
L 122.478391 75.022699 
L 124.832689 73.974566 
L 127.175061 72.866667 
L 129.504254 71.698121 
L 131.81891 70.468039 
L 134.117562 69.175524 
L 136.398631 67.819689 
L 138.660415 66.399653 
L 140.90109 64.91456 
L 143.1187 63.363585 
L 145.311157 61.745944 
L 147.476233 60.060908 
L 149.611563 58.307816 
L 151.714637 56.486088 
L 153.782803 54.59524 
L 155.813268 52.6349 
L 157.803099 50.604827 
L 159.749225 48.504922 
L 161.648446 46.335255 
L 163.497442 44.096075 
L 165.292777 41.787834 
L 167.030917 39.411202 
L 168.708246 36.967086 
L 170.32108 34.456648 
L 171.865687 31.881317 
L 173.338317 29.242808 
L 173.753576 28.461927 
L 173.753577 28.461926 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 7.2 20.657065 
L 7.243479 20.733717 
L 8.316977 22.561791 
L 9.4387 24.352194 
L 10.60693 26.10375 
L 11.819934 27.815388 
L 13.075978 29.486138 
L 14.373328 31.115129 
L 15.710259 32.701584 
L 17.085062 34.244818 
L 18.496048 35.744229 
L 19.94155 37.1993 
L 21.419935 38.609589 
L 22.929597 39.974725 
L 24.468971 41.294405 
L 26.036526 42.568387 
L 27.630774 43.796486 
L 29.250267 44.978567 
L 30.893603 46.114543 
L 32.559421 47.204368 
L 34.246406 48.248034 
L 35.953286 49.245564 
L 37.678833 50.197011 
L 39.421865 51.102452 
L 41.181238 51.961984 
L 42.955854 52.77572 
L 44.744653 53.543787 
L 46.546613 54.266322 
L 48.360753 54.94347 
L 50.186122 55.575377 
L 52.021808 56.162195 
L 53.866928 56.704071 
L 55.720627 57.201152 
L 57.582081 57.653578 
L 59.45049 58.061483 
L 61.325076 58.424993 
L 63.205083 58.744223 
L 65.089774 59.019277 
L 66.978426 59.250249 
L 68.870332 59.437216 
L 70.764796 59.580244 
L 72.66113 59.679383 
L 74.558654 59.734668 
L 76.456692 59.746119 
L 78.354569 59.71374 
L 80.251609 59.637519 
L 82.147136 59.51743 
L 84.040465 59.353429 
L 85.930905 59.14546 
L 87.817754 58.893451 
L 89.7003 58.597316 
L 91.577811 58.256957 
L 93.449544 57.872265 
L 95.31473 57.443118 
L 97.172583 56.969388 
L 99.02229 56.450937 
L 100.863013 55.88762 
L 102.693886 55.279292 
L 104.514009 54.625802 
L 106.322454 53.927002 
L 108.118255 53.182744 
L 109.900412 52.392887 
L 111.667884 51.5573 
L 113.419595 50.675862 
L 115.154423 49.748465 
L 116.871207 48.775022 
L 118.568743 47.755468 
L 120.245783 46.689764 
L 121.901033 45.577899 
L 123.533157 44.419901 
L 125.140775 43.215834 
L 126.722461 41.965808 
L 128.276748 40.66998 
L 129.802128 39.328562 
L 131.297054 37.941825 
L 132.75994 36.510103 
L 134.189168 35.033801 
L 135.583087 33.513396 
L 136.940019 31.949446 
L 138.258266 30.342589 
L 139.536109 28.693557 
L 140.771819 27.003169 
L 141.963661 25.272344 
L 143.109901 23.5021 
L 144.208811 21.693557 
L 145.258682 19.847942 
L 146.257827 17.966587 
L 147.204591 16.050932 
L 148.097363 14.102527 
L 148.934578 12.123029 
L 149.714736 10.114198 
L 150.4364 8.077902 
L 150.72656 7.2 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 30.479021 7.2 
L 30.83753 7.898083 
L 31.486819 9.09739 
L 32.167975 10.276669 
L 32.880265 11.435081 
L 33.622936 12.571823 
L 34.395218 13.686128 
L 35.196323 14.777262 
L 36.025452 15.844528 
L 36.881791 16.887266 
L 37.76452 17.904846 
L 38.672808 18.896677 
L 39.605819 19.862198 
L 40.562713 20.800882 
L 41.542647 21.712234 
L 42.544776 22.59579 
L 43.568256 23.451116 
L 44.612243 24.277806 
L 45.675896 25.075484 
L 46.758377 25.843799 
L 47.858853 26.582425 
L 48.976495 27.291063 
L 50.11048 27.969434 
L 51.25999 28.617284 
L 52.424216 29.234379 
L 53.602354 29.820503 
L 54.793608 30.375461 
L 55.997189 30.899074 
L 57.212315 31.391179 
L 58.438212 31.851631 
L 59.674113 32.280297 
L 60.919259 32.677058 
L 62.172896 33.041807 
L 63.434278 33.374451 
L 64.702664 33.674905 
L 65.977321 33.943096 
L 67.257518 34.178962 
L 68.542532 34.382446 
L 69.831642 34.553503 
L 71.124132 34.692096 
L 72.419288 34.798193 
L 73.7164 34.871772 
L 75.014759 34.912818 
L 76.313656 34.92132 
L 77.612384 34.897279 
L 78.910235 34.840698 
L 80.206502 34.75159 
L 81.500474 34.629974 
L 82.79144 34.475876 
L 84.078684 34.28933 
L 85.361488 34.070378 
L 86.63913 33.819071 
L 87.910884 33.535466 
L 89.176018 33.219633 
L 90.433795 32.871648 
L 91.683471 32.491602 
L 92.924297 32.079593 
L 94.155517 31.635735 
L 95.376367 31.160152 
L 96.586077 30.652983 
L 97.78387 30.114383 
L 98.968961 29.54452 
L 100.140556 28.943583 
L 101.297856 28.311776 
L 102.440055 27.649322 
L 103.566338 26.956466 
L 104.675887 26.233473 
L 105.767875 25.480633 
L 106.84147 24.698256 
L 107.895838 23.88668 
L 108.930138 23.046268 
L 109.943529 22.177411 
L 110.935166 21.280527 
L 111.904204 20.356065 
L 112.8498 19.404503 
L 113.771111 18.426352 
L 114.667299 17.422155 
L 115.537531 16.392487 
L 116.38098 15.337957 
L 117.196829 14.259209 
L 117.984271 13.156923 
L 118.742511 12.031812 
L 119.470772 10.884625 
L 120.168289 9.716147 
L 120.834319 8.527198 
L 121.46814 7.318634 
L 121.528078 7.200001 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
    <path clip-path="url(#p6af88d2914)" d="M 64.347917 7.2 
L 64.696052 7.379124 
L 65.290618 7.669398 
L 65.89245 7.943786 
L 66.501131 8.20212 
L 67.116236 8.444243 
L 67.737342 8.670008 
L 68.36402 8.87928 
L 68.995842 9.071933 
L 69.632376 9.247852 
L 70.27319 9.406932 
L 70.917848 9.54908 
L 71.565916 9.674212 
L 72.216955 9.782254 
L 72.87053 9.873142 
L 73.526199 9.946823 
L 74.183526 10.003255 
L 74.842068 10.042404 
L 75.501387 10.064248 
L 76.161041 10.068773 
L 76.820589 10.055978 
L 77.479592 10.025869 
L 78.137609 9.978464 
L 78.794199 9.913792 
L 79.448923 9.831888 
L 80.101342 9.732803 
L 80.751017 9.616593 
L 81.397511 9.483326 
L 82.040388 9.333082 
L 82.679213 9.165949 
L 83.313551 8.982027 
L 83.94297 8.781424 
L 84.567041 8.564261 
L 85.185335 8.330668 
L 85.797426 8.080785 
L 86.402889 7.814766 
L 87.001304 7.532771 
L 87.592252 7.234974 
L 87.659183 7.2 
" style="fill:none;stroke:#b0b0b0;stroke-width:0.8;"/>
   </g>
   <g id="patch_3">
    <path d="M 7.2 170.28 
L 7.2 7.2 
L 173.753577 7.2 
L 173.753577 170.28 
L 7.2 170.28 
" style="fill:none;stroke:#000000;stroke-linejoin:miter;stroke-width:0.8;"/>
   </g>
  </g>
 </g>
 <defs>
  <clipPath id="p6af88d2914">
   <path d="M 7.2 170.28 
L 7.2 7.2 
L 173.753577 7.2 
L 173.753577 170.28 
L 7.2 170.28 
"/>
  </clipPath>
 </defs>
</svg>
<pre>_PROJ4Projection(+ellps=WGS84 +k=1.0 +lat_0=0.0 +lon_0=0.0 +no_defs=True +proj=tmerc +type=crs +units=m +x_0=0.0 +y_0=0.0 +no_defs)</pre>



<br>

We can now carry out the resampling using the `pyresample` library

```python
%%time

resampled_scene = scene.resample(tm_area_def, resampler='nearest')
```

    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\pyresample\spherical.py:123: RuntimeWarning: invalid value encountered in true_divide
      self.cart /= np.sqrt(np.einsum('...i, ...i', self.cart, self.cart))
    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\pyresample\spherical.py:178: RuntimeWarning: invalid value encountered in double_scalars
      return (val + mod) % (2 * mod) - mod
    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\pyproj\crs\crs.py:543: UserWarning: You will likely lose important projection information when converting to a PROJ string from another format. See: https://proj.org/faq.html#what-is-the-best-format-for-describing-coordinate-reference-systems
      proj_string = self.to_proj4()
    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\numpy\lib\function_base.py:1280: RuntimeWarning: invalid value encountered in subtract
      a = op(a[slice1], a[slice2])
    

    Wall time: 8.86 s
    

<br>

We'll quickly check that the reprojection looks ok

```python
fig = plt.figure(dpi=250, figsize=(10, 10))
ax = plt.axes(projection=ccrs.TransverseMercator())

resampled_scene['HRV'].plot.imshow(ax=ax)

ax.coastlines(resolution='50m', alpha=0.8, color='white')
```

    <ipython-input-20-ec0e500c536a>:2: UserWarning: The default value for the *approx* keyword argument to TransverseMercator will change from True to False after 0.18.
      ax = plt.axes(projection=ccrs.TransverseMercator())
    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\dask\core.py:121: RuntimeWarning: invalid value encountered in sin
      return func(*(_execute_task(a, cache) for a in args))
    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\dask\core.py:121: RuntimeWarning: invalid value encountered in cos
      return func(*(_execute_task(a, cache) for a in args))
    




    <cartopy.mpl.feature_artist.FeatureArtist at 0x2948d7e00d0>




![png](img/nbs/output_35_2.png)


<br>

We want to gain a deeper understanding of the reprojection that's being carried out, to do this we'll manually reproject a sample of the original gridded coordinates

```python
%%time

orig_x_values = scene['HRV'].x.values[::50]
orig_y_values = scene['HRV'].y.values[::50]

XX, YY = np.meshgrid(orig_x_values, orig_y_values)

df_proj_points = (gpd
                    .GeoSeries([
                        Point(x, y) 
                        for x, y 
                        in np.stack([XX.flatten(), YY.flatten()], axis=1)
                    ])
                    .set_crs(crs=scene['HRV'].area.crs_wkt)
                    .to_crs(crs=resampled_scene['HRV'].area.crs_wkt)
                    .apply(lambda point: pd.Series(list(point.coords)[0]))
                    .rename(columns={0: 'x_reproj', 1: 'y_reproj'})
                    .replace(np.inf, np.nan)
                    .pipe(lambda df: df.assign(x_orig=XX.flatten()))
                    .pipe(lambda df: df.assign(y_orig=YY.flatten()))
                   )

df_proj_points.head()
```

    Wall time: 5.41 s
    




|    |    x_reproj |    y_reproj |      x_orig |      y_orig |
|---:|------------:|------------:|------------:|------------:|
|  0 | 4.86340e+06 | 1.91779e+06 | 3.16442e+06 | 1.39519e+06 |
|  1 | 4.78132e+06 | 1.89942e+06 | 3.11442e+06 | 1.39519e+06 |
|  2 | 4.70062e+06 | 1.88175e+06 | 3.06441e+06 | 1.39519e+06 |
|  3 | 4.62123e+06 | 1.86473e+06 | 3.01440e+06 | 1.39519e+06 |
|  4 | 4.5431e+06  | 1.84834e+06 | 2.9644e+06  | 1.39519e+06 |</div>



<br>

We can then visualise the reprojection of the original grid against the regridded reprojection

```python
%%time

fig = plt.figure(dpi=250, figsize=(10, 10))
ax = plt.axes(projection=ccrs.TransverseMercator())

resampled_scene['HRV'].plot.imshow(ax=ax, cmap='Greys_r')

ax.coastlines(resolution='50m', alpha=0.8, color='white')
ax.scatter(df_proj_points['x_reproj'][::10], df_proj_points['y_reproj'][::10], s=2, color='red')
```

    <timed exec>:2: UserWarning: The default value for the *approx* keyword argument to TransverseMercator will change from True to False after 0.18.
    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\dask\core.py:121: RuntimeWarning: invalid value encountered in cos
      return func(*(_execute_task(a, cache) for a in args))
    C:\Users\Ayrto\anaconda3\envs\satip_dev\lib\site-packages\dask\core.py:121: RuntimeWarning: invalid value encountered in sin
      return func(*(_execute_task(a, cache) for a in args))
    

    Wall time: 27.8 s
    




    <matplotlib.collections.PathCollection at 0x294fb0be3a0>




![png](img/nbs/output_39_3.png)


<br>

This is useful for quick visual inspection, for example we can see that the y axis gets stretched further the nearer to the pole. However, we want to get a better understanding of how the local cell resolution is changing for any given point, we'll begin by looking at this change for Greenwich.

```python
def lon_lat_to_new_crs(lon, lat, crs):
    x, y = list(gpd
                .GeoSeries([Point(lon, lat)])
                .set_crs(4326)
                .to_crs(crs)
                .iloc[0]
                .coords
               )[0]
    
    return x, y

def calc_res_change(src_x, src_y, src_da, dst_da, src_dx=10, src_dy=10):
    src_crs = src_da.area.crs_wkt
    dst_crs = dst_da.area.crs_wkt
    
    src_x_width = np.abs(np.diff(src_da.x.values)[0])
    src_y_width = np.abs(np.diff(src_da.y.values)[0])
    dst_x_width = np.abs(np.diff(dst_da.x.values)[0])
    dst_y_width = np.abs(np.diff(dst_da.y.values)[0])
    
    s_points = (gpd
                .GeoSeries([
                    Point(src_x, src_y),
                    Point(src_x+src_dx, src_y),
                    Point(src_x, src_y+src_dy)
                ])
                .set_crs(src_crs)
                .to_crs(dst_crs)
               )
    
    dst_dx = s_points.iloc[0].distance(s_points.iloc[1])
    dst_dy = s_points.iloc[0].distance(s_points.iloc[2])
    
    x_ratio_change = (dst_dx/dst_x_width) / (src_dx/src_x_width)
    y_ratio_change = (dst_dy/dst_y_width) / (src_dy/src_y_width)
    
    return x_ratio_change, y_ratio_change

lon = 0
lat = 51.4934

src_x, src_y = lon_lat_to_new_crs(lon, lat, scene['HRV'].area.crs_wkt)

x_ratio_change, y_ratio_change = calc_res_change(src_x, src_y, 
                                                 scene['HRV'], 
                                                 resampled_scene['HRV'])

x_ratio_change, y_ratio_change
```




    (0.27381567467569573, 0.528776076616483)



<br>

We'll double check this by calculating it through a different method, in this case by locating the nearest cell for each scene and comparing their sizes in a common coordinate system

```python
def get_da_nearest_cell_width_height(da, x, y, units_crs):
    nearest_loc = da.sel(x=x, y=y, method='nearest')

    nearest_x = nearest_loc.x.values
    nearest_y = nearest_loc.y.values

    next_nearest_x = da.x.values[list(da.x.values).index(nearest_x)+1]
    next_nearest_y = da.y.values[list(da.y.values).index(nearest_y)+1]
    
    s_points = (gpd
                .GeoSeries([
                    Point(nearest_x, nearest_y),
                    Point(next_nearest_x, nearest_y),
                    Point(nearest_x, next_nearest_y)
                ])
                .set_crs(da.area.crs_wkt)
                .to_crs(units_crs)
               )
    
    x_width = s_points.iloc[0].distance(s_points.iloc[1])
    y_height = s_points.iloc[0].distance(s_points.iloc[2])
    
    return x_width, y_height

src_x, src_y = lon_lat_to_new_crs(lon, lat, scene['HRV'].area.crs_wkt)
dst_x, dst_y = lon_lat_to_new_crs(lon, lat, resampled_scene['HRV'].area.crs_wkt)

src_x_width, src_y_height = get_da_nearest_cell_width_height(scene['HRV'], src_x, src_y, 27700)
dst_x_width, dst_y_height = get_da_nearest_cell_width_height(resampled_scene['HRV'], dst_x, dst_y, 27700)

print(f'The width has changed from {round(src_x_width/1000, 2)} km to {round(dst_x_width/1000, 2)} km')
print(f'The height has changed from {round(src_y_height/1000, 2)} km to {round(dst_y_height/1000, 2)} km')
```

    The width has changed from 1.09 km to 4.0 km
    The height has changed from 2.12 km to 4.0 km
    

<br>

This can easily be converted into a x and y pixel size ratio change which almost exactly matches our previous calculation. The first calculation is more accurate as the dx and dy can approach 0 and get closer to the true ratio change, however the `get_da_nearest_cell_width_height` function is still useful as it allows us to determine the cell width and height in more interpretable units

```python
x_ratio_change, y_ratio_change = src_x_width/dst_x_width, src_y_height/dst_y_height

x_ratio_change, y_ratio_change
```




    (0.2738180115545141, 0.5290020702784486)



<br>

Iceland is stretched further still

```python
def print_pixel_change(lon, lat, da_src, da_dst):
    src_x, src_y = lon_lat_to_new_crs(lon, lat, da_src.area.crs_wkt)
    dst_x, dst_y = lon_lat_to_new_crs(lon, lat, da_dst.area.crs_wkt)

    src_x_width, src_y_height = get_da_nearest_cell_width_height(da_src, src_x, src_y, 27700)
    dst_x_width, dst_y_height = get_da_nearest_cell_width_height(da_dst, dst_x, dst_y, 27700)

    print(f'The width has changed from {round(src_x_width/1000, 2)} km to {round(dst_x_width/1000, 2)} km')
    print(f'The height has changed from {round(src_y_height/1000, 2)} km to {round(dst_y_height/1000, 2)} km')
    
    return

lon = -18.779208
lat = 64.887370

print_pixel_change(lon, lat, scene['HRV'], resampled_scene['HRV'])
```

    The width has changed from 1.52 km to 3.99 km
    The height has changed from 4.75 km to 3.99 km
    

<br>

And contrasts with Marrakesh which is stretched less than Greenwich in the y axis

```python
lon = -8.005657
lat = 31.636355

print_pixel_change(lon, lat, scene['HRV'], resampled_scene['HRV'])
```

    The width has changed from 1.11 km to 3.99 km
    The height has changed from 1.33 km to 3.99 km
    

<br>

We can check what the cell height and width are at the center of the image, they should both be close to 1km according to the SEVIRI documentation
> <b>LineDirGridStep</b> gives the grid step size in km SSP in the line direction. Default value is 3km for VIS and IR, and 1km for HRV. The on-ground grid step size of 3 km at the SSP represents an instrument scan step of 251.53 microrad divided by 3. - <a href="http://www.eumetsat.int/website/wcm/idc/idcplg?IdcService=GET_FILE&dDocName=PDF_TEN_05105_MSG_IMG_DATA&RevisionSelectionMethod=LatestReleased&Rendition=Web">EUMETSAT</a>

```python
round_m_to_km = lambda m: round(m/1000, 2)

UTM_35N_epsg = 32632 # should be relatively accurate and is in meters

src_x = np.median(scene['HRV'].x.values)
src_y = np.median(scene['HRV'].y.values)

src_x_width, src_y_height = get_da_nearest_cell_width_height(scene['HRV'], src_x, src_y, UTM_35N_epsg)

round_m_to_km(src_x_width), round_m_to_km(src_y_height)
```




    (1.04, 1.36)



<br>

### Comparing Reprojection Libraries

In the last section we used `pyresample` to carry out the data reprojection, here we'll explore <a href="https://pangeo-pyinterp.readthedocs.io/en/latest/examples.html">`pyinterp`</a>.

Before we start we'll quickly extract the xarrays for the original and reprojected coordinates.

```python
def extract_formatted_scene(scene, variable='HRV', 
                            x_coords_name='x', 
                            y_coords_name='y', 
                            x_units='metre', 
                            y_units='metre'):
    da = (scene
          [variable]
          .copy()
          .rename({
              'x': x_coords_name, 
              'y': y_coords_name
          })
         )
    
    da[x_coords_name].attrs['units'] = x_units
    da[y_coords_name].attrs['units'] = y_units
    
    return da

da = extract_formatted_scene(scene)
da_resampled = extract_formatted_scene(resampled_scene)

da_resampled
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
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;my_index-25c95e08ed138cbd282b6596ed55c066&#x27; (y: 1831, x: 1870)&gt;
dask.array&lt;copy, shape=(1831, 1870), dtype=float32, chunksize=(1831, 1870), chunktype=numpy.ndarray&gt;
Coordinates:
    crs      object PROJCRS[&quot;unknown&quot;,BASEGEOGCRS[&quot;unknown&quot;,DATUM[&quot;Unknown ba...
  * y        (y) float64 9.012e+06 9.008e+06 9.004e+06 ... 1.696e+06 1.692e+06
  * x        (x) float64 -3.088e+06 -3.084e+06 -3.08e+06 ... 4.384e+06 4.388e+06
Attributes:
    orbital_parameters:                     {&#x27;projection_longitude&#x27;: 9.5, &#x27;pr...
    sun_earth_distance_correction_applied:  True
    sun_earth_distance_correction_factor:   0.9697642568677852
    units:                                  %
    wavelength:                             0.7â€¯ÂµmÂ (0.5-0.9â€¯Âµm)
    standard_name:                          toa_bidirectional_reflectance
    platform_name:                          Meteosat-9
    sensor:                                 seviri
    start_time:                             2020-12-08 09:00:08.206321
    end_time:                               2020-12-08 09:05:08.329479
    area:                                   Area ID: TM\nDescription: Transve...
    name:                                   HRV
    resolution:                             1000.134348869
    calibration:                            reflectance
    modifiers:                              ()
    _satpy_id:                              DataID(name=&#x27;HRV&#x27;, wavelength=Wav...
    ancillary_variables:                    []</pre><div class='xr-wrap' hidden><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'my_index-25c95e08ed138cbd282b6596ed55c066'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>y</span>: 1831</li><li><span class='xr-has-index'>x</span>: 1870</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-77e13003-33d7-42fb-a045-b7cc4e978072' class='xr-array-in' type='checkbox' checked><label for='section-77e13003-33d7-42fb-a045-b7cc4e978072' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>dask.array&lt;chunksize=(1831, 1870), meta=np.ndarray&gt;</span></div><div class='xr-array-data'><table>
<tr>
<td>
<table>
  <thead>
    <tr><td> </td><th> Array </th><th> Chunk </th></tr>
  </thead>
  <tbody>
    <tr><th> Bytes </th><td> 13.70 MB </td> <td> 13.70 MB </td></tr>
    <tr><th> Shape </th><td> (1831, 1870) </td> <td> (1831, 1870) </td></tr>
    <tr><th> Count </th><td> 360 Tasks </td><td> 1 Chunks </td></tr>
    <tr><th> Type </th><td> float32 </td><td> numpy.ndarray </td></tr>
  </tbody>
</table>
</td>
<td>
<svg width="170" height="167" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="120" y2="0" style="stroke-width:2" />
  <line x1="0" y1="117" x2="120" y2="117" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="117" style="stroke-width:2" />
  <line x1="120" y1="0" x2="120" y2="117" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 120.0,0.0 120.0,117.49732620320856 0.0,117.49732620320856" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="60.000000" y="137.497326" font-size="1.0rem" font-weight="100" text-anchor="middle" >1870</text>
  <text x="140.000000" y="58.748663" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,140.000000,58.748663)">1831</text>
</svg>
</td>
</tr>
</table></div></div></li><li class='xr-section-item'><input id='section-584cdfbb-94bb-436c-ab31-71efda3ea3bd' class='xr-section-summary-in' type='checkbox'  checked><label for='section-584cdfbb-94bb-436c-ab31-71efda3ea3bd' class='xr-section-summary' >Coordinates: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>crs</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>PROJCRS[&quot;unknown&quot;,BASEGEOGCRS[&quot;u...</div><input id='attrs-2882d840-ac48-4ea1-a41d-3cbdc0852c29' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-2882d840-ac48-4ea1-a41d-3cbdc0852c29' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-26a2260d-9a0b-4121-8797-b091563b3ba1' class='xr-var-data-in' type='checkbox'><label for='data-26a2260d-9a0b-4121-8797-b091563b3ba1' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(&lt;Projected CRS: PROJCRS[&quot;unknown&quot;,BASEGEOGCRS[&quot;unknown&quot;,DATUM[&quot;Unk ...&gt;
Name: unknown
Axis Info [cartesian]:
- E[east]: Easting (metre)
- N[north]: Northing (metre)
Area of Use:
- undefined
Coordinate Operation:
- name: unknown
- method: Transverse Mercator
Datum: Unknown based on WGS84 ellipsoid
- Ellipsoid: WGS 84
- Prime Meridian: Greenwich
, dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>9.012e+06 9.008e+06 ... 1.692e+06</div><input id='attrs-948816fd-dce8-4899-a909-1dd029f03f32' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-948816fd-dce8-4899-a909-1dd029f03f32' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b909c86a-e8f8-4631-b0f7-d15fe2d9646e' class='xr-var-data-in' type='checkbox'><label for='data-b909c86a-e8f8-4631-b0f7-d15fe2d9646e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>metre</dd></dl></div><div class='xr-var-data'><pre>array([9012000., 9008000., 9004000., ..., 1700000., 1696000., 1692000.])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>-3.088e+06 -3.084e+06 ... 4.388e+06</div><input id='attrs-73f17606-cb6e-4da7-8f33-e51df5f6c167' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-73f17606-cb6e-4da7-8f33-e51df5f6c167' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c85378f0-d705-4996-88f8-f3b891d9ad59' class='xr-var-data-in' type='checkbox'><label for='data-c85378f0-d705-4996-88f8-f3b891d9ad59' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>metre</dd></dl></div><div class='xr-var-data'><pre>array([-3088000., -3084000., -3080000., ...,  4380000.,  4384000.,  4388000.])</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-5cfc7e6c-3a34-410b-8086-8060ca3b654f' class='xr-section-summary-in' type='checkbox'  ><label for='section-5cfc7e6c-3a34-410b-8086-8060ca3b654f' class='xr-section-summary' >Attributes: <span>(17)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>orbital_parameters :</span></dt><dd>{&#x27;projection_longitude&#x27;: 9.5, &#x27;projection_latitude&#x27;: 0.0, &#x27;projection_altitude&#x27;: 35785831.0}</dd><dt><span>sun_earth_distance_correction_applied :</span></dt><dd>True</dd><dt><span>sun_earth_distance_correction_factor :</span></dt><dd>0.9697642568677852</dd><dt><span>units :</span></dt><dd>%</dd><dt><span>wavelength :</span></dt><dd>0.7â€¯ÂµmÂ (0.5-0.9â€¯Âµm)</dd><dt><span>standard_name :</span></dt><dd>toa_bidirectional_reflectance</dd><dt><span>platform_name :</span></dt><dd>Meteosat-9</dd><dt><span>sensor :</span></dt><dd>seviri</dd><dt><span>start_time :</span></dt><dd>2020-12-08 09:00:08.206321</dd><dt><span>end_time :</span></dt><dd>2020-12-08 09:05:08.329479</dd><dt><span>area :</span></dt><dd>Area ID: TM
Description: Transverse Mercator
Projection ID: TM
Projection: {&#x27;ellps&#x27;: &#x27;WGS84&#x27;, &#x27;k&#x27;: &#x27;1&#x27;, &#x27;lat_0&#x27;: &#x27;0&#x27;, &#x27;lon_0&#x27;: &#x27;0&#x27;, &#x27;no_defs&#x27;: &#x27;None&#x27;, &#x27;proj&#x27;: &#x27;tmerc&#x27;, &#x27;type&#x27;: &#x27;crs&#x27;, &#x27;units&#x27;: &#x27;m&#x27;, &#x27;x_0&#x27;: &#x27;0&#x27;, &#x27;y_0&#x27;: &#x27;0&#x27;}
Number of columns: 1870
Number of rows: 1831
Area extent: (-3090000, 1690000, 4390000, 9014000)</dd><dt><span>name :</span></dt><dd>HRV</dd><dt><span>resolution :</span></dt><dd>1000.134348869</dd><dt><span>calibration :</span></dt><dd>reflectance</dd><dt><span>modifiers :</span></dt><dd>()</dd><dt><span>_satpy_id :</span></dt><dd>DataID(name=&#x27;HRV&#x27;, wavelength=WavelengthRange(min=0.5, central=0.7, max=0.9, unit=&#x27;Âµm&#x27;), resolution=1000.134348869, calibration=&lt;calibration.reflectance&gt;, modifiers=())</dd><dt><span>ancillary_variables :</span></dt><dd>[]</dd></dl></div></li></ul></div></div>



<br>

We'll now save the coordinates of the grid we're using in the new projection

```python
new_grid_4km_TM = {
    'x_coords': list(da_resampled.x.values),
    'y_coords': list(da_resampled.y.values)
}

save_data = True

if save_data == True:
    with open('../data/intermediate/new_grid_4km_TM.json', 'w') as fp:
        json.dump(new_grid_4km_TM, fp)

JSON(new_grid_4km_TM)
```




    <IPython.core.display.JSON object>



<br>

As well as calculate the locations of those points in the original CRS

```python
%%time

def chunks(list_, n):
    """
    Yield successive n-sized chunks from `list_`.
    """
    
    for i in range(0, len(list_), n):
        yield list_[i:i + n]
        
def reproject_geometries(da, old_crs, new_crs, chunk_size=5000):
    xx, yy = np.meshgrid(da.x.values, da.y.values, indexing='ij')
    geometry = gpd.points_from_xy(xx.flatten(), yy.flatten())

    new_coords_samples = []

    for geometry_sample in chunks(geometry, chunk_size):
        df_new_coords_sample = (gpd
                                .GeoSeries(geometry_sample, crs=old_crs)
                                .to_crs(new_crs)
                                .apply(lambda x: list(x.coords[0]))
                                .apply(pd.Series)
                                .rename(columns={0: 'x', 1: 'y'})
                               )

        new_coords_samples += [df_new_coords_sample]

    df_new_coords = pd.concat(new_coords_samples, ignore_index=True)
    
    return df_new_coords

if not os.path.exists(intermediate_data_dir):
    os.makedirs(intermediate_data_dir)

if calculate_reproj_coords == True:
    df_new_coords = reproject_geometries(da_resampled, '+proj=tmerc', seviri_crs.proj4_init)
    df_new_coords.to_csv(f'{intermediate_data_dir}/reproj_coords_TM_4km.csv', index=False)
elif 'reproj_coords.csv' not in os.listdir(intermediate_data_dir):
    df_new_coords = pd.read_csv('https://storage.googleapis.com/reprojection_cache/reproj_coords_TM_4km.csv')
else:
    df_new_coords = pd.read_csv(f'{intermediate_data_dir}/reproj_coords_TM_4km.csv')

df_new_coords.head()
```

    Wall time: 3.36 s
    




|    |   x |   y |
|---:|----:|----:|
|  0 | inf | inf |
|  1 | inf | inf |
|  2 | inf | inf |
|  3 | inf | inf |
|  4 | inf | inf |</div>



<br>

We can layer these on top of each other to get an alternative view of the transform operation

```python
%%time

old_x_positions, old_y_positions = [elem.flatten() for elem in np.meshgrid(da.x.values[::100], da.y.values[::100], indexing='ij')]
new_x_positions, new_y_positions = df_new_coords['x'][::100], df_new_coords['y'][::100]

# Plotting
fig, ax = plt.subplots(dpi=150)

ax.scatter(old_x_positions, old_y_positions, s=0.1)
ax.scatter(new_x_positions, new_y_positions, s=0.1)

hlp.hide_spines(ax)
```

    Wall time: 98.8 ms
    


![png](img/nbs/output_59_1.png)


<br>

We'll now use `pyinterp` to take these and use them to carry out the resampling. We'll also create a wrapper for converting the result back into an Xarray object.

```python
#exports
def reproj_with_manual_grid(da, x_coords, y_coords, new_grid):
    
    x_axis = pyinterp.Axis(da.x.values)
    y_axis = pyinterp.Axis(da.y.values)

    grid = pyinterp.Grid2D(x_axis, y_axis, da.data.T)

    reproj_data = (pyinterp
                   .bivariate(grid, x_coords, y_coords)
                   .reshape((len(new_grid['x_coords']), len(new_grid['y_coords'])))
                  )

    return reproj_data

def reproj_to_xarray(da, x_coords, y_coords, new_grid):
    # We'll reproject the data
    reproj_data = reproj_with_manual_grid(da, x_coords, y_coords, new_grid)

    # Then put it in an XArray DataArray
    da_reproj = xr.DataArray(np.flip(reproj_data.T, axis=(0, 1)), 
                             dims=('y', 'x'), 
                             coords={
                                 'x': new_grid['x_coords'][::-1], 
                                 'y': new_grid['y_coords'][::-1]
                             }, 
                             attrs=da.attrs)
    
    return da_reproj
```

<br>

We'll load the grid back in

```python
with open('../data/intermediate/new_grid_4km_TM.json', 'r') as fp:
    new_grid = json.load(fp)

JSON(new_grid)
```




    <IPython.core.display.JSON object>



<br>

Confirm that the size of the grid definition arrays match the number of coordinates we have

```python
df_new_coords['y'].size == len(new_grid['x_coords'])*len(new_grid['y_coords'])
```




    True



<br>

And finally carry out the reprojection

```python
%%timeit

da_reproj = reproj_to_xarray(da, df_new_coords['x'], df_new_coords['y'], new_grid)
```

    1.78 s Â± 239 ms per loop (mean Â± std. dev. of 7 runs, 1 loop each)
    

<br>

Most importantly we'll carry out a visual check that the reprojection was carried out properly.

```python
da_reproj = reproj_to_xarray(da, df_new_coords['x'], df_new_coords['y'], new_grid)

# Plotting
fig = plt.figure(dpi=250, figsize=(10, 10))
ax = plt.axes(projection=ccrs.TransverseMercator())

da_reproj.plot.imshow(ax=ax, cmap='Greys_r')

ax.coastlines(resolution='50m', alpha=0.8, color='white')
```

    <ipython-input-37-c765a7c3ab68>:5: UserWarning: The default value for the *approx* keyword argument to TransverseMercator will change from True to False after 0.18.
      ax = plt.axes(projection=ccrs.TransverseMercator())
    




    <cartopy.mpl.feature_artist.FeatureArtist at 0x2948c866b50>




![png](img/nbs/output_69_2.png)


```python
#exports
def full_scene_pyresample(native_fp):
    # Loading scene
    scene = load_scene(native_fp)
    dataset_names = scene.all_dataset_names()
    scene.load(dataset_names)
    
    # Constructing target area definition
    tm_area_def = construct_TM_area_def(scene)
    
    # Reprojecting
    reproj_vars = list()

    for dataset_name in dataset_names:
        da = scene[dataset_name].sortby('y', ascending=False).sortby('x')
        num_y_pixels, num_x_pixels = da.shape
        seviri_area_def = get_seviri_area_def(native_fp, num_x_pixels=num_x_pixels, num_y_pixels=num_y_pixels)
        
        resampler = satpy.resample.KDTreeResampler(seviri_area_def, tm_area_def)
        da_reproj = resampler.resample(da)
        reproj_vars += [da_reproj]
    
    variable_idx = pd.Index(dataset_names, name='variable')
    
    ds_reproj = (xr
                 .concat(reproj_vars, dim=variable_idx)
                 .to_dataset(name='stacked_eumetsat_data')
                 .drop(labels='crs')
                )
    
    return ds_reproj

def full_scene_pyinterp(native_fp, new_x_coords, new_y_coords, new_grid_fp):
    # Loading data
    scene = load_scene(native_fp)
    dataset_names = scene.all_dataset_names()
    scene.load(dataset_names)
    
    with open(new_grid_fp, 'r') as fp:
        new_grid = json.load(fp)

    # Correcting x coordinates
    seviri_area_def = get_seviri_area_def(native_fp)
    area_extent = seviri_area_def.area_extent
    x_offset = calculate_x_offset(native_fp)

    width = scene['HRV'].x.size
    corrected_x_coords = np.linspace(area_extent[2], area_extent[0], width)
    scene['HRV'] = scene['HRV'].assign_coords({'x': corrected_x_coords})
    
    # Reprojecting
    reproj_vars = list()

    for dataset_name in dataset_names:
        da_reproj = reproj_to_xarray(scene[dataset_name], new_x_coords, new_y_coords, new_grid)
        reproj_vars += [da_reproj]
    
    variable_idx = pd.Index(dataset_names, name='variable')
    ds_reproj = xr.concat(reproj_vars, dim=variable_idx).to_dataset(name='stacked_eumetsat_data')
    
    return ds_reproj

class Reprojector:
    def __init__(self, new_coords_fp=None, new_grid_fp=None):
        if new_coords_fp is None and new_grid_fp is None:
            return
        
        df_new_coords = pd.read_csv(new_coords_fp)
        
        self.new_x_coords = df_new_coords['x']
        self.new_y_coords = df_new_coords['y']
        self.new_grid_fp = new_grid_fp
        
        return
    
    def reproject(self, native_fp, reproj_library='pyresample'):
        if reproj_library == 'pyinterp':
            ds_reproj = full_scene_pyinterp(native_fp, self.new_x_coords, self.new_y_coords, self.new_grid_fp)
        elif reproj_library == 'pyresample':
            ds_reproj = full_scene_pyresample(native_fp)
        else:
            raise ValueError(f'`reproj_library` must be one of: pyresample, pyinterp. {reproj_library} can not be passed.')
            
        return ds_reproj
```

```python
%%capture --no-stdout
%%timeit 

new_coords_fp = f'{intermediate_data_dir}/reproj_coords_TM_4km.csv'
new_grid_fp='../data/intermediate/new_grid_4km_TM.json'

reprojector = Reprojector(new_coords_fp, new_grid_fp)
ds_reproj = reprojector.reproject(native_fp, reproj_library='pyinterp')
```

    15.9 s Â± 2.24 s per loop (mean Â± std. dev. of 7 runs, 1 loop each)
    

```python
%%capture --no-stdout
%%timeit 

reprojector = Reprojector()
ds_reproj = reprojector.reproject(native_fp, reproj_library='pyresample')
```

    9.24 s Â± 1.18 s per loop (mean Â± std. dev. of 7 runs, 1 loop each)
    

```python
%%capture --no-stdout

ds_reproj = reprojector.reproject(native_fp)

# Plotting
fig = plt.figure(dpi=250, figsize=(10, 10))
ax = plt.axes(projection=ccrs.TransverseMercator())

ds_reproj['stacked_eumetsat_data'].sel(variable='HRV').plot.imshow(ax=ax, cmap='Greys_r')

ax.coastlines(resolution='50m', alpha=0.8, color='white')
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-41-4aa2b08f07bf> in <module>
    ----> 1 ds_reproj = reprojector.reproject(native_fp)
          2 
          3 # Plotting
          4 fig = plt.figure(dpi=250, figsize=(10, 10))
          5 ax = plt.axes(projection=ccrs.TransverseMercator())
    

    NameError: name 'reprojector' is not defined

