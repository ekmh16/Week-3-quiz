# Week-3-quiz
import pandas as pd
import numpy as np
energy = pd.read_excel('Energy Indicators.xls')
energy = energy[16:243]
energy = energy.drop(energy.columns[[0, 1]], axis=1)
energy.rename(columns={'Environmental Indicators: Energy': 'Country','Unnamed: 3':'Energy Supply','Unnamed: 4':'Energy Supply per Capita','Unnamed: 5':'% Renewable'}, inplace=True)
energy.replace('...', np.nan,inplace = True)
energy['Energy Supply'] *= 1000000
def remove_digit(data):
    newData = ''.join([i for i in data if not i.isdigit()])
    i = newData.find('(')
    if i>-1: newData = newData[:i]
    return newData.strip()
energy['Country'] = energy['Country'].apply(remove_digit)
di = {"Republic of Korea": "South Korea",
"United States of America": "United States",
"United Kingdom of Great Britain and Northern Ireland": "United Kingdom",
"China, Hong Kong Special Administrative Region": "Hong Kong"}
energy.replace({"Country": di},inplace = True)

GDP = pd.read_csv('world_bank.csv', skiprows=4)
GDP.rename(columns={'Country Name': 'Country'}, inplace=True)
di = {"Korea, Rep.": "South Korea", 
"Iran, Islamic Rep.": "Iran",
"Hong Kong SAR, China": "Hong Kong"}
GDP.replace({"Country": di},inplace = True)

ScimEn = pd.read_excel('scimagojr-3.xlsx')
df = pd.merge(pd.merge(energy, GDP, on='Country'), ScimEn, on='Country')

# We only need 2006-2015 data
df.set_index('Country',inplace=True)
df = df[['Rank', 'Documents', 'Citable documents', 'Citations', 'Self-citations', 'Citations per document', 'H index', 'Energy Supply', 'Energy Supply per Capita', '% Renewable', '2006', '2007', '2008', '2009', '2010', '2011', '2012', '2013', '2014', '2015']]
df = (df.loc[df['Rank'].isin([1,2,3,4,5,6,7,8,9,10,11,12,13,14,15])])
df.sort('Rank',inplace=True)

def answer_one():
    x = pd.ExcelFile('Energy Indicators.xls')
    energy = x.parse(skiprows=17,skip_footer=(38))
    energy = energy[[1,3,4,5]]
    energy.columns = ['Country', 'Energy Supply', 'Energy Supply per Capita', '% Renewable']
    energy[['Energy Supply', 'Energy Supply per Capita', '% Renewable']] =  energy[['Energy Supply', 'Energy Supply per Capita', '% Renewable']].replace('...',np.NaN).apply(pd.to_numeric)
    energy['Energy Supply'] = 1000000*energy['Energy Supply']
    energy['Country'] = energy['Country'].replace({'China, Hong Kong Special Administrative Region':'Hong Kong','United Kingdom of Great Britain and Northern Ireland':'United Kingdom','Republic of Korea':'South Korea','United States of America':'United States','Iran (Islamic Republic of)':'Iran'})
    energy['Country'] = energy['Country'].str.replace(r" \(.*\)","")

    GDP = pd.read_csv('world_bank.csv',skiprows=4)
    GDP['Country Name'] = GDP['Country Name'].replace({'Korea, Rep.':'South Korea','Iran, Islamic Rep.':'Iran','Hong Kong SAR, China':'Hong Kong'})
    GDP = GDP[['Country Name','2006','2007','2008','2009','2010','2011','2012','2013','2014','2015']]
    GDP.columns = ['Country','2006','2007','2008','2009','2010','2011','2012','2013','2014','2015']

    ScimEn = pd.read_excel(io='scimagojr-3.xlsx')
    ScimEn = ScimEn[:15]
    df = pd.merge(ScimEn,energy,how='inner',left_on='Country',right_on='Country')
    new_df = pd.merge(df,GDP,how='inner',left_on='Country',right_on='Country')
    new_df = new_df.set_index('Country')
    return new_df
answer_one()





def answer_two():
    # Union A, B, C - Intersection A, B, C
    union = pd.merge(pd.merge(energy, GDP, on='Country', how='outer'), ScimEn, on='Country', how='outer')
    intersect = pd.merge(pd.merge(energy, GDP, on='Country'), ScimEn, on='Country')
    return len(union)-len(intersect)

answer_two()




def answer_three():
    Top15 = answer_one()
    years = ['2006', '2007', '2008', '2009', '2010', '2011', '2012', '2013', '2014', '2015']
    return (Top15[years].mean(axis=1)).sort_values(ascending=False).rename('avgGDP')

answer_three()




def answer_four():
    Top15 = answer_one()
    Top15['avgGDP'] = answer_three()
    Top15.sort_values(by='avgGDP', inplace=True, ascending=False)
    return abs(Top15.iloc[5]['2015']-Top15.iloc[5]['2006'])

answer_four()







def answer_five():
    Top15 = answer_one()
    print()
    return Top15['Energy Supply per Capita'].mean()

answer_five()





def answer_six():
    Top15 = answer_one()
    ct = Top15.sort_values(by='% Renewable', ascending=False).iloc[0]
    return (ct.name, ct['% Renewable'])

answer_six()






def answer_seven():
    Top15 = answer_one()
    Top15['Citation_ratio'] = Top15['Self-citations']/Top15['Citations']
    ct = Top15.sort_values(by='Citation_ratio', ascending=False).iloc[0]
    return ct.name, ct['Citation_ratio']

answer_seven()





def answer_eight():
    Top15 = answer_one()
    Top15['Population'] = Top15['Energy Supply']/Top15['Energy Supply per Capita']
    return Top15.sort_values(by='Population', ascending=False).iloc[2].name

answer_eight()



