# Building Lifespan Prediction

## Summary

This project uses regression analysis to solve the problem of disposable buildings, or usage of buildings for a short life cycle. To reduce waste and increase ROI, building owners want to extend the lifespan. The potential users who would benefit from this analysis include long-term building owners such as real estate corporations and government agencies. 

This analysis studies demolished commercial buildings and predicts building lifespan using linear regression. Polynomial Regression and a decision tree were attempted, however the best results came from linear regression.

## Description



## Data

[NYC Property Land Use Data PLUTO](https://aws.amazon.com/marketplace/pp/NYC-Property-Land-Use-Data-Pluto/prodview-hj454d7mz5b6s#overview)

[NYC Open Data Permit Issuance](https://data.cityofnewyork.us/Housing-Development/DOB-Permit-Issuance/ipu4-2q9a)

## Process

The process follows a CRISP-DM sequence including Business Understanding, Data Understanding, Data Preparation, Modeling and Evaluation. 

## Data Cleaning

**Data cleaning of the NYC PLUTO data included the following steps:**

1. Filter out 1 and 2 family dwellings in Land use column.
2. Filter numbldgs column to include only lots with one building.
3. Drop masdate, polidate for having too many missing values.
4. Drop the following columns beause they are not relevant:
        - zmcode
        - condono
        - owner name
        - ct2010
        - tract2010
        - cb2010
        - firecomp
        - sanborn
        - sanitsub
        - address
        - areasource
        - zonemap
        - taxmap
        - appbbl
        - appdate
        - plutomapid
        - rpaddate
        - dcasdate
        - edesigdate
        - geom
        - zipcode
        
5. Drop the following columns because they are redundant with another column.
        - sanitboro
        - Borough
        - version

6. Drop rows with missing values in the yearbuilt column.

**Data cleaning of the Permits Data included the following steps:**

1. Create Issuance Year column from Issuance Date column.
2. Remove NAN values from Issuance Date.
3. Filter by 'Bldg Type' column to remove residential properties.
4. Add column for borough code, for the purpose of making a bbl column (borough, block and lot) so that the dataframe can be merged with lot data.
5. Change the number of digits of the Block column to 5.
6. Change the number of digits of the Lot column to 4.
7. Make a new column for bbl or borough, block and lot, concatenating the 3 columns.



**Data cleaning of the Merged Permits/PLUTO data included the following steps:**

1. Remove lots where a new building was built after the demolition permit issuance date.
2. Drop the following columns:
        - Block, Lot, Borough Code, bbl (not needed after the merge)
        - Unnamed:0 because this column is not listed in data dictionary
        - date columns 'zoningdate', 'landmkdate' 'basempdate' (not relevant)
        - 'numbldgs' because the data was filtered to lots that have 1 building. 
3. Address missing values (see technical notebook for more details)
4. Make lifespan feature using 'yearbuilt' and 'Issuance Year'
5. Make correlation heatmaps to see correlation levels. Remove the following variables because of multicolinarity, missing data, or because they are used for feature engineering but not for prediction: 
    'borocode', 'cd', 'schooldist', 'council', 'assessland', 'exempttot', 
    'unitsres', 'comarea', 'resarea', 'officearea', 'residfar', 'commfar', 'facilfar', 
    'pfirm15_flag', 'firm07_flag', 'spdist3', 'lotfront', 'unitstotal', 
    'bldgarea', 'builtfar',
    'numfloors', 'yearalter1', 'yearalter2', 'garagearea', 'easements', 'bldgdepth', 
    'lotdepth','zonedist4', 'spdist3', 'overlay2', 'existbldg', 'Issuance Year', 
    'retailarea', 'otherarea'
6. Make a second heatmap and remove bsmtcode, proxcode for multicolinearity. Remove edesignum because it is not relevant. Remove Issuance date because it was only needed to make lifespan column. Remove landuse because it is redundant with bldgclass (see Data Dictionary.)
7. Separate independent variables into categorical & continuous variables.

## Analysis

First do Exploratory Data Analysis.

### Modeling

Model continuous variables first, evaluate results then model using categorical variables.

2 linear regression models were made using 2 different methods to double check results. Both models yielded an R-squared value of 0.90. 

Polynomial Regression and a decision tree were done, however the results were not as good as linear regression. 


### Improving the Model

Some of the predictors had a p-value of over 0.05, so these variables will be removed and categorical variables inserted to replace them. 


### Results








