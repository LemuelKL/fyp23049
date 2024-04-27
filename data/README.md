# FYP23049 - Data

This repository contains the code that is used to process and analyse data in the project "FYP23049 - Modelling of European Interconnectors for Sustainable Energy Integration".

## Obtaining data from ENTSO-E Transparency Platform

First, register an account on the ENTSO-E Transparency Platform: https://transparency.entsoe.eu/

Then, connect to their FTP server by following the instructions on https://transparency.entsoe.eu/content/static_content/Static%20content/knowledge%20base/SFTP-Transparency_Docs.html

Download the data and store them at root as such:
```
.project-root/
├── ActualTotalLoad_6.1.A/
    ├── 2014_12_ActualTotalLoad_6.1.A.csv
    ├── 2015_01_ActualTotalLoad_6.1.A.csv
    ├── ...
|-- AggregatedGenerationPerType_16.1.B_C/
    |-- ...
|-- DayAheadPrices_12.1.D/
    |-- ...
|-- PhysicalFlows_12.1.G/
    |-- ...
```

## PostgreSQL Database Setup

Install PostgreSQL on your machine and create a database named `euics`.

## Environment Setup

Create a file at project root named `.env` with the following content:
```
SQLALCHEMY_DATABASE_URI=postgresql://<username>:<password>@localhost:5432/euics
```

and replace `<username>` and `<password>` with your PostgreSQL username and password.

## Importing Data to Database

The notebook `import_sql.ipynb` is used to import the data from the CSV files to the PostgreSQL database.

There are 4 main cells in the notebook, each responsible for importing data from the respective data folder.

Note that the method that does the actual SQL writing `df.to_sql(...)` is commented out by default. This is to prevent accidental overwriting of the database. Uncomment the line to write to the database.

After this step, all the data will be stored in the PostgreSQL database.

## Data Processing

The data imported into the database contain a lot of unnecessary columns and rows. The notebook `process_sql.ipynb` is used created slimmed down tables that contain only the necessary columns and rows.

The notebook also does some data cleaning and processing to make the data more usable. That includes doing pivotting and aggregating for generation fuel types.


## Alternative

In earlier stage of the project, the Python package `entsoe-py` was used to download data from the ENTSO-E Transparency Platform. https://github.com/EnergieID/entsoe-py has a self-contained example of how to use the package to download data.

## entsoe.py

This script serves as an interface between the SQL database and any other models developed in the project. It pulls data from the SQL database and returns it as graph data structures.

It caches the data as `.npy` files to disk to prevent unnecessary querying of the database.

```python
...
np.save("data/datetime_intersect.npy", datetime_intersect)
np.save("data/node_features.npy", node_features)
np.save("data/edge_indices.npy", edge_indices)
np.save("data/edge_attributes.npy", edge_attributes)
np.save("data/edge_labels.npy", edge_labels)
...
```

These cached files provide an additional advantage where they can be transferred to other machines without needing the database. This is especially useful when running the models on say Google Colab.


## Data Analysis

The directory `analytic` contains the code used generate some analytic parts of the final report. R and RStudio are required to run the code. There you can find the code for one of the SVM used in the writing of the reports. There is also a notebook for generating some plots.

## SVR.ipynb

This notebook contains the code for the Support Vector Regression model used in the project. It uses the `scikit-learn` library to train and test the model.

This SVR model regresses one of the ten countries' day-ahead prices using the other nine countries' day-ahead prices as features.

It generates a plot for the actual vs predicted day-ahead prices each of the ten countries. The summary of the performance of the model is also printed to `benchmark.tex`.