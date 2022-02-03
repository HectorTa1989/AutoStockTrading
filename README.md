# AutoStockTrading
Automatic trading Python | Buy/sell stocks using AI and rules

## Introduction
I have been an investor in the stock and crypto market for a while. Due to the busy schedule, I often miss out on some timing to buy or sell stocks. Also, I wanted to limit the number of times I see my brokerage account to control my addiction. So I decided to create an advanced portfolio rebalancing tool.

## Use of stock portfolio rebalancing
I have built a decent software tool to trade autonomously according to my needs using rules and AI. It also provides me a morning breakfast of the information I wanted in a single email, which helps me reduce the time by searching 100 different websites. Also, it can trade autonomously on the stocks which I wanted. In the repository, I will explain the code and how it works.
This repository will show some highlights of different parts of code. So it is a technical article. However, the article will also help the beginner!

## Setup
Below are some of the packages and tools required to get started
Airflow — Workflow management tool
Airflow setup in local or cloud instance
Robinhood, Alpaca, or any brokerage account
Robin-stocks python package
Finviz or Quandl or any trade data package
Email server or AWS SES

It is quite a bit of setup. However, I have linked the articles and update them when I write more about them. If you require more information, please leave a comment.

Email server or AWS SES is an optional service; if you wanted to send an email daily, you need to configure it with airflow. Else, you can also store the information in a database according to your needs. In this article, I will be using email service using AWS SES.
All of my installations will be on Ubuntu pc, either on local or AWS cloud instance.
Workflow
Below is the detailed workflow of the items we will perform in this article. Yes, it includes a lot of parts of components! Do you think making money in the stock market is easy??

![image](https://user-images.githubusercontent.com/31132150/152369309-a6c5d70a-a980-4001-b45b-d2cc4c3417ac.png)

The workflow shown above is the current flow for the article. I will write about more features with “bells and whistles” frequently. Also, this workflow can be easily extendable with different features.

## The Core (Coding)
As it includes different coding parts, we will walk through it in different parts. Also, for simplicity, I will focus on workflow, getting information, trading, and emailing.

### Dags
Dags are an integral part of Airflow. Check this article to learn more about airflow. A DAG is a Directed Acyclic Graph that represents an individual workflow. DAGs indicate the workflow order of execution and connect different parts. Dags also have some parameters which can be changed.

![image](https://user-images.githubusercontent.com/31132150/152369541-60680fda-8a3c-4dfb-bae2-59012be9a02d.png)

### Operator
Operators represent what task is being executed at each step of a DAG. Such as python_operator, s3_file_transform_operator, etc. You can check the list of operators here
For this article, we will be using below operators
1. PythonOperator
2. EmailOperator
3. BashOperator — Optional
If we check the initial workflow, we need to first connect to Robinhood and get the details.

![image](https://user-images.githubusercontent.com/31132150/152369824-f00dcf3d-92ad-4973-b7e5-8846e8706f5c.png)

Connecting Robinhood or brokerage account:
My current brokerage account is Robinhood, so I have to connect to Robinhood and use it in this article. However, feel free to use any brokerage account which has API. Below are some options on the US market brokerage accounts
1. Robinhood
2. TD Ameritrade
3. Alpaca
4. Interactive brokers

If you are in international territory, please perform your search and find out the brokerage, with API enabled.

### Connecting to Robinhood
Connecting with Robinhood account is made simple using the robin-stocks package. API has support for normal login or using MFA.
1. Normal login
`
\\ normal login
login = r.login('username@test.com','password')
`
2. MFA login with pyotp
Here pyotp can generate OTP using the secret code for your app. To use this feature, you will have to sign in to your Robinhood account and turn on two-factor authentication. Robinhood will ask you which two-factor authorization app you want to use. Select “other” Robinhood will present you with an alphanumeric code. This code is what you will use for “My2factorAppHere” in the code below. Run the following code and put the resulting MFA code into the prompt on your Robinhood app.

`
import pyotp
import robin_stocks as r
totp  = pyotp.TOTP("My2factorAppHere").now()
login = r.login('username','passowrd', mfa_code=totp)
`

3. 2FA login on the server without pyotp
If you do not want to change or create other code from 2FA, you can log in manually using option 1 in the system you will use. For the first time, you can enter the OTP. The session will be saved temporarily. Until it the session ends, you can use with adding the OTP.

### Fetching portfolio stocks
As a next step, we need to get the holdings of the account. Here we are filtering the stocks list with my buy price. We can select only the lagging stocks in the portfolio or get all the stocks in the portfolio.
In fact, we can also fetch the watchlist and monitor the stocks on my watchlist category. We can also customize the stocks without even going to a brokerage account by having an external file.

![image](https://user-images.githubusercontent.com/31132150/152370472-dd07b6c8-2374-442e-8dc3-c5823cb888a1.png)

### Getting market data
As a next step, we need to fetch all the details about the stock. Here we have different options to get the details about the performance of stocks. Below are some of the packages
1. Robin-stocks via Robinhood
2. Finviz
3. Alpaca
4. Polygon.io
5. Quandl
To keep it simple, we will fetch the “Company name, Sector, Performance month,” but the options are limitless! You can select the metric according to your needs.

![image](https://user-images.githubusercontent.com/31132150/152370658-514ae990-afb6-471e-91f7-1e9943d1d36b.png)

### Analyst updates

Analyst updates are one of the important information which I used to check. Analysts have already analyzed a ton of information; why not use them? If many of them are mentioning it to sell, then there is something to look at it.
As there are many ratings at different times, we can get the analyst ratings for the last 30 days in the stock.

![image](https://user-images.githubusercontent.com/31132150/152370821-ad8dd81b-9df0-43d6-aed4-49cd7346a254.png)

![image](https://user-images.githubusercontent.com/31132150/152370954-389a1da5-f0b6-4cd5-95bc-2a8ae8f95bba.png)

### News Updates

One information that is always required is news updates about the stock. News can be positive or negative about the company. This metric completely affects the stock price. To fetch the news, we can get from different parts
1. Robin-stocks
2. Finviz
3. stocknewsapi
4. stocktwits
We also need to get the sentiment of the articles and community updates, which we will discuss in this or another article.
Combining all information
As a final step, we need to collect all the information from different places and create a final pandas data frame with different information. We will merge different data frames into one. Here is the code snipped to combine everything.

![image](https://user-images.githubusercontent.com/31132150/152371088-8efa9a2f-486e-4718-a944-f8bf0ba0f411.png)

![image](https://user-images.githubusercontent.com/31132150/152371206-5e1a2191-66e7-4911-82b5-49ef337d9975.png)

Dataframe has more columns, and it is hidden after the news 1 column.

![image](https://user-images.githubusercontent.com/31132150/152371337-407aeef2-322b-48e0-9ef4-1478efeb6c1c.png)

Analyst ratings are in different columns. We can combine it with the main data frame; however, I would like to have it separately.

### Saving all data

As a next step, we need to save all the data. We need to perform this for frequent email notification and to input it into our AI model. In my personal mode, I have used AWS Dynamo DB and Amazon Aurora. We will save the article for saving into a database later.
For the sake of this article, I have used it to save as an HTML file to append other required information in the later part of the email.

### Rebalancing Portfolio or Trading

Here comes the fun part of this article. Automatic rebalancing means it is not just rebalancing and can be used in different ways.
1. Stock trading
2. Cumulative buying of a stock
3. Average out buying
4. Create your own rules!!

I have written articles on how to use AI in trading here. The problem with the approach is we are monitoring the stock most of the time. And there are so many stocks which we need to track. As I wanted to nail down only to specific stocks in my portfolio, this approach works for me, and it may work for you too!
I will use all the sentiments of Twitter, AI model prediction, and all other calculations only if it passes my initial set of conditions or rules. If it does not pass, I am not going to waste time and resources on it.

### My Conditions or Rules

I usually buy stocks that I won’t sell. Also, the stock is exciting if it falls. Generally, it is easier said than done. When a stock falls 5%, we will wait for the stock to go down 6% to buy as per human tendency!
So if I create my conditions in code, I don’t need to worry about the monkey nature of mind.
Even God Couldn’t Beat Dollar-Cost Averaging
Is one of the golden statement which I follow. Yes, you might miss some profits if you average it out. However, you might not lose on the stocks unless you are investing in crap. Uff enough about philosophy, let’s get into the technical part!

#### Stage 1 — Filtering

Here are some rules which I have been using in selection criteria
Buy a small quantity of stock if it is 5% down. Ex: $500
Buy more if it is 10% down. Ex: $1000.
And the list goes on; you get the point!
Buy a stock if the total value is 30% less than my current price — Ex: 70% of current value.
Buy a stock if the total value is 50% less than my current price — Ex: 70% of current value.
However, the point is these are my initial set of conditions. These will act as a screening for my next stage.

#### Stage 2 — Balance and portfolio check

Even if the stock is down and excellent, it should not cross 20% of my overall portfolio. So I would filter the stocks if it is more on my portfolio. And, I should have money in my account either in the margin or as cash.

![image](https://user-images.githubusercontent.com/31132150/152371695-c4ef15bd-92a6-43a8-8ddb-1014d708c6c9.png)

You might see the word Crypto. Yes, we can save that for another article. Let me know in the comments if you need a crypto article.

#### Stage 3 — Sentiment
The sentiment of stocks is important while buying a stock too. This is an optional step to understanding the wisdom of crowds — two places to get the sentiment.
Twitter
Stocktwits
Both have an API to get the sentiment from the wisdom of crowds. I will discuss more integration of sentiment in another article as this one is getting so lengthier!

#### Stage 4 — AI model
This is again an optional setup for solid confirmation. I will throw a ton of information into an AI Model and see how it responds. If the model says it is the right time to buy, this gives more confidence. You can check out how to use an AI model in stocks using this article.

https://towardsdatascience.com/24x5-stock-trading-agent-to-predict-stock-prices-with-deep-learning-with-deployment-c15570720ae9

We can have it in an external service as a Rest API call for model prediction and get back the results.
Similar model can also be used for buying the stock!

### Finally, buy a stock
If a stock passes a series of stages, we will have more confidence in buying the stock. To buy stock in Robinhood, we can use the below code.

`r.order_buy_fractional_by_price(stock_ticker, 1000, extendedHours=True)`

Here, we are using fractional share by price. However, we can also place a limit order.

#### Adding to email notification

We should also notify you in an email about the list of stocks that can be bought. It can either be saved in the database or added to the HTML content which we used before.

### Final summary email

This will be sort of a summary email with all the information we have gathered and purchased. I have also added some formatting code to beautify the email. However, the email is completely generated automatically.

![image](https://user-images.githubusercontent.com/31132150/152372524-71fc3d75-1530-42da-9799-f843c33fb63d.png)

Yes, I do have some additional information on my personal level email with twitter trends, scores, crypto, podcasts info, scrapping additional websites and a lot more. However, those are my additional information. Let me know in comments if you wanted to any item on detail.
With that you also got the golden losing stocks which I am holding for free :)

### Final Thoughts

1. It will be a bit of work by making all the parts work.
2. You might face some integration challenges and deployment.
3. Once you make it work, it will help in the long term.
4. An email will act as breakfast info with all required info
5. It saves a ton of time and from monkey mind!
6. You can also use it in different brokerage API’s and replace it with premium data
7. Rebalance logic can also be applied for buying the stocks
8. Use different sentiment and AI model with technical data

### Disclaimer

This article is entirely informative. None of the content presented in this notebook constitutes a recommendation of any particular security. All trading strategies are used at your own risk.
I am planning to write additional articles based on the comments and feedback on the article. Follow me for more updates. Please check related articles

1. [24x5 AI Stock Trading agent to predict stock prices | Live trading] (https://medium.com/code-sprout/automatic-trading-python-portfolio-buy-sell-stocks-using-ai-and-rules-fa6182646f3d#:~:text=check%20related%20articles-,24x5%20AI%20Stock%20Trading%20agent%20to%20predict%20stock%20prices%20%7C%20Live%20trading,-2.%2024x7%20Live)

2. [24x7 Live crypto trading] (https://medium.com/make-money-with-code/24x7-live-crypto-trading-buy-doge-or-crypto-using-sentiment-151400dfaaa3?sk=60cd35de6529f24fdbafa3a35ed22082)
