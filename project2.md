import pandas as pd
import numpy as np


# Load data
pwt90 = pd.read_stata('https://www.rug.nl/ggdc/docs/pwt90.dta')
pwt1001 = pd.read_stata('https://dataverse.nl/api/access/datafile/354098')

# Filter and select relevant columns
data = pwt90.loc[pwt90['country'].isin(["France", "Germany","Canada","Italy","Japan","United Kingdom","United States"])][['year', 'countrycode', 'rgdpna', 'rkna', 'emp', 'labsh']]
data = data.loc[(data['year'] >= 1995) & (data['year'] <= 2019)].dropna()

# Calculate additional columns
data['y_pc'] = np.log(data['rgdpna'] / data['emp'])  # GDP per worker
data['k_pc'] = np.log(data['rkna'] / data['emp'])  # Capital per worker
data['a'] = 1 - data['labsh']  # Capital share


# Order by year
data = data.sort_values('year')

# Group by isocode
grouped_data = data.groupby('countrycode')



# Calculate growth rates and Solow residual
data['g'] = grouped_data['y_pc'].diff() * 100  # Growth rate of GDP per capita

data['c_d'] = grouped_data['k_pc'].diff()* 100* data['a']
data['T_g'] = data['g'] - data['c_d']



# Remove missing values
data = data.dropna()




# Calculate summary statistics
summary = data.groupby('countrycode').agg({'g': 'mean',
                                           'T_g': 'mean',
                                           'c_d': 'mean',
                                           'a': 'mean'})



# Rename columns
summary.rename(columns={'g': 'Growth Rate',
                        'T_g': 'TFP Growth',
                        'c_d': 'Capital Deepening',
                         'a': 'Capital Share'}, inplace=True)

# Create a new row for average values
average_row = pd.DataFrame({'Growth Rate': average_growth_rate,
                            'TFP Growth': average_tfp_growth,
                            'Capital Deepening': average_capital_deepening,
                            'Capital Share': average_capital_share}, index=['Average'])

# Concatenate the average row with the summary table
summary_with_average = pd.concat([summary, average_row])

# Print
print(summary_with_average)



