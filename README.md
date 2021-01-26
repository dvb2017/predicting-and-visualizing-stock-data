# Predicting and Visualizing Stock Data 

## Introduction

**Business Case**  The business care here is a bit different than previous projects.  Rather than focusing solely on an in depth analysis of the data in order to answer a specific question, my goal is to create a data tool and incorporate the analysis into that. Specifically, I would like to create an interface that provides a wealth of information about stock market prices.  It should be simple to use and provide a high level of interactivity.  As this is a data science project, there will also be an emphasis on machine learning and how it could be used to predict future prices.  Additionally, there were two specific topics I wanted to examine and visualize, these were yield rates and economic sectors.



## Table of Contents

- `presentation.pdf` - a pdf file containing the powerpoint slides
- `student.ipynb` - Jupyter Notebook containing notated code
- `README.md` - this file, serving as a directory
- `charts` - a folder containing all of the charts generated
- `data` - folder containing the following data
- `data/yield2020.tsv` - .tsv file containing US Treasury yield data (2020)
- `data/yield2019.tsv` - .tsv file containing US Treasury yield data (2019)
- `data/midcap_data.csv` - main dataset exported from notebook
- `data/nasdaq_list.csv` - dataset containing the NASDAQ stock symbols
- `data/nyse_list.csv` - dataset containing the NYSE stock symbols 

## The Datasets

There were a few datasets used here, so I will briefly describe them and where they came from. To start with there were two datasets from the main US stock exchanges, `nasdaq_list.csv` and `nyse_list.csv`.  These were categorical, containing stock names and symbols along with some other information about the stocks such as their sector and market capitalization.  These were then combined to form a list of every stock that I wanted to include in the analysis.  The primary dataset used throughout the analysis was called `midcap_data.csv`, however this was exported from my Jupyter Notebook. It is a time series containing one year of historical stock prices, most importantly including the high, low, open and closing prices for each day.  It was gathered by scraping Yahoo Finance historical data off of the web using the list of stock symbols from the previous datasets as a reference.  The two yield datasets contain daily yield values published by the US Treasury.

## Data Cleaning / Feature Engineering

There was a fairly intensive amount of data cleaning to be done here.  As mentioned, the first step was combining the two stock exchange datasets so that I could use the resulting list to scrape the web.  Once the price data had been pulled from the web I added some columns from the stock exchange list such as sector and market cap to the resulting dataframe as I wanted to include sectors in my analysis.  Market cap was used to thin out the number of stocks so that computation time could be cut down significantly.  I did this by filtering out any stock smaller than what is generally considered "mid-cap" which means any that had a market cap of less than two billion dollars.  I then indexed the data using multi-level indexing, so that each index was composed of a stock symbol and a date.  I also eliminated any stocks that began trading within the past year.  Since I was already working with a somewhat small timeframe, examining anything shorter would have only complicated the analysis for no real benefit. I then moved on to some simple feature engineering for this dataset. I added a column for a one week moving average in order to check for short term trends more easily, as well as one to show the percentage change over the previous day.  Finally, I added one last column to show the difference between the high and low prices for the day so that I would have an interesting measure of volatility.  I then created a dataframe that was similar to this one, but instead of individual stocks it contained average values for each sector.  I also added a column to represent a sector composite value.  

Moving on the the final datasets which contained yield data, I started by simply combining the two separate year files into a single dataframe.  I then removed any dates that fell outside the timeframe I was examining.  Next, I added two columns representing different 'spreads' which are values used in the field of finance.  This is just the difference between yield values for two different times to maturity, and it is generally used as an easy benchmark to measure the steepness of the yield curve.  In this case I used the two year versus ten year spread as well as the five year versus thirty year spread since those are among the most common measurements.  For my last data cleaning task, I created a custom frequency to use for the date values since the stock market is not open every day, and I wanted to account for holidays as well. 

## Modeling

I tried two different modeling approaches for this data, as I was interested in whether one might perform better than the other.  The first model was an autoregressive integrated moving average (ARIMA).  For this model, I decided that rather than tuning ARIMA parameters for each individual stock, I would optimize parameters based on each sector index I had created and then apply those parameters to all stocks within that sector.  I had two main reasons for this choice.  First, it saved a great deal of computation time, and since stocks tend to follow the general trends of their sectors I was likely not losing much accuracy in exchange.  Second, single stocks tend to be subject to much more unpredictable 'noise' in the data, while a sector average will smooth out that type of issue.  I also opted to try a new method for parameter selection.  Rather than run a grid search I used the pmdarima module, which contains a helpful function called auto_arima.  This function runs some initial tests to determine the order of differencing (the 'd' parameter), and then uses stepwise selection to determine the remaining parameters.  It also allowed me to select which criterion I would like it to use to select the best model.  In this case I went with the Akaike information criterion (AIC) as it protects against both overfitting and underfitting.  Once I had the optimal parameters for each sector, I fit each stock to its corresponding model.  

For the second modeling technique I went with a neural network, specifically long short-term memory (LSTM).  LSTM is useful here because unlike many other neural networks it has feedback connections, so it is able to remember values.  This means it is ideal when making predictions based on time series data.   

## Visualization Techniques

Since the main purpose behind this particular project was the interface itself, the visualizations were very important.  In order to make the graphs feel more interactive I opted to use the Plotly graphing library.  This allowed me to create graphs with engaging features like data values that pop up when you hover over them with the mouse, as well as date range sliders.  Another interesting aspect of Python that I was able to try out was widgets.  These allowed me to put selection menus above the graph, allowing them to be updated simply by clicking a drop-down box and making a new selection.  One drop-down menu even allows the user to switch between the different graphs.  It also allows for selections to be dependent on others, for example if the user chooses 'Technology' as the sector, it will return only technology stocks to select from in the next box.

## Results - The Interface

For a complete look at the final interface, it can be found at the bottom of the Jupyter Notebook.  Some images will be included here along with brief descriptions of the graphs and their purposes.  The first graph also includes the three drop-down boxes that compose the simple interface.

1. Stock vs. Sector - On this graph you choose a sector, and the stocks associated with that sector can be chosen as well.  The stock value and sector index value are plotted, along with their 5-day predictions and confidence intervals. The sector value is scaled so that it has the same starting value as the stock.  There is a slider on the bottom to adjust the date range you would like to view.  Can be used to easily gauge whether a stock is outperforming its sector, as well as to see a 5-day prediction. 
![stockvsector](https://github.com/dvb2017/predicting-and-visualizing-stock-data/blob/main/charts/stockvsector.PNG)
2. Stock vs. Sector w/ Table - Same as above, but also returns a table showing the predicted percent change in value for both the sector index and stock. 
![stockvsectortable](https://github.com/dvb2017/predicting-and-visualizing-stock-data/blob/main/charts/stockvsector_table.PNG)
3. LSTM Forecasts - Plots the actual test value as well as the predicted value of selected stocks using LSTM.  It also returns the root mean squared error.  Can be compared to the One Step Ahead to see which model is more accurate with its predictions.
![lstm-graph](https://github.com/dvb2017/predicting-and-visualizing-stock-data/blob/maain/charts/LSTM-graph.PNG)
4. Compare Sectors - This graph shows the performance of each sector over the past year, with each index starting at 100.  Hovering over it makes all of the names appear from highest to lowest at that particular date. Easy to see which sectors are performing well and which ones are not.  
![sectors-graph](https://github.com/dvb2017/predicting-and-visualizing-stock-data/blob/main/charts/sectors_graph.png)
5. One Step Ahead Forecast - Plots the selected stock, along with its one step ahead prediction and confidence interval. Can be compared to the LSTM to give an idea of which model makes the better predictions.  
![one-step](https://github.com/dvb2017/predicting-and-visualizing-stock-data/blob/main/charts/one-step.PNG)
6. Yield vs. Stock Value - Plots the stock of your choice along with your choice of yield curve.  Makes it easy to see any potential relationships between stocks and yield spreads.  Also has a date slider at the bottom.  
![yield-stock](https://github.com/dvb2017/predicting-and-visualizing-stock-data/blob/main/charts/yieldvstock.PNG)
7. Yield vs. Sector Value - Plots the sector index of your choice along with your choice of yield curve.  Makes it easy to see any potential relationships between sector averages and yield spreads.
![yield-sector](https://github.com/dvb2017/predicting-and-visualizing-stock-data/blob/main/charts/yieldvsector.PNG)

## Future Work

In terms of future work, there is still a lot that can be done here.  The interface has a good amount of functionality right now, but there are still plenty of additional plots and options that could be added.  A simple one that I am already planning to add later would be a chart comparing the average root mean squared error across all stocks between the LSTM and ARIMA models, that way there is an even simpler way to compare them.  Another addition I would like to make is to have the full list of stocks as well as sector indices available to graph on the LSTM model (unfortunately this one took a very long time to run).  I also wouldn't mind double checking the auto_arima parameters with a grid search just to make sure they are optimized.  Another idea would be to run auto_arima for each individual stock, and then compare the results against when they were modeled by sector to see which option makes better predictions.  I think that the LSTM model could also be tuned better, and I also want to try running it using a yield spread as a feature.  

## Citation

1. https://finance.yahoo.com/
2. https://www.treasury.gov/resource-center/data-chart-center/interest-rates/Pages/TextView.aspx?data=yield
3. https://old.nasdaq.com/screening/companies-by-name.aspx?letter=0&exchange=nyse&render=download
4. https://old.nasdaq.com/screening/companies-by-name.aspx?letter=0&exchange=nasdaq&render=download
