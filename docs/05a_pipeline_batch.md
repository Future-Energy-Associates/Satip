# Pipeline for backfilling / batching



Dagster can only run one pipeline per module, and `05_pipeline.ipynb` ie `mario.py` already has one pipeline defined for continuous linear retrieval where the steps take place one after the other.

<br>

### Imports

```python
#exports
import pandas as pd
import xarray as xr
import os
import glob
import dotenv
import warnings
from dagster import execute_pipeline, pipeline, solid, Field, OutputDefinition, DagsterType, Output
from itertools import islice
import shutil

from IPython.display import JSON

from satip import eumetsat, reproj, io, gcp_helpers
from satip.mario import (df_metadata_to_dt_to_fp_map, 
                         reproject_datasets, 
                         save_metadata, 
                         compress_and_save_datasets, 
                         compress_export_then_delete_raw)
```

```python
# Filter some warnings
#exports
warnings.filterwarnings('ignore', message='divide by zero encountered in true_divide')
warnings.filterwarnings('ignore', message='invalid value encountered in sin')
warnings.filterwarnings('ignore', message='invalid value encountered in cos')
warnings.filterwarnings('ignore', message='invalid value encountered in subtract')
warnings.filterwarnings('ignore', message='You will likely lose important projection information when converting to a PROJ string from another format. See: https://proj.org/faq.html#what-is-the-best-format-for-describing-coordinate-reference-systems')
```

```python
eumetsat_zarr_bucket='solar-pv-nowcasting-data/satellite/EUMETSAT/SEVIRI_RSS/zarr_full_extent_TM_int16'
missing_datasets = io.identifying_missing_datasets("2020-01-01 00:00", "2020-01-01 01:00", eumetsat_zarr_bucket=eumetsat_zarr_bucket)
JSON(missing_datasets)
```

    Earliest 2020-01-01 00:00, latest 2020-01-01 01:00
    


<div><span class="Text-label" style="display:inline-block; overflow:hidden; white-space:nowrap; text-overflow:ellipsis; min-width:0; max-width:15ex; vertical-align:middle; text-align:right"></span>
<progress style="width:60ex" max="1" value="1" class="Progress-main"/></progress>
<span class="Progress-label"><strong>100%</strong></span>
<span class="Iteration-label">1/1</span>
<span class="Time-label">[00:01<00:01, 1.38s/it]</span></div>


    identify_available_datasets: found 12 results from API
    




    <IPython.core.display.JSON object>



```python
#exports
def chunks(data, SIZE=10000):
    """Turn dict into iterator of length SIZE chunks"""
    it = iter(data)
    for i in range(0, len(data), SIZE):
        yield {k:data[k] for k in islice(it, SIZE)}
```

```python
#exports

# create pandas DataFrame type definition for Dagster
DataFrame = DagsterType(
    name="DataFrame",
    type_check_fn=lambda _, x: isinstance(x, pd.DataFrame),
)

@solid(output_defs=[OutputDefinition(name='df_new_metadata', dagster_type=DataFrame, is_required=False)])
def download_missing_eumetsat_files(context, env_vars_fp: str, data_dir: str, metadata_db_fp: str, debug_fp: str, table_id: str, project_id: str, start_date: str='', end_date: str=''):
    _ = dotenv.load_dotenv(env_vars_fp)
    dm = eumetsat.DownloadManager(os.environ.get('USER_KEY'), os.environ.get('USER_SECRET'), data_dir, metadata_db_fp, debug_fp, slack_webhook_url=os.environ.get('SLACK_WEBHOOK_URL'), slack_id=os.environ.get('SLACK_ID'))
    
    missing_datasets = io.identifying_missing_datasets(start_date, end_date)
    context.log.info(f"Missing data: {len(missing_datasets)}")
    
    df_new_metadata = dm.download_datasets(missing_datasets)

    # if df_new_metadata is None, pipeline will skip subsequent solids
    if df_new_metadata is None:
        context.log.info("*******************")
        context.log.info("Files already in zarr. Exiting.")
        context.log.info("*******************")
        return

    yield Output(df_new_metadata, 'df_new_metadata')
```

```python
#exports
@solid()
def reproject_compress_save_datasets_batch(_, datetime_to_filepath: dict, new_coords_fp: str, new_grid_fp: str, zarr_bucket: str, var_name: str='stacked_eumetsat_data'):
    """Batch up the reprojection and saving to zarr steps
    
    xAarray concat or some other processing step gives memory crashes beyond around 1hr of time range
    which is around 12 file items
    """
    # datetime_to_filepath -> batches
    batch_size = 10
    batches = [i for i in chunks(datetime_to_filepath, batch_size)]
    
    for batch in batches:
        reprojector = reproj.Reprojector(new_coords_fp, new_grid_fp)

        reprojected_dss = [
            (reprojector
             .reproject(filepath, reproj_library='pyresample')
             .pipe(io.add_constant_coord_to_da, 'time', pd.to_datetime(datetime))
            )
            for datetime, filepath 
            in batch.items()
        ]

        if len(reprojected_dss) > 0:
            ds_combined_reproj = xr.concat(reprojected_dss, 'time', coords='all', data_vars='all')
        else:
            print("compress_and_save_datasets: No new data to save to zarr")
            return

        # Compressing the datasets
        compressor = io.Compressor()

        var_name = var_name
        da_compressed = compressor.compress(ds_combined_reproj[var_name])

        # Saving to Zarr
        ds_compressed = io.save_da_to_zarr(da_compressed, zarr_bucket)

    # fine to just return last one, we are just checking it exists
    return ds_compressed


@solid()
def save_metadata_batch(context, ds_combined_compressed, df_new_metadata, table_id: str, project_id: str):
    if ds_combined_compressed is not None:
        if df_new_metadata.shape[0] > 0:
            gcp_helpers.write_metadata_to_gcp(df_new_metadata, table_id, project_id, append=True)
            context.log.info(f'{df_new_metadata.shape[0]} new metadata entries were added')
        else:
            context.log.info('No metadata was available to be added')     
    return True

@solid()
def compress_export_then_delete_raw_batch(context, ready_to_delete, data_dir: str, compressed_dir: str, BUCKET_NAME: str='solar-pv-nowcasting-data', PREFIX: str='satellite/EUMETSAT/SEVIRI_RSS/native/'):
    if ready_to_delete == True:
        eumetsat.compress_downloaded_files(data_dir=data_dir, compressed_dir=compressed_dir, log=context.log)
        eumetsat.upload_compressed_files(compressed_dir, BUCKET_NAME=BUCKET_NAME, PREFIX=PREFIX, log=None)
        
        for dir_ in [data_dir, compressed_dir]:
            context.log.info(f'Removing directory {dir_}')
            shutil.rmtree(dir_)
            os.mkdir(dir_) # recreate empty folder
```

```python
#exports
@pipeline
def download_missing_data_pipeline():  
    # Retrieving data, reprojecting, compressing, and saving to GCP
    df_new_metadata = download_missing_eumetsat_files()
    datetime_to_filepath = df_metadata_to_dt_to_fp_map(df_new_metadata)
    
    last_batch_compressed = reproject_compress_save_datasets_batch(datetime_to_filepath)
    
    ready_to_delete = save_metadata_batch(last_batch_compressed, df_new_metadata)
    compress_export_then_delete_raw_batch(ready_to_delete)
```

`datetime_to_filepath` is a dict, looking like:  
`{Timestamp(): str`  

```python
# e.g
# {Timestamp('2019-04-01 00:04:19.045000+0000', tz='UTC'): '../data/raw_bfill/MSG3-SEVI-MSG15-0100-NA-20190401000419.045000000Z-NA.nat'}
```

Test the configuration and execute the pipeline:
