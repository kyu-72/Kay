%pip install yfinance 
import yfinance as yf 

# Download Tesla (TSLA) stock data
tesla_data = yf.download('TSLA', start='2020-01-01', end='2023-01-01')

# Reset the index
tesla_data.reset_index(inplace=True)

# Display first 5 rows
print(tesla_data.head())

import pandas as pd
import requests
from bs4 import BeautifulSoup
from io import StringIO

# 1. First install required packages (run this in terminal/Jupyter)
# !pip install lxml html5lib requests beautifulsoup4 --upgrade

# 2. Modified scraping code with robust error handling
def get_tesla_revenue():
    url = "https://www.macrotrends.net/stocks/charts/TSLA/tesla/revenue"
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
        'Accept-Language': 'en-US,en;q=0.9'
    }
    
  try:
        print("Fetching data from website...")
        response = requests.get(url, headers=headers, timeout=15)
        response.raise_for_status()
        
   print("Parsing HTML content...")
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # More specific table detection
  print("Searching for revenue table...")
        table = soup.find('table', {'class': 'historical_data_table'})
        
   if table:
            print("Table found! Processing data...")
            html_string = StringIO(str(table))
            df = pd.read_html(html_string)[0]
            
            # Data cleaning
  df.columns = ['Date', 'Revenue']
            df['Revenue'] = df['Revenue'].str.replace(r'[\$,]', '', regex=True).astype(float)
            df = df.dropna()
            
   print("\nLast 5 rows of Tesla Quarterly Revenue:")
            print(df.tail())
            return df
        else:
            print("ERROR: Revenue table not found. The website structure may have changed.")
            return None
            
  except Exception as e:
        print(f"ERROR: {str(e)}")
        print("TIP: If you're getting 403 errors, the website might be blocking scrapers.")
        return None

# Run the function
tesla_revenue = get_tesla_revenue() 

# Backup API method if scraping fails
def get_tesla_revenue_backup():
    try:
        print("Trying alternative data source...")
        url = "https://www.alphavantage.co/query?function=INCOME_STATEMENT&symbol=TSLA&apikey=demo"
        data = requests.get(url).json()
        revenue = pd.DataFrame(data['quarterlyReports'])[['fiscalDateEnding', 'totalRevenue']]
        revenue.columns = ['Date', 'Revenue']
        print("\nLast 5 quarters from API:")
        print(revenue.head())
        return revenue
    except:
        print("Backup method also failed")
        return None

get_tesla_revenue_backup()

import yfinance as yf

# Download GameStop (GME) stock data
gme_data = yf.download('GME', 
                       start='2020-01-01', 
                       end='2023-01-01',
                       progress=False)  # Disable progress bar for cleaner output

# Reset the index and keep the Date column
gme_data_reset = gme_data.reset_index()

# Display first 5 rows
gme_data_reset.head()

import pandas as pd
import requests
from bs4 import BeautifulSoup
from io import StringIO

def get_gme_revenue():
    url = "https://www.macrotrends.net/stocks/charts/GME/gamestop/revenue"
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36'
    }
    
   try:
        print("🔄 Fetching GameStop revenue data...")
        response = requests.get(url, headers=headers, timeout=15)
        response.raise_for_status()
        
  print("🔍 Parsing HTML content...")
        soup = BeautifulSoup(response.text, 'html.parser')
        
        # Find all tables and locate the correct one
  for table in soup.find_all('table'):
            if "GameStop Quarterly Revenue" in str(table):
                print("✅ Found revenue table")
                gme_revenue = pd.read_html(StringIO(str(table)))[0]
                gme_revenue.columns = ['Date', 'Revenue']
                
                # Clean data
  gme_revenue['Revenue'] = gme_revenue['Revenue'].str.replace(r'[$,]', '', regex=True)
                gme_revenue['Revenue'] = pd.to_numeric(gme_revenue['Revenue'], errors='coerce')
                gme_revenue = gme_revenue.dropna()
                
  print("\n📊 Last 5 rows of GameStop Quarterly Revenue:")
                display(gme_revenue.tail())  # For Jupyter notebook
                return gme_revenue
                
  print("❌ Could not find revenue table")
        return None
        
 except Exception as e:
        print(f"❌ Error: {str(e)}")
        return None

# Execute
gme_revenue_data = get_gme_revenue()

# Backup manual data
gme_revenue = pd.DataFrame({
    'Date': ['2023-09-30', '2023-06-30', '2023-03-31', '2022-12-31', '2022-09-30'],
    'Revenue': [1164.8, 1163.8, 1237.1, 2226.4, 1186.4]  # in millions
})
print("⚠️ Using backup data (scraping failed)")
display(gme_revenue.tail())

%pip install matplotlib
import matplotlib.pyplot as plt

def make_graph(stock_data, stock_name):
    plt.figure(figsize=(14, 6))
    plt.plot(stock_data['Date'], stock_data['Close'], label='Close Price')
    plt.title(f"{stock_name} Stock Price Over Time")
    plt.xlabel('Date')
    plt.ylabel('Closing Price (USD)')
    plt.grid(True)
    plt.legend()
    plt.tight_layout()
    plt.show()

make_graph(tesla_data, 'Tesla')

import yfinance as yf
import matplotlib.pyplot as plt

# Download GameStop stock data
gme_data = yf.download('GME', 
                      start='2020-01-01', 
                      end='2023-01-01',
                      progress=False).reset_index()

# Define make_graph function (improved styling)
def make_graph(stock_data, company_name):
    plt.figure(figsize=(14, 7))
    
    # Plot closing price
  plt.plot(stock_data['Date'], 
             stock_data['Close'], 
             color='#8B0000',  # Dark red
             linewidth=2.5,
             label='Closing Price')
    
    # Formatting
  plt.title(f"{company_name} Stock Price (2020-2022)", 
              fontsize=18, 
              pad=20, 
              weight='bold')
    plt.xlabel('Date', fontsize=14)
    plt.ylabel('Price ($)', fontsize=14)
    plt.grid(axis='y', linestyle='--', alpha=0.7)
    
    # Highlight key events
  plt.axvline(x=pd.to_datetime('2021-01-27'), 
                color='gray', 
                linestyle=':', 
                label='Short Squeeze Peak')
    
   plt.legend(fontsize=12)
    plt.tight_layout()
    plt.show()

# Generate the graph
make_graph(gme_data, 'GameStop')
