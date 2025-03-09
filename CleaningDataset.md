# Cleaning the House Prices dataset
## Imports
```python
import pandas as pd
import matplotlib as plt
```
## 1. Initial Checks
First, lets have a look at a snapshot of our data:
```python
df = pd.read_csv('./housing_data/Housing.csv')
print(df)
```
### Output
```
               id             date  ...  sqft_living15  sqft_lot15
0      7229300521  20141013T000000  ...           1340        5650
1      6414100192  20141209T000000  ...           1690        7639
2      5631500400  20150225T000000  ...           2720        8062
3      2487200875  20141209T000000  ...           1360        5000
4      1954400510  20150218T000000  ...           1800        7503
...           ...              ...  ...            ...         ...
21608   263000018  20140521T000000  ...           1530        1509
21609  6600060120  20150223T000000  ...           1830        7200
21610  1523300141  20140623T000000  ...           1020        2007
21611   291310100  20150116T000000  ...           1410        1287
21612  1523300157  20141015T000000  ...           1020        1357

[21613 rows x 21 columns]
```
We can see we have 21613 rows of data and 21 columns to work with. The column attributes are:
```python
print(df.columns)
```
### Output
```
Index(['id', 'date', 'price', 'bedrooms', 'bathrooms', 'sqft_living', 'sqft_lot', 'floors', 'waterfront', 'view', 'condition', 'grade', 'sqft_above', 'sqft_basement', 'yr_built', 'yr_renovated', 'zipcode', 'lat', 'long', 'sqft_living15', 'sqft_lot15'], dtype='object')
```
Some attributes to note are:
* waterfront - Indicates if the property has a waterfront view (0 = no, 1 = yes)
* view - A score from 0 to 4 of the quality of the view from the property
* condition - Overall condition rating from 1 to 5
* grade - Overall grade rating from 1 to 13
* yr_renovated - The year the property was last renovated (0 if not renovated)
* sqft_living15 - Living area size of 15 nearest properties in square feet
* sqft_lot15 - Lot size of 15 nearest properties in square feet

## 2. Reformatting and Small Changes
### id
The id value is a unique identifier for each property, so this can help us find out how many different properties were actually recorded in the sample.
```python
print(df['id'].nunique())
```
### Output
```
21436
```
This means we have only 177 repeat entries (where a property already recorded had an additional record made.) This is okay as the only benefit of having multiple surveys would be to assess the impact of the date on the price, which we are not looking into as much in this study.

### date
Its important that the date values are in a workable format for the analysis, ideally in the 'datetime' data type. Right now however it is an object type. We can ignore the time part of the data since the dataset has treated the time as 00:00:00, so to change the type to datetime we just need it to be in a format like DDMMYYYY. This requires us to get rid of the "T000000" part at the end of each value. Fortunately this is easy to do by simply cutting off the last 7 characters of each value:
```python
df['date'] = df['date'].str[:8]
df['date'] = pd.to_datetime(df['date'], format='%Y%m%d')
print(df)
```
### Output
```
               id       date     price  ...     long  sqft_living15  sqft_lot15
0      7229300521 2014-10-13  231300.0  ... -122.257           1340        5650
1      6414100192 2014-12-09  538000.0  ... -122.319           1690        7639
2      5631500400 2015-02-25  180000.0  ... -122.233           2720        8062
3      2487200875 2014-12-09  604000.0  ... -122.393           1360        5000
4      1954400510 2015-02-18  510000.0  ... -122.045           1800        7503
...           ...        ...       ...  ...      ...            ...         ...
21608   263000018 2014-05-21  360000.0  ... -122.346           1530        1509
21609  6600060120 2015-02-23  400000.0  ... -122.362           1830        7200
21610  1523300141 2014-06-23  402101.0  ... -122.299           1020        2007
21611   291310100 2015-01-16  400000.0  ... -122.069           1410        1287
21612  1523300157 2014-10-15  325000.0  ... -122.299           1020        1357

[21613 rows x 21 columns]
```
### Decimal values where integers are expected
If we have a look at the values for 'floors', we can see some values that are not integers as we would expect. In this case, quite a few values have 'half' of a floor:
```python
print(df['floors'][12:18])
```
### Output
```
12    1.5
13    1.0
14    1.5
15    2.0
16    2.0
17    1.5
Name: floors, dtype: float64
```
The exact meaning for these values wasn't stated by the publisher of the dataset, although it could be interpretted as an attic/basement floor which wouldn't be equivalent to an additional story to a house (therefore this value differentiates between them.) The data does also contain a basement area value, so we should comparre it to this:
```python
print(df['sqft_basement'][12:18])
```
### Output
```
12      0
13      0
14      0
15    970
16      0
17      0
Name: sqft_basement, dtype: int64
```
For all of the values shown before with a decimal value, we see there is no basement on the property. Now we should check if this means there is an attic on the property. To do this, note that a value of 1.5 means that if there is no attic or second floor then the value of 'sqft_above' (Living area above ground level in square feet) should be zero:
```python
print(df['sqft_above'][12:18])
```
### Output
```
12    1430
13    1370
14    1810
15    1980
16    1890
17    1600
Name: sqft_above, dtype: int64
```
However, the values are not zero and are in fact the same as the values for 'sqft_living'. This does raise another problem though, data value 13 has only 1 recorded floor but also has a value of sqft_above that isnt zero.
