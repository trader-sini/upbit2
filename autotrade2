import time
import pyupbit
import datetime
import telepot

access = "wy1S4y8qhcWOEE0JMpCxZeBYd3XRGfNMQBcKc2sO"
secret = "SwazVKN6i0Nr4Sw2yuxDxcwHZBifNjjwiQxgphBj"

def get_target_price(ticker, k):
    """변동성 돌파 전략으로 매수 목표가 조회"""
    df = pyupbit.get_ohlcv(ticker, interval="day", count=2)
    target_price = df.iloc[0]['close'] + (df.iloc[0]['high'] - df.iloc[0]['low']) * k
    return target_price

def get_start_time(ticker):
    """시작 시간 조회"""
    df = pyupbit.get_ohlcv(ticker, interval="day", count=1)
    start_time = df.index[0]
    return start_time

def get_ma15(ticker):
    """15일 이동 평균선 조회"""
    df = pyupbit.get_ohlcv(ticker, interval="day", count=15)
    ma15 = df['close'].rolling(15).mean().iloc[-1]
    return ma15

def get_balance(ticker):
    """잔고 조회"""
    balances = upbit.get_balances()
    for b in balances:
        if b['currency'] == ticker:
            if b['balance'] is not None:
                return float(b['balance'])
            else:
                return 0
    return 0

def get_current_price(ticker):
    """현재가 조회"""
    return pyupbit.get_orderbook(tickers=ticker)[0]["orderbook_units"][0]["ask_price"]

def get_ror(k, ticker):
    df = pyupbit.get_ohlcv(ticker)
    df['range'] = (df['high'] - df['low']) * k
    df['target'] = df['open'] + df['range'].shift(1)

    df['ror'] = np.where(df['high'] > df['target'], df['close'] / df['target'], 1)

    ror = df['ror'].cumprod()[-2]
    return ror


# 로그인
upbit = pyupbit.Upbit(access, secret)
print("autotrade start")

#원화로 거래 되는 ticker 가져오기
tickers = pyupbit.get_tickers("KRW")
tickers.sort()


# 텔레그램으로 메세지 보내기
token="1269160830:AAFshIib8P2nICaScQwru0r8Pn1LCVGmtCs"
mc = "497235997"
bot = telepot.Bot(token)



# 자동매매 시작
while True:
    try:
        now = datetime.datetime.now()
        start_time = get_start_time("KRW-BTC")
        end_time = start_time + datetime.timedelta(days=1)


        if start_time < now < end_time - datetime.timedelta(seconds=300):
            for ticker in tickers :
                ror = {}
                for k_n in np.arange(0.1, 1.0, 0.1):
                    ror[k_n] = get_ror(k_n, ticker)

                k = max(ror.keys(), key=lambda k : ror[k])


                target_price = get_target_price(ticker, k)
                ma15 = get_ma15(ticker)
                current_price = get_current_price(ticker)

                if target_price < current_price and ma15 < current_price:
                    unit = upbit.get_amount('ALL')+ upbit.get_balance()
                    unit1 = unit *0.02
                    krw = get_balance("KRW")

                    if krw > unit1 and upbit.get_balance(ticker) ==0:
                        upbit.buy_market_order(ticker, unit1)


                time.sleep(0.1)
        else:
            for ticker in tickers:
                if upbit.get_balance(ticker) >0 :
                    upbit.sell_market_order(ticker, upbit.get_balance(ticker))
                time.sleep(0.2)
            bot.sendMessage(mc, "현재 자산은 %s입니다. " % upbit.get_balance())
            time.sleep(300)
        time.sleep(1)
    except Exception as e:
        print(e)
        time.sleep(1)
