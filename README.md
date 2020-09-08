# Building Lifespan Prediction

## Summary

This project uses regression analysis to solve the problem of disposable buildings, or usage of buildings for a short life cycle. To reduce waste and increase ROI, building owners want to extend the lifespan. The potential users who would benefit from this analysis include long-term building owners such as real estate corporations and government agencies. 

This analysis studies demolished commercial buildings and predicts building lifespan using linear regression. 


## Data

[NYC Property Land Use Data PLUTO](https://aws.amazon.com/marketplace/pp/NYC-Property-Land-Use-Data-Pluto/prodview-hj454d7mz5b6s#overview)

The PLUTO data includes a wide variety of information about tax lots in New York City, such as zoning information, lot characteristics, architectural features, geolocation data, and information about the owner, building, and construction date. The regression model uses features from this dataset to predict the target.

[NYC Open Data Permit Issuance](https://data.cityofnewyork.us/Housing-Development/DOB-Permit-Issuance/ipu4-2q9a)

The NYC Permit data includes construction permit and geolocation data. By sorting the permit data to exclude new construction, only demolition permits remain. Once the permit data is turned into a list of only demolished buildings, it is then combined with the PLUTO data using bbl number as a method of sorting the PLUTO data to only demolished buildings. BBL is a NYC land use geolocation method which stands for Borough, Block, and Lot. For example, a building may be in the Borough of Manhattan, block 4028, lot 29. Coding the boroughs 1-5 and combining borough, block, and lot results in the bbl number.


## Process

The process follows a CRISP-DM sequence including Business Understanding, Data Understanding, Data Preparation, Modeling and Evaluation. 


## Files

Readme.md
Start with this file.

Cap_Presentation.pdf\
Slide deck describing the project. Review presentation for an overview before diving into the technical notebooks.

Technical-Notebook.ipynb\
Code for entire project summarized into one notebook.

01_Cleaning_PLUTO.ipynb\
Code for cleaning PLUTO data.

01_Cleaning_Permits.ipynb\
Code for cleaning Permits data.

02_EDA.ipynb\
Code for Exploratory Data analysis.

03_Modeling.ipynb\
Code for Linear Regression Modeling.

Images folder\
Graphics for readme file.

Data folder\
Data dictionary for PLUTO data. For original data files, download from links above.


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


**Final Features Used for Modeling Include:**

These features describe zoning characteristics:
- ltdheight 
- overlay1
- spdist1 
- spdist2 
- zonedist1 
- zonedist2
- zonedist3 
- splitzone 

These features describe lot characteristics:
- irrlotcode
- lotarea 

These features describe architectural characteristics:
- bldgfront 
- ext 
- histdist 

These features relate to the type of building, owner, and construction date:
- bldgclass 
- ownertype 
- yearbuilt 

For more detailed feature descriptions, refer to Technical-Notebook. 


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

Exploratory Data Analysis looked at the following:

**Outliers:**\
Several outliers were in the lot area column. They were determined not to be errors, so they were kept.

**Distribution of Target Variable**\
The distribution of the target variable is not normal. Instead, it has two peaks - one at the 35 year mark, and one at the 85 year mark. The mean building lifespan is 51 years, right inbetween the two peaks. This reflects reality as there are many older buildings that last 70-90 years, and many newer buildings that last 25-40 years. This results from the poorer overall building quality of newer buildings and their material compared to older buildings.  

![Lifespan distribution](https://github.com/clairmarie8/Capstone-Project/blob/master/images/viz_lifespan_3.png?raw=true)

**Relationship Between Lifespan to Year Built**\
It was observed that lifespan and year built have a strongly linear relationship. The reason for the thickness of the band is that the average lifespan is 50 years. Buildngs that are not renovated around this time span tend to be demolished. The sharp edge at the 50 year mark most likely reflects schedules that require buildings to be inspected at certain age marks, which result in building owners becoming aware of problems with a building that may result in the decisiont to demolish the building. 

![Lifespan to Year Built](https://github.com/clairmarie8/Capstone-Project/blob/master/images/viz_lifespan+yearbuilt.png?raw=true)

**Building Types Demolished**\
Looking at the bldgtype feature reveals that a wide variety of buildings are demolished. The top types include multifamily residential and general commercial buildings. There are also many retail and warehouse buildings demolished.

![Building Types Demolished](https://github.com/clairmarie8/Capstone-Project/blob/master/images/viz_bldgtypes-demo.png?raw=true)

**Lot Types**

Analysis of the lottype variable reveals that the majority of demolitions occur on inside lots. This is likely because smaller lots tend to be aggregated with nearby lots in order to create larger buildings. Larger buildings have an economy of scale which results in a lower cost per square foot for construction, increasing the ROI of a developer's initial investment. The 5% other lots includes unique lots such as waterfront, which tend to be highly desireable and therefore less likely to result in demolition.

![Lot Types](https://github.com/clairmarie8/Capstone-Project/blob/master/images/viz_lots.jpg?raw=true)

## Modeling

First, continuous and categorical variables are split. 

Train test split with a 33% test size is done for each model. 


**Model 1A - Linear Regression with Continuous Features**\
This model uses only continuous variables. Results included R-Squared of 0.90, and an RMSE of 9.13 indicating that predictions are accurate within plus or minus 9 years. This degree of precision is useful for the business problem. Cross validation confirmed the results were within an acceptable range. The residuals plot revealed some patterns, so modeling continued.

Next, the same input was fed into statsmodels OLS to verify the results, which yielded essentially the same results except for slightly different p-values. Several variabes had p-values over 0.05, which may not be statistically significant. These variables were removed from the model in later iterations. 

**Model 2 - Linear Regression with Continuous & Categorical Features**\
This model used both continuous & categorical variables and achieved an R-Squared of 0.909, and an RMSE of 8.7. Three variables with high p-values were removed. Cross-validation confired the results. The residuals plot still showed a pattern. 

**Model 3A - Linear Regression with All Categorical Features**\
This model used all categorical features from the PLUTO Dataset. Only features determined to be not-relevant based on business understanding were omitted. Refer to Technical-Notebook.ipynb for a description of why individual features were omitted. 

This model had an R-Squared of  0.919 accuracy and an RMSE of 8.27. There were many columns with large p-values. 

**Model 3B - Linear Regression with Maximum Features**\
This model used all categorical features from the PLUTO Dataset as well as the three features earlier identified as having high-pvalues. 

This model had an R-Squared of 0.930 and an RMSE of 7.6. This was the highest performing model. 

**Model 3C - Linear Regression with Limited Features**\
This model used only the features with a p-value of less than 0.05 and achieved an R-Squared of 0.91 and RMSE of 8.59.

Since all of the above models had patterns in the residuals plots, interactions and polynomial terms were explored.

**Model 4 - Linear Regression with Interactions**\
This model resulted in an extremely high RMSE of 276, rendering it unusable. 

**Model 5 - Polynomial Regression Degree 2**\
This model had a very strong pattern in the residuals plot, even stronger than the other models. 

**Conclusion**\
Model 3B was the highest performing model. The features used in this model were:\
lotarea\
strgearea\
factryarea\
bldgfront\
assesstot\
yearbuilt\
irrlotcode_1\
overlay1\
splitzone\
ownertype\
ext_1\
zonedist3\
spdist2\
ltdheight\
histdist\
bldgclass\
spdist1_1\
zonedist1\
zonedist2


## Results

**Interpretation of Coefficients**

**Lot Area**\
lotarea 0.0000019327, slight positive relationship.
For every 10,000 SF of lot area, lifespan goes up by one hundreth of a year.

**Storage Area**\
strgearea -0.000014582, slight negative relationship.
For every 10,000 SF of storage area, lifespan goes down by 0.14 year.
This is likely because buildings with storage use have poor architectural characteristics, otherwise they would be used for a more profitable function.

**Factory Area**\
strgearea 0.00001219, slight positive relationship
For every 10,000 SF of factory area, lifespan goes down up 0.12 year.
This is most likely because factory buildings are structurally robust and can easily be converted to other building types.

**Building Frontage**\
bldgfront, 0.0215, positive relationship 
For every 100 feet of building frontage, lifespan goes up by 2 years. 

**Assessed Total Lot Value**\
'assesstot': -3.17e-08\
For every $10M assessed lot value, lifespan goes down by 0.3 year or roughly 3.5 months.

**Year Built**\
'yearbuilt': -1.0118, negative relationship\
For every year younger that a building is, lifespan goes down by 1 year.
This results from reductions in construction quality, materials and structural integrity as economic forces have pressured buildings to have a lower intial construction cost.

**Building Class**

bldgclass_1: Multifamily,  -3.84 
Multifamily buildings have a lifespan of roughly 4 years less than other buildings, all other factors considered equal. 

bldgclass_2: Garage, -5.24
Commercial garages have a lifespan of roughly 5 years less than other buildings, all other factors considered equal. 

bldgclass_3: Educational, -4.09 
Educational garages have a lifespan of roughly 4 years less than other buildings, all other factors considered equal. 

bldgclass_4: Mixed Use,  -1.06 
Mixed use buildings have a lifespan of roughly 1 years less than other buildings, all other factors considered equal. 

bldgclass_5: Office, -1.67
Office buildings have a lifespan of roughly 2 years less than other buildings, all other factors considered equal. 

bldgclass_6: Transportation, -2.13 
Transportation buildings have a lifespan of roughly 2 years less than other buildings, all other factors considered equal. 

bldgclass_7: Assembly, -3.10
Assembly buildings have a lifespan of roughly 3 years less than other buildings, all other factors considered equal. 

bldgclass_8: Loft, 0.47
Loft buildings have a lifespan of roughly a half year longer than other buildings, all other factors considered equal. Loft buildings are flexible-use structures with open floor plans and high ceilings. These buildings can easily be converted to another use.

bldgclass_9: Utility, 2.73
Utility buildings have a lifespan of roughly 3 years longer than other buildings, all other factors considered equal. Utility Buildings include structures for utility infrastructure such as electrical substations.

bldgclass_10: Hospitality, -1.8 
Hospitality buildings have a lifespan of roughly 2 years less than other buildings, all other factors considered equal. 

bldgclass_11: Theatres, -7.24
Theatres have a lifespan of roughly 7 years less than other buildings, all other factors considered equal. This most likely results from theatres having little natural light and floor plans that are difficult to convert to other uses. 

bldgclass_12: Government, -9.03 
Government buildings have a lifespan of roughly 9 years less than other buildings, all other factors considered equal. This result is surprising and may warrant futher exploration. 

**Irregular Lot Shape**\
irrlotcode_1: 0.79
Irrlotcode values are 0 for not being irreguarly shaped, and 1 for being irregularly shaped.
The interpretation of this feature depends on the shape of the individual lot. For lots whose irregular shape impedes light, view or the potential to expand, this would most likely reduce lifespan of the building. For lots with the opposite effect, this would most likely increase lifespan of the building.   

**Overlay**

This coefficient refers to lots which had an overlay zone. This means there are additional zoning requirements on top of the base zone. In most cases, lots with overlay zones increased the longevity of the building by up to 4 years.

overlay1_1: C2-4, 3.97  
overlay1_2: C2-3, 2.32\
overlay1_3: C1-3, 3.18\
overlay1_4: C2-2, 1.64\
overlay1_5: C1-4, 3.91\
overlay1_6: C1-2, -2.73\
overlay1_7: C1-1, -1.77\
overlay1_8: C2-1, -0.23\
overlay1_9: C1-5, 2.13\
overlay1_10: C2-5, -0.55

**Split Zone**\
This coefficient refers to lots which had requirements for more than one zone. Any form of split zoning reduced building lifespan.\
splitzone_1: -3.71\
splitzone_2: -4.11

**Zoning District 3**\
Lots with a third zone had a 2.5 year shorter lifespan.\
zonedist3_1: -2.51 

**Owner Type**\
This coefficient showd that lots with mixed city and private ownership had a very strong negative impact on lifespan, reducing lifespan by more than 15 years. Most likely, this resulted from disputes over how the property should be used or developed.\
ownertype_1: Tax-Exempt Property, -1.26\
ownertype_2: City Ownership, -4.99\
ownertype_3: Mixed City and Private Ownership, -15.64

**Special Districts**\
Lots with special districts had a variety of relationships to lifespan, some positive and some negative. This depends on the specific zoning regulations of each special district.

spdist1_1: 1.38\
spdist1_2: 1.62\
spdist1_3: 1.06\
spdist1_4: -0.80\
spdist1_5: -0.55\
spdist1_6: 0.98\
spdist1_7: -0.71\
spdist1_8: 5.79\
spdist1_10: 0.049

There was only one lot with a second special district, the 125th Street special district. This resulted in a 9-year shorter lifespan. Most likely, the city was encouraging construction in this special district to increase commercial development.\
spdist2_1: -9.2 

**Historic Districts**\
Lots in a historic district had a lifespan 2 years longer than other buildings, all else considered equal. This is surprisingly low.\
histdist_1: 2.23 

**Limited Height Districts**\
Lots in a limited height district had a lifespan 1 year shorter than other buildings, all else considered equal.\
ltdheight_1: -1.15


## Improving the Model

Ways to improve the model include identifying what went wrong in the interactions and polynomial regression, and using further EDA to decode the patterns in the residual plot, and deeper analysis of complex coefficients. 


## Recommendations

The recommendations for this analysis affect both urban planning and design. 

Urban Planning:
Urban planners and policymakers can encourage longer lifepans by doing the following:
- Plan fewer small lots where possible
- Increase Use of overlay zones
- Eliminate split-zone lots
- Anticipate demolitions in areas with rapidly rising assessed land values.
- Eliminate mixed-ownership properties.

Design: 
Architects can increase building lifespan with these design interventions:
- Prioritize Street Frontage
- Design new buildings with some of the architectural characteristics of older buildings.
- Encourage design of loft-style structures.










