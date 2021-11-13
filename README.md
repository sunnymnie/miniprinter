# Miniprinter
A simple, lightweight crypto trader. 

Limitations:
- Each asset can only have one strategy operating on it. ie, two strategies cannot both share BTCUSDT pair. 

## Call-graph (theoretical)
1. Init with list of strategies
2. For each strategy:
    1. strategy.consider_trade(): updates strategy to have the latest data, strategy consider if now is the time for a trade.
        1. If NOT: returns trade object encoding NO-trade
        2. If YES: returns a trade object with all necessary trade information. The strategy object is responsible for determining actual leverage multiples (using Kelly's formula). 
    2. Model analyzes trade object:
        1. If trade object is NO-trade: 
            1. If there's currently an open limit order, cancels it, transfers USDT back to main account via the Trader class, mark trade as cancalled
            2. If no open orders, pass
        2. If trade object is ACTUAL-trade:
            1. If no open orders and there are sufficient funds, use Trader object to initalize trade. Document trade
            2. If there are open orders, ignore
    3. 
3. Model does clean-up crew: 
    - If position is active, secure with OCO and mark as secured
    - If position is liquidated, moves USDT back to main account, documents trade as finished
    - IF secured trade does not have an order, mark as cancelled and transfer USDT back to main account via Trader class
    

## Classes and functions
### Strategy
- Hosts the price-series data for it's coins, is responsible for updating them when consider_trade() is called
- After considering trade, returns a trade object
- Automatically uses the kelly formula to determine optimal leverage, needs as paramaters the win/loss ratio, strategy determines 
    - limit margin buy/short price
    - tp and sl price, tp is always tp, and is less than sl for shorting
    - amount of asset to buy/short

### Trader
- takes in a trade object and activates the trade object as a limit order
- sends the limit order to binance
- optionally given a trade object to set an oco order

### Model
- is the main book-keeper, keeps tabs on what positions need to be secured by an oco, keeps a list of  
- keeps a dataframe of trades and current status:
|id|status|tp|stop|sl|
|---|---|---|---|---|
|1000|FINISHED|10|5|4|
|2000|CANCELLED|4|9|10|
|3000|SECURED|5|6|7|
|4000|ACTIVE|3|2|1|

where:
- active is when limit long/short order is placed
- secured is when order is activated and OCO order is placed
- cancelled is a progression from active if limit order was too late and order is cancelled. Else done manually
- finished is progression from secured, when OCO order is hit. 

### Printer (though not a class)
- keeps track of static variables:
    - Min trade amount. No trade is executed if trade amount (+borrowed amounts) is less than min trade amount. 
