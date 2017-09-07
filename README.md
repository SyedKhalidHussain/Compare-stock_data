# Compare-stock_data
# Compare the average opening price of two compay stock (Data access from Yahoo finance data)

import re
import requests
import json
import pandas as pd
from datetime import date
import matplotlib.pyplot as plt
import time

def retrieve_quotes_historical(stock_code):      # defining the function "retrieve_quotes_historical" with arguent "stock_code"
    quotes = list()
    url = 'https://finance.yahoo.com/quote/%s/history?p=%s' % (stock_code, stock_code)   # modifying the url with address/parameter "stock_code"
    fhandle = requests.get(url)            # Getting the file handle of url
    m = re.findall('"HistoricalPriceStore":{"prices":(.*?),"isPending"',fhandle.text)     #Finding the content having search pattern from string "fhandle.text" using re package
    if m:              # True if there m has some value, False if m is empty
        quotes = json.loads(m[0])             # Load the json data from string into collection of list/dict
        quotes = quotes[::-1]                 # reversing the order of quotes (a list of list/dict)
    return [item for item in quotes if not 'type' in item]       # returning the item if there is no string "type" in it

def avg_open_price(stock_code):              # Defining the function "avg_open_price" to calculate the average of monthly open price of stock with paramter/argument (stock_code)
    quotes = retrieve_quotes_historical(stock_code)         # Calling the function "retrieve_quotes_historical"
    #print(quotes)
    list1 = list()                    # empty-list lis1
    for i in range(len(quotes)):      # Iterate from 0~length of quotes to make the dates as index
        x = date.fromtimestamp(quotes[i]['date'])        # Extracting the date from quotes in time format
        y = date.strftime(x,'%Y-%m-%d')                 # Transfer the time format into string format using date package
        list1.append(y)                 # append the string format date in list1
    quotes_df_original = pd.DataFrame(quotes,index = list1)     # make the DataFrame (quotedataframe_original) using quotes with index "list1" containing dates
    listtemp = list()           # empty-list "listtemp"
    for i in range(len(quotes_df_original)):      # iterate through the length of "quotes_df_original"
        temp = time.strptime(quotes_df_original.index[i],"%Y-%m-%d")    # format string into time of index of quotes_df_original and store in variable temp
        listtemp.append(temp.tm_mon)                # append the month of temp (containing date in time format) in listtemp
    tempdf = quotes_df_original.copy()              # make the copy of quotes_df_original and store in tempdf
    tempdf["month"] = listtemp                      # make the new column with name "month" in temporary dataframe and store listtemp in it
    avg_open_price = tempdf.groupby('month').open.mean()      # Group all the data with respect to month and take the average of column open i.e. opening price
    return avg_open_price                         # return the average opening price of each month with indexes are month

open1 = avg_open_price('INTC')                     # calling the function "avg_open_price" of "Intel Corporation"
open2 = avg_open_price('IBM')                     # calling the function "avg_open_price" of "IBM"
plt.subplot(211)
plt.title('Intel Corporation')
plt.plot(open1.index, open1.values, color = 'r', marker = 'o')        # plot the open1 with index and values as 1st and 2nd paramter
plt.subplot(212)
plt.title('IBM')
plt.plot(open2.index, open2.values, color = 'g', marker = 'o')
plt.show()

