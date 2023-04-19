# DataCleaningProjectUS_Census
Data Cleaning Project
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import glob


#using glob library and regex * to read all of the states.csv file
#creating a data frame using list which contain a loop to read all of the files
#using concate to concatenate/merge all of the csv into one 

us_sensus= glob.glob("states*.csv")

df_list = []

for filename in us_sensus:
  df_list.append(pd.read_csv(filename))
  
us_sensus = pd.concat(df_list)

#printing the first five rows
#printing the total row of the data

print(us_sensus.head())
print(len(us_sensus))


#printing the data types to see if the data is plotable
#printing the columns name

print(us_sensus.dtypes)
print(us_sensus.columns)

# remove % signs and convert to numeric type for multiple columns
cols_to_convert = ['Hispanic', 'White','Black', 'Native', 'Asian', 'Pacific']

for col in cols_to_convert:
    us_sensus[col] = us_sensus[col].replace('%', '', regex=True)
    us_sensus[col] = pd.to_numeric(us_sensus[col])

# remove $ signs from Income column
us_sensus['Income'] = us_sensus['Income'].replace('[\$,]', '', regex=True)
us_sensus['Income'] = pd.to_numeric(us_sensus['Income'])

#separating GenderPop column into male and female
us_sensus[['MalePop', 'FemalePop']] = us_sensus['GenderPop'].str.split('_', expand=True)

#removing the string M and F from the column to make the data plotable
us_sensus.MalePop = us_sensus['MalePop'].replace('M', '', regex=True)
us_sensus.FemalePop = us_sensus['FemalePop'].replace('F', '', regex=True)

#converting the sring data in MalePop and FemalePop into numerical data
us_sensus.MalePop = pd.to_numeric(us_sensus['MalePop'])
us_sensus.FemalePop = pd.to_numeric(us_sensus['FemalePop'])  

# convert the State column to categorical codes
us_sensus['StateCode'] = pd.Categorical(us_sensus['State']).codes + 1

# drop the 'Unnamed' and GenderPop column because it's unnecesary
us_sensus = us_sensus.drop('GenderPop', axis=1)
us_sensus = us_sensus.drop(us_sensus.columns[0], axis=1)


# I want to move StateCode from furthest right to next to State column 
# using df.pop() to move and remove it from the dataframe
# then inserting the column at the desired index. Index 1 is the second.
col_to_move = us_sensus.pop('StateCode')
us_sensus.insert(1, 'StateCode', col_to_move)

#printing the had and data type to confirm. Yes data is now plotable
print(us_sensus.head())
print(us_sensus.dtypes)



#checking if there is any nan value

for col in us_sensus:
    has_nan = us_sensus[col].isnull().values.any()
    if has_nan:
        print(f'Column {col} contains NaN values.')
        
# we know now column Pacific and FemalePop has nan value.
# those column are the percentage, so to fill it we just need to
# subtract from it's data counterpart

us_sensus['FemalePop'] = us_sensus['FemalePop'].fillna(us_sensus['TotalPop'] - us_sensus['MalePop'])

us_sensus['Pacific'] = us_sensus['Pacific'].fillna((100 - (us_sensus['Hispanic'] + us_sensus['White'] +
                        us_sensus['Black'] + us_sensus['Native'] + us_sensus['Asian'])))
us_sensus['Pacific'] = us_sensus['Pacific'].round(2)

#checking if there is any duplicates    
        
for col in us_sensus:
    duplicates = us_sensus.duplicated(subset=col)
    print(f"Duplicates in {col}: {sum(duplicates)}")    

# dropping the duplicates. The value can be a duplicate but a category which is 'State' can not
# be a duplicate. Hence needs to be dropped
us_sensus = us_sensus.drop_duplicates(subset=['State'])

#creating csv files
us_sensus.to_csv('cleaned_us_sensus.csv', index=False)
