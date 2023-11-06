# FinBERT & LSTM-based Tech Stocks Trading Assistant Web App </br>

## Introduction </br>

</br> This project is a simple Streamlit **stock trading assistant** web-app, which includes **3 tools** that can help a trader have more accurate and profitable trades. </br>
You can try our app here: [App Link in HugginFace Spaces](https://huggingface.co/spaces/IoannisTr/Tech_Stocks_Trading_Assistant) </br>
An overview of this dashboard-like web app can be seen in the following figure: </br>
</br>
![Dashboard Picture](sample_2.PNG "Dashboard") </br>
</br>
The app-assistant consists of 3 parts-tools: </br>
1. Stock Headers' of www.marketwatch.com articles sentiment analysis (FinBERT model) 
2. 7-day stock price prediction (LSTM model)
3. Outlooks of four different technical analysis methods for stock trading</br>
</br>
When the user chooses a stock from the app's drop-down menu, each of these 3 "tools" will return their "advice"/outlook. </br>

## FinBERT-based Sentiment Analysis (1st tool)
The 1st tool is the Sentiment Analysis lineplot. When the user chooses a stock, our code scraps the last 17 articles' headlines of this stock from www.marketwatch.com and labels them as either "Positive", "Negative", or "Neutral", which are encoded as scores (+100, 0, -100 correspondingly). As there is no standard way to quantify the sentiment of something, we will use a scale of (-100, +100). It must be noted that even though we get the last 17 articles' headers in each execution, only the last 12 headers' sentiment can be seen in the plot. That is because we use a rolling average with a sliding window of size 5, in order to "smooth" the line. </br>
</br>
The model used can be found here: [HuggingFace ProsusAI finbert](https://huggingface.co/ProsusAI/finbert) </br>
This model is a pre-trained NLP model that can analyze sentiment of financial text and it is built by further training the BERT language model in financial text. </br>
</br>
Additionaly, the model was fine-tuned by using the following dataset: [Financial PhraseBank (2014)](https://huggingface.co/datasets/financial_phrasebank).</br> 
This dataset includes 4840 headlines-like sentences about financial news, labeled as either **Positive** or **Negative** or **Neutral**. </br>
</br>
After training the model, we save the weights in **.bin** file and reuse them each time we want to classify a headline. After the 17 "scraped" headlines are classified, they "get" a score of either *+100, 0, or -100*, and the rolling average of them is "printed" in the lineplot.</br>
When this line is mostly "negative" (below 0), we expect that most of the last articles have a negative sentiment, when they are mostly positive (above 0), we expect that there are many articles with positive sentiment, while when the line is close to 0, or revolves around it with no specific trend, we expect mixed outlook articles.</br>
</br>

`Interpretation for traders` </br>
</br>
Many recent negative news may lead to stock price decline, which means that the stock now is cheaper than it was recently, and the explanations are the following:</br>

* Either the bad news are minor and temporary, so the stock will soon recover (Often a :+1: **Buy Signal**) </br>
* Or the bad news are major and non-temporary, so the stock price may decline even more (Often a :-1: **Sell Signal**) </br>
</br>

Many recent positive news may lead to stock price increase which means that the stock now is more expensive than it was recently, and the explanations are the following: </br>

* Either the good news are minor and temporary, so the stock will soon drop back to it's usuall price soon (Often a :-1: **Sell Signal**) </br>
* Or the good news are major and non-temporary, so the stock price may increase even more in the next days or hours (Often a :+1: **Buy Signal**) </br>
</br>

## LSTM model for 7 days stock price prediction (2nd tool)
The second tool is an LSTM model that uses historical data of adjusted stock prices of a chosen stock (data for last 2 years), and predicts the stock price for the next 7 days. The output of these predictions can be seen in a time-series plot (lower-right side of dashboard). </br>
</br>
The data are acquired by **yfinance** library [yfinance Documentation](https://pypi.org/project/yfinance/) that gets data from Yahoo's API. </br>
It must be noted that **yfinance** returns us the following fields: </br>
1. **Date**: The date in which the record corresponds to
2. **Open**: The stock's opening price of the day
3. **High**: The stock's highest price of the day
4. **Low**: The stock's lowest price of the day
5. **Close**: The stock's closing price of the day
6. **Adj Close**: The stock's closing price of the day, **adjusted to stock's splits, dividends, and other corporate actions** </br>

</br>
In our analysis, we will use the "Adj Close" price only. </br>

It is notable that no model weights need to be saved, and in each stock selection from the user, the data acquistion and model training happen **on the go**. </br>

At last, some model arguments (like batch_size, etc.) are manually set for each stock specifically, in order to ensure as high **R-squared** scores as possible.</br>
</br>
`Interpretation for traders` </br>

* Upwards projection could be a potential :+1: **Buy Signal** (especially for large R-squared scores, which can be seen in the plot) </br>
* Downwards projectios could be a potential :-1: **Sell Signal** (especially for large R-squared scores, which can be seen in the plot) </br>
</br>

## Technical Analysis Methods (3rd tool) 

This set of methods are out of the scope of the course but they were added because we believe that they can be used supplementary with the previous 2 tools. </br>
Technical Analysis Methods are **Statistical Methods** for stock trading which usually use price rolling averages and standard deviations, in order to return their outlook (e.g. Buy, or Hold, or Sell). These methods usually belong to one of the following trading timeframes:
* **Short term trading** (Buying and Selling Stocks within hours,minutes or even seconds)
* **Medium term trading** (Buying and Selling Stocks within days or weeks)
* **Long term trading** (Buying and Selling Stocks within months)
</br>
To use the following methods, we have to get the realtime stock price when the NYSE and NASDAQ are open. To do so we "scrap" the current price from www.marketwatch.com </br>

### 1st method: Bollinger Bands of 20 days and 2 standard deviations
</br>

This method relies on 3 lines: 
* the 1st one is the 20 day rolling average of the stock price
* the 2nd one is the 20 day rolling average + 2 standard deviations (Upper Limit)
* the 3rd one is the 20 day rolling average - 2 standard deviation (Lower Limit) </br>

and is a "Medium-term" trading method (days or weeks)

`Interpretation for traders`

* When a stocks current price is between the rolling avg. and the Upper Limit, it is considered as **Overbought** (which is a 👎 **Sell Signal**)
* When a stocks current price is between the rolling avg and the Lower Limit it is considered as **Oversold** (which is a 👍 **Buy signal**)
* When a stocks current price is equal to the rolling average, it is considered as neither **Overbought** or **Oversold**, (which is a :fist: **Hold Signal**)
* When a stocks current price is higher than the upper limit or lower than the lower limit, it is considered as **Unusuall Event** (which signals :exclamation: **Caution**)</br>
 
*In the following figure, we can see that Apple's stock is considered as a "Buy" currently, according to the Bollinger Bands of 20 days and 2 st. deviations:*
![Bollinger Bands example 1](BB_example_1.PNG "Bollinger Bands example 1") </br>
</br>

### 2nd method: Bollinger Bands of 10 days and 1.5 standard deviations 
</br>

This method relies on 3 lines: 
* the 1st one is the 10 day rolling average of the stock price
* the 2nd one is the 10 day rolling average +1.5 standard deviations (Upper Limit)
* the 3rd one is the 10 day rolling average -1.5 standard deviation (Lower Limit) </br>

and is a "Short-term" trading method (hours, minutes or even seconds)

`Interpretation for traders`

* When a stocks current price is between the rolling avg. and the Upper Limit, it is considered as **Overbought** (which is a 👎 **Sell Signal**)
* When a stocks current price is between the rolling avg and the Lower Limit it is considered as **Oversold** (which is a 👍 **Buy signal**)
* When a stocks current price is equal to the rolling average, it is considered as neither **Overbought** or **Oversold**, (which is a :fist: **Hold Signal**)
* When a stocks current price is higher than the upper limit or lower than the lower limit, it is considered as **Unusuall Event** (which signals :exclamation: **Caution**)</br>
 
*In the following figure, we can see that Lyft's stock is considered as a "Buy" currently, according to the Bollinger Bands of 10 days and 1.5 st. deviations:*
![Bollinger Bands example 2](BB_example_2.PNG "Bollinger Bands example 2") </br>
</br>


### 3rd method: Bollinger Bands of 50 days and 3 standard deviations 
</br>

This method relies on 3 lines: 
* the 1st one is the 50 day rolling average of the stock price
* the 2nd one is the 50 day rolling average +3 standard deviations (Upper Limit)
* the 3rd one is the 50 day rolling average -3 standard deviation (Lower Limit) </br>

and is a "Long-term" trading method (weeks)

`Interpretation for traders`

* When a stocks current price is between the rolling avg. and the Upper Limit, it is considered as **Overbought** (which is a 👎 **Sell Signal**)
* When a stocks current price is between the rolling avg and the Lower Limit it is considered as **Oversold** (which is a 👍 **Buy signal**)
* When a stocks current price is equal to the rolling average, it is considered as neither **Overbought** or **Oversold**, (which is a :fist: **Hold Signal**)
* When a stocks current price is higher than the upper limit or lower than the lower limit, it is considered as **Unusuall Event** (which signals :exclamation: **Caution**)</br>
 
*In the following figure, we can see that MongoDB's stock is considered as a "Sell" currently, according to the Bollinger Bands of 50 days and 3 st. deviations:*
![Bollinger Bands example 3](BB_example_3.PNG "Bollinger Bands example 3") </br>
</br>

### 4th method: Moving Average Convergence Divergence (MACD)
</br>
MACD is a trend following momentum indicator based on 2 lines: </br>

* The 1st one is the difference between the 26-day exponential moving average (EMA) and the 12-day EMA (usually called **White Line**) </br>
* The 2nd one is the 9-day EMA (usually called **Red Line**) </br>


`Interpretation for traders`

* When **White Line** > **Red Line** the stock is considered to have a **Downtrend and No Signal** outlook (which is a 👎 **Sell Signal**)

* When **White Line** < **Red Line** the stock is considered to have a **Uptrend and No Signal** (which is a 👍 **Buy signal**)

* When **White Line** > **Red Line** and the former crossed the latter from upwards today or yesterday, the stock's outlook is **Downtrend and Sell** (which is a 👎 **Sell Signal**)

* When **White Line** < **Red Line** and the former crossed the latter from downwards today or yesterday, the stock's outlook is **Uptrend and Buy** (which is a 👍 **Buy signal**)

*In the following figure, we can see that Paypal's stock has a "Downtrend and No Signal" outlook currently (since the white line < red line), according to the MACD:*
![MACD](MACD_example.PNG "MACD example") </br>
</br> 

## Final Thoughts
None of the above methods is used alone for stocks trading, so we expect that using them all together when deciding the next trade is the best solution. After manually testing this simple, yet intuitive web app, we concluded that it can be valuable mostly when all 3 tools "agree" on their outlooks.

## How to Run the Project </br>
To begin with , the main files are : all-data.csv, FinBERT_training.py , main.py , functions.py and the stocks.py and the fine_tuned_FinBERT folder.
Moreover, you can check the two .ipynb files which contain the EDA and the evaluation of our models.

In more details : 
1) The __fine_tuned_FinBERT folder__ contains :
All the weights that have been saved during the training of the model. The user by downloading the folder does not need to retrain the model. This folder can be downloaded from [here](https://drive.google.com/file/d/1-UnJawor6s6roOU4wD_Ppdxdm4EJqRmV/view?usp=sharing) as the GitHub does not allow us to upload large files.
2) The __stocks.py__ contains : 
 The necessary functions in order to acquire our data for the LSTM and the FinBERT model.
3) The __functions.py__ contains:
 The function that creates the Bollinger plots and our two models .
4) The __FinBERT_training.py__ contains : 
 All functions needed to train the FinBERT model. 
5) The __all-data.csv__ contains : 
 The data that has been used for the training of the FinBERT model (Financial Prasebank dataset) .
6) The __main.py__ contains : 
The calling of the functions from the two above mentioned files (functions.py ,stocks.py) and the creation of the web app application (streamlit).In order to enable the application the user has to perform the below procedure:
* Execute the main.py in an editor
* Type : streamlit run main.py
* A browser window will pop up with our interactive Dashboard.
* Or, double click the "Tech Stocks Trading Assistant" Windows Batch File.

**NOTE: All the above-mentioned files along with the weights folder need to be in the same folder in order to run the project.**

## Team Members

Ilias Dimos (f2822102) [GitHub Profile](https://github.com/ildim9) </br>
Charilaos Petrakogiannis (f2822112) [GitHub Profile](https://github.com/CharilaosPetrakogiannis) </br>
Ioannis Triantafyllakis (f2822115) [GitHub Profile](https://github.com/Ioannis-Triantafyllakis) </br>
Nikolaos Tsekas (f2822116) [GitHub Profile](https://github.com/nicktsekas) </br>
