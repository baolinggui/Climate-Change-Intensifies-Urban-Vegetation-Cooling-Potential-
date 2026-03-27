Steps 1-3 pertain to data construction, which is relatively complex. I first constructed a collection of ERA5 re-climate data and other variables (a dataset format suitable for training) (GlobalUrbanCooling_dataset_v2.parquet). 
Then, based on this dataset, I aggregated the MERRA-2 data into the aforementioned dataset to form GlobalUrbanCooling_dataset_v2_ERA5_plus_MERRA2hybrid.parquet, which can be used for comparative analysis. 
This data construction is relatively complicated; you can also try to construct it yourself. Alternatively, you can directly obtain these two datasets and perform the processing described in steps 4-6 below to obtain the results of this project. Below is the link to obtain the data:
https://figshare.com/s/b3eba93a87f51c558a39
