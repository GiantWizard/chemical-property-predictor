# Documentation here
---

## 11/14/2025

### Test results - 2D:

#### Melting Point:  
RMSE: 282.07  
R2: 0.44  
#### Boiling point:  
RMSE: 234.01  
R2: 0.81  
#### Heat of Fusion:  
RMSE: 51530.83  
R2: 0.44
#### Heat of Vaporization:  
RMSE: 92181.32  
R2: 0.82  
#### Critical Temperature:  
RMSE: 30.75  
R2: 0.92  
#### Critical Pressure:   
RMSE: 1248094.49  
R2: 0.72  
#### Flash Point:  
RMSE: 20.98  
R2: 0.90  

### Test results - 3D:

#### Melting Point:  
RMSE: 273.43  
R2: 0.38  
#### Boiling point:  
RMSE: 39.85  
R2: 0.87  
#### Heat of Fusion:   
RMSE: 54104.68  
R2: 0.42  
#### Heat of Vaporization:  
RMSE: 90210.16  
R2: 0.82  
#### Critical Temperature:  
RMSE: 30.27  
R2: 0.93  
#### Critical Pressure:   
RMSE: 1201639.53  
R2: 0.74  
#### Flash Point:  
RMSE: 23.05  
R2: 0.88  

### Test results - Combined:

#### Melting Point:  
RMSE: 43.05  
R2: 0.69  
#### Boiling point:  
RMSE: 37.46  
R2: 0.89  
#### Heat of Fusion:   
RMSE: 49104.21  
R2: 0.52  
#### Heat of Vaporization:  
RMSE: 97416.51  
R2: 0.79  
#### Critical Temperature:  
RMSE: 32.81  
R2: 0.91  
#### Critical Pressure:  
RMSE: 1178134.01  
R2: 0.75  
#### Flash Point:  
RMSE: 21.35  
R2: 0.90  

## 11/16/2025

Switched RMSE to MAPE
After performing data cleaning steps, such as removing 0 variation features, removing non-organic substances, and removing outliers via the IQR method, R2 and MAPE results are as shown:

#### Melting Point:  
MAPE: 0.14  
R2: 0.63  
#### Boiling point:  
MAPE: 0.10  
R2: 0.86   
#### Heat of Fusion:   
MAPE: 0.10  
R2: 0.79  
#### Heat of Vaporization:  
MAPE: 0.13  
R2: 0.82  
#### Critical Temperature:  
MAPE: 0.05  
R2: 0.92  
#### Critical Pressure:  
MAPE: 0.09  
R2: 0.91  
#### Flash Point:  
MAPE: 0.06  
R2: 0.81  

As you can see the model is starting to have some actual predictive power.

Correction: the melting point and flash point R2 figures above were from an intermediate run. The final reported figures for these two properties are R2: 0.674 (melting point) and R2: 0.843 (flash point), as reflected in the README results table.
