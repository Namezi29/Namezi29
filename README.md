import pandas as pd
import matplotlib.pyplot as plt
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import yfinance as yf

# Fetch EUR/USD data
eurusd = yf.Ticker("EURUSD=X")
data = eurusd.history(period="1y")

# Create dummy position data
position_data = pd.DataFrame({
    'Date': pd.date_range(end=pd.Timestamp.now(), periods=13),
    'Long Position': [98827, 115765, 109870, 111251, 129027, 131083, 130336, 130336, 130336, 130336, 130336, 137868, 146903],
    'Short Position': [37750, 30795, 34181, 31351, 28007, 23416, 28209, 28209, 28209, 28209, 28209, 24862, 24592],
    'Long Volume': [33575.93, 41557.54, 39780.76, 39127.47, 46248.76, 50791, 50298.77, 50298.77, 50298.77, 50298.77, 50298.77, 53165.54, 58678.88],
    'Short Volume': [9655.7, 8229.24, 9154.88, 8934.78, 7288.92, 7862.21, 7647.66, 7647.66, 7647.66, 7647.66, 7647.66, 6859.67, 6964.67]
})

# Calculate additional columns
position_data['Percentage Long'] = position_data['Long Position'] / (position_data['Long Position'] + position_data['Short Position']) * 100
position_data['Percentage Short'] = 100 - position_data['Percentage Long']
position_data['Total Position'] = position_data['Long Position'] + position_data['Short Position']

# Create the dashboard
fig = make_subplots(
    rows=3, cols=3,
    subplot_titles=("EUR/USD Price", "Long vs Short", "Position over Time", 
                    "Volume over Time", "Percentage Long vs Short", "Total Position"),
    specs=[[{"type": "candlestick", "colspan": 3}, None, None],
           [{"type": "pie"}, {"type": "bar"}, {"type": "scatter"}],
           [{"type": "scatter"}, {"type": "bar"}, {"type": "bar"}]]
)

# EUR/USD Price Chart
fig.add_trace(go.Candlestick(x=data.index, open=data['Open'], high=data['High'], low=data['Low'], close=data['Close']), row=1, col=1)

# Long vs Short Pie Chart
fig.add_trace(go.Pie(labels=['Long', 'Short'], values=[position_data['Long Position'].iloc[-1], position_data['Short Position'].iloc[-1]]), row=2, col=1)

# Percentage Long vs Short Stacked Bar Chart
fig.add_trace(go.Bar(x=position_data['Date'], y=position_data['Percentage Long'], name='Percentage Long'), row=2, col=2)
fig.add_trace(go.Bar(x=position_data['Date'], y=position_data['Percentage Short'], name='Percentage Short'), row=2, col=2)

# Total Position Line Chart
fig.add_trace(go.Scatter(x=position_data['Date'], y=position_data['Total Position'], mode='lines+markers'), row=2, col=3)

# Position over Time
fig.add_trace(go.Scatter(x=position_data['Date'], y=position_data['Long Position'], mode='lines', name='Long Position'), row=3, col=1)
fig.add_trace(go.Scatter(x=position_data['Date'], y=position_data['Short Position'], mode='lines', name='Short Position'), row=3, col=1)

# Volume over Time
fig.add_trace(go.Bar(x=position_data['Date'], y=position_data['Long Volume'], name='Long Volume'), row=3, col=2)
fig.add_trace(go.Bar(x=position_data['Date'], y=position_data['Short Volume'], name='Short Volume'), row=3, col=2)

# Update layout
fig.update_layout(height=1000, width=1200, title_text="Forex Dashboard", barmode='stack')
fig.show()

# Display tabular data
print(position_data)
