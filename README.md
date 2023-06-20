# Initial imports
import os
import requests
import pandas as pd
from dotenv import load_dotenv
import alpaca_trade_api as tradeapi
from MCForecastTools1 import MCSimulation
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

# Load .env enviroment variables
path = os.path.join(os.getcwd(),".env")
path = os.path.normpath(path)
from dotenv import load_dotenv
load_dotenv(path)



## Part 1 - Personal Finance Planner
### Collect Crypto Prices Using the `requests` Library

# Set current amount of crypto assets
my_btc = 1.2
my_eth = 5.3
# Crypto API URLs
btc_url = "https://api.alternative.me/v2/ticker/Bitcoin/?convert=CAD"
eth_url = "https://api.alternative.me/v2/ticker/Ethereum/?convert=CAD"
# Fetch current BTC price
btc_response = requests.get(btc_url).json()
btc_price = btc_response['data']['1']['quotes']['CAD']['price']
# Fetch current ETH price
eth_response = requests.get(eth_url).json()
eth_price = eth_response['data']['1027']['quotes']['CAD']['price']
​
# Compute current value of my crpto
my_btc_value = my_btc * btc_price
my_eth_value = my_eth * eth_price
# Print current crypto wallet balance
print(f"The current value of your {my_btc} BTC is ${my_btc_value:0.2f}")
print(f"The current value of your {my_eth} ETH is ${my_eth_value:0.2f}")




### Collect Investments Data Using Alpaca: `SPY` (stocks) and `AGG` (bonds)
# Set current amount of shares
my_agg = 200
my_spy = 50
# Set Alpaca API key and secret
alpaca_api_key = os.getenv('ALPACA_API_KEY')
alpaca_secret_key = os.getenv('ALPACA_SECRET_KEY')
# Create the Alpaca API object
alpaca = tradeapi.REST(
    alpaca_api_key,
    alpaca_secret_key,
    api_version='v2')
inner
# Format current date as ISO format
today = pd.Timestamp('2023-06-15', tz='America/New_York').isoformat()
​
​
# Set the tickers
tickers = ["AGG", "SPY"]
​
# Set timeframe to "1Day" for Alpaca API
timeframe = "1Day"
​
# Get current closing prices for SPY and AGG
df_investments = alpaca.get_bars(
    tickers,
    timeframe,
    start = today
).df
# Reorganize the DataFrame
# Separate ticker data
df_agg = df_investments.loc[df_investments['symbol'] == 'AGG']
df_spy = df_investments.loc[df_investments['symbol'] == 'SPY']
​
# Concatenate the ticker DataFrames
df_concatenated = pd.concat([df_agg, df_spy],axis=1, keys=['AGG', 'SPY'],join='inner')
​
# Preview DataFrame
df_concatenated.head()



# Pick AGG and SPY close prices
agg_close_price = df_agg['close'].iloc[-1]
spy_close_price = df_spy['close'].iloc[-1]
​
# Print AGG and SPY close prices
print(f"Current AGG closing price: ${agg_close_price}")
print(f"Current SPY closing price: ${spy_close_price}")

# Compute the current value of shares
my_spy_value = my_spy * spy_close_price
my_agg_value = my_agg * agg_close_price
​
# Print current value of shares
print(f"The current value of your {my_spy} SPY shares is ${my_spy_value:0.2f}")
print(f"The current value of your {my_agg} AGG shares is ${my_agg_value:0.2f}")


### Savings Health Analysis
# Set monthly household income
monthly_income = 12000
​
# Consolidate financial assets data
crypto_value = my_btc_value + my_eth_value 
shares_value = my_spy_value + my_agg_value 
​
# Create savings DataFrame
data = {'amount': [crypto_value, shares_value]}
df_savings = pd.DataFrame(data, index=['crypto', 'shares'])
# Display savings DataFrame
display(df_savings)

# Plot savings pie chart
df_savings.plot(kind='pie', y='amount', title="Composition of Personal Savings")

# Set ideal emergency fund
emergency_fund = monthly_income * 3

# Calculate total amount of savings
total_savings = df_savings['amount'].sum()

# Validate saving health
if total_savings > emergency_fund:
    print("Congratulations! You have enough money in your emergency fund.")
elif total_savings == emergency_fund:
    print("Congratulations! You have reached your financial goal.")
else:
    savings_shortfall = emergency_fund - total_savings
    print(f"You are ${savings_shortfall} away from reaching your financial goal.")


## Part 2 - Retirement Planning

### Monte Carlo Simulation
# Set start and end dates of five years back from today.
# Sample results may vary from the solution based on the time frame chosen
start_date = pd.Timestamp('2018-06-15', tz='America/New_York').isoformat()
end_date = pd.Timestamp('2023-06-15', tz='America/New_York').isoformat()
timeframe = "1Day"
# Get 5 years' worth of historical data for SPY and AGG


df_stock_data = alpaca.get_bars(
    tickers,
    timeframe,
    start=start_date,
    end=end_date
).df

# Reorganize the DataFrame
# Separate ticker data


df_agg_5yrs = df_stock_data.loc[df_stock_data['symbol'] == 'AGG']
df_spy_5yrs = df_stock_data.loc[df_stock_data['symbol'] == 'SPY']

# Concatenate the ticker DataFrames
df_concatenated_5yrs = pd.concat([df_agg_5yrs, df_spy_5yrs],axis=1,keys=['AGG', 'SPY'],join='inner')


# Preview DataFrame
df_concatenated_5yrs.head()

# Configuring a Monte Carlo simulation to forecast 30 years cumulative returns

# Set the portfolio weights (60% SPY; 40% AGG)
weights = [0.6, 0.4]

# Create the MCSimulation object
MC_sim = MCSimulation(
    portfolio_data = df_concatenated_5yrs,  # This is your DataFrame with stock data
    weights = weights,  # Portfolio distribution
    num_simulation = 500,  # Number of simulations
    num_trading_days = 252*30  # Trading days over 30 years
)


# Printing the simulation input data
MC_sim.portfolio_data.head()


# Running a Monte Carlo simulation to forecast 30 years cumulative returns
MC_sim.calc_cumulative_return()


# Plot simulation outcomes
line_plot = MC_sim.plot_simulation()

# Display the plot
plt.show()


# Plot probability distribution and confidence intervals
plt.figure(figsize=(12, 6))
MC_sim.plot_distribution()


### Retirement Analysis

# Fetch summary statistics from the Monte Carlo simulation results
summary_stats = MC_sim.summarize_cumulative_return()

# Print summary statistics
print(summary_stats)



### Calculate the expected portfolio return at the `95%` lower and upper confidence intervals based on a `$20,000` initial investment.
# Set initial investment
initial_investment = 20000

# Calculate the percentile values for the lower and upper confidence intervals
ci_lower = round(summary_stats[8]*10000,2)
ci_upper = round(summary_stats[9]*10000,2)


# Print results
print(f"There is a 95% chance that an initial investment of ${initial_investment} in the portfolio"
      f" over the next 30 years will end within the range of"
      f" ${ci_lower:.2f} and ${ci_upper:.2f}")


### Calculate the expected portfolio return at the `95%` lower and upper confidence intervals based on a `50%` increase in the initial investment.


# Set initial investment
initial_investment = 20000 * 1.5

# Use the lower and upper `95%` confidence intervals to calculate the range of the possible outcomes of our $30,000
ci_lower_increase = ci_lower * 1.5
ci_upper_increase = ci_upper * 1.5
# Print results
print(f"There is a 95% chance that an initial investment of ${initial_investment} in the portfolio"
      f" over the next 30 years will end within in the range of"
      f" ${ci_lower} and ${ci_upper}")


## Optional Challenge - Early Retirement


### Five Years Retirement Option


# Configuring a Monte Carlo simulation to forecast 5 years cumulative returns

# Set the portfolio weights (60% SPY; 40% AGG)
weights = [0.6, 0.4]

# Create the MCSimulation object
MC_sim_5years = MCSimulation(
    portfolio_data = df_concatenated_5yrs,  # This is your DataFrame with stock data
    weights = weights,  # Portfolio distribution
    num_simulation = 1000,  # Number of simulations
    num_trading_days = 252*5  # Trading days over 30 years
)
MC_sim_5years.portfolio_data.head()


# Running a Monte Carlo simulation to forecast 5 years cumulative returns
MC_sim_5years.calc_cumulative_return()

# Plot simulation outcomes
# Plot simulation outcomes
line_plot = MC_sim_5years.plot_simulation()

# Display the plot
plt.show()

# Plot probability distribution and confidence intervals
plt.figure(figsize=(12, 6))
MC_sim_5years.plot_distribution()

# Fetch summary statistics from the Monte Carlo simulation results
summary_stats_5years = MC_sim_5years.summarize_cumulative_return()

# Print summary statistics
print(summary_stats_5years)

# Set initial investment
initial_investment = 20000

# Calculate the percentile values for the lower and upper confidence intervals
ci_lower_five = round(summary_stats_5years[8]*10000,2)
ci_upper_five = round(summary_stats_5years[9]*10000,2)



# Print results
print(f"There is a 95% chance that an initial investment of ${initial_investment} in the portfolio"
      f" over the next 5 years will end within in the range of"
      f" ${ci_lower_five} and ${ci_upper_five}")
