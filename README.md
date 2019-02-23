# Replicating-Goyal-and-Welch-2008-
Replicate the analysis of : " A comprehensive look at the empirical performance of equity premium prediction (2008)
#############################
# Return forecasters
#
#############################

import pandas as pd
import numpy as np
import datetime as dt
import wrds
import psycopg2 
from dateutil.relativedelta import *
from pandas.tseries.offsets import *
from scipy import stats

# read the state variable file from Goyal's website

url = 'http://www.hec.unil.ch/agoyal/docs/PredictorData2017.xlsx'
    # If the file changes, use local copy
    #url = "..\\Python\\local\\PredictorData2017.xlsx"
gm = pd.read_excel(url, sheet_name='Monthly')
gq = pd.read_excel(url, sheet_name='Quarterly')
ga = pd.read_excel(url, sheet_name='Annual')

# all datasets to same date standard 

#month
gm['year'] = gm['yyyymm'] // 100
gm['month'] = gm['yyyymm'] % 100
#quarter
gq['year'] = gq['yyyyq'] // 10
gq['quarter'] = gq['yyyyq'] % 10
#year
ga['year'] =ga['yyyy']

# Line up dates to be end of period + datetime
gm['date']=pd.to_datetime(gm['year']*10000+gm['month']*100+1,format="%Y%m%d")
gm['date']=gm['date']+MonthEnd(0)
gm['quarter']= gm['date'].dt.quarter

gq['date']=pd.to_datetime(gq['year']*10000+gq['quarter']*300+1,format="%Y%m%d")
gq['date']=gq['date']+MonthEnd(0)

ga['date']=pd.to_datetime(ga['year']*10000+1200+1,format="%Y%m%d")
ga['date']=ga['date']+MonthEnd(0)

# collect all variables in monthly frequencies (i.e., add what is missing in gm)
varm = pd.merge(gm, gq[['cay','ik','quarter','year']], how='left',on=['year','quarter'])
varm = pd.merge(varm, ga[['eqis','year']], how='left',on=['year'])

# disaggregate dividends and earnings from D12 and E12:
# assume equal for the first 12 months (in 1871) 
varm['d'] = np.where(varm['date'].dt.year==1871, 
                     varm['D12']/12, np.nan)
varm = varm.sort_values(by=['date'])

for t in range(12, len(varm)):
    varm.loc[t, 'd'] = varm.loc[t, 'D12'] - varm.loc[t-1, 'D12'] + varm.loc[t-12, 'd'] 
    
# repeat for earnings:
varm['e'] = np.where(varm['date'].dt.year==1871,
                     varm['E12']/12, np.nan)

varm = varm.sort_values(by=['date'])
for t in range(12, len(varm)):
    varm.loc[t, 'e'] = varm.loc[t, 'E12'] - varm.loc[t-1, 'E12'] + varm.loc[t-12, 'e'] 

varm
#####################
# Raw data complete #
#####################
