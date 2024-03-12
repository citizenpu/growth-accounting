# -*- coding: utf-8 -*-
"""
Created on Mon Mar 11 11:18:56 2024

@author: citiz
"""

import numpy as np
import pandas as pd
from statsmodels.tsa.filters.hp_filter import hpfilter
# inefficient codes have been commented
pop1 = pd.read_excel("Downloads/潜在产出.xlsx", sheet_name="annuGDP",index_col=0) #set index to the the first column
pop1.index=pop1.index.year
pop1.index.name='YearCode'
# pop_trunc1=pop1['annuGDP'][3:-3]
# pop_trunc2=pop1['annuGDP'][3:-2]
# pop_trunc3=pop1['annuGDP'][3:-1]
# pop_trunc4=pop1['annuGDP'][3:]
dic={}
for i in range(1,5):
	if i<4:
		dic[f"pop_truc{i}"]=pop1['annuGDP'][3:-4+i]
	else:
		dic[f"pop_truc{i}"]=pop1['annuGDP'][3:]
	
# cycle1,trend1=hpfilter(pop_trunc1,6.25)
# cycle2,trend2=hpfilter(pop_trunc2,6.25)
# cycle3,trend3=hpfilter(pop_trunc3,6.25)
# cycle4,trend4=hpfilter(pop_trunc4,6.25)
dichp={}
for i in range(1,5):
	dichp[f'cycle{i}'],dichp[f'trend{i}']=hpfilter(dic[f"pop_truc{i}"],6.25)
	j=2019+i
	pop1[f'cycle{j}'],pop1[f'trend{j}']=np.nan,np.nan
	if i<4:
		pop1[f'cycle{j}'][3:-4+i],pop1[f'trend{j}'][3:-4+i]=dichp[f'cycle{i}'],dichp[f'trend{i}']
	else:
		pop1[f'cycle{j}'][3:],pop1[f'trend{j}'][3:]=dichp[f'cycle{i}'],dichp[f'trend{i}']

# pop1['cycle']=pop1['annuGDP']
# pop1['cycle'][3:-3]=cycle1
# pop1['cycle'][:3]=np.nan
# pop1['cycle'][-3:]=np.nan
# pop1['trend']=pop1['annuGDP']
# pop1['trend'][3:-3]=trend1
# pop1['trend'][:3]=np.nan
# pop1['trend'][-3:]=np.nan
with pd.ExcelWriter("Downloads/潜在产出.xlsx",mode='a',engine='openpyxl',if_sheet_exists="replace",) as writer:
	pop1.to_excel(writer,sheet_name=" annuGDPhp_updateagain")
	
#raw data come from the penn world table
penn = pd.read_excel("Downloads/FebPwtExport3112024.xlsx", sheet_name="Data") 
penn=pd.pivot(penn,index='YearCode',columns='VariableCode',values='AggValue')
penn['human capital']=penn['emp']*penn['avh']*penn['hc']
##penn=penn.loc(penn['human capital']!=np.nan) (drop observations with missing values)
penn=penn.dropna()
penn_trunc=penn.loc[penn.index>=1970]
penn_trunc['cyclehc'],penn_trunc['trendhc']=hpfilter(penn_trunc['human capital'],6.25)
penn_trunc['cyclec'],penn_trunc['trendc']=hpfilter(penn_trunc['rnna'],6.25)
penn_trunc['cycletfp'],penn_trunc['trendtfp']=hpfilter(penn_trunc['rtfpna'],6.25)
penn_trunc['cyclegdp'],penn_trunc['trendgdp']=hpfilter(penn_trunc['rgdpna'],6.25)
#now we can do growth accounting based on the hp filtered labor input, capaital and tfp
#to merge pop1 series from NBS to the penn series, on has to be followed by a variable NAME
penn_trunc=pd.merge(penn_trunc,pop1['trend2023'],on=penn_trunc.index.name)
#we cannot get the growth rate this way
#penn_trunc['dtrendgdp_nbs']=penn_trunc['trend2023'][1:]/penn_trunc['trend2023'][:-1]
penn_trunc['dtrendgdp_nbs']=penn_trunc['trend2023'].pct_change()*100
penn_trunc['dtrendhc']=penn_trunc['trendhc'].pct_change()*100
penn_trunc['dtrendc']=penn_trunc['trendc'].pct_change()*100
#first we need to calculate the tfp growth as rgdpna in the penn world table is not in line with the official one, we assume a fixed capital
#ratio of 0.4
penn_trunc['dtrendtfp_nbs']=penn_trunc['dtrendgdp_nbs']-0.4*penn_trunc['dtrendc']-0.6*penn_trunc['dtrendhc']
with pd.ExcelWriter("Downloads/FebPwtExport3112024.xlsx",mode='a',engine='openpyxl',if_sheet_exists="replace",) as writer:
	penn_trunc.to_excel(writer,sheet_name="pennhp_updateagain")
