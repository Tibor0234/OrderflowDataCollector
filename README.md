# Orderflow Data Collector

An asynchronous market data collector for Binance USDⓈ-M Futures instruments. It collects trading, order book, open interest, news and OHLCV data for the configured pairs and stores the data in either SQLite or PostgreSQL.

## Architecture

```mermaid
flowchart LR
    Binance[Binance USDⓈ-M Futures]
    Finnhub[Finnhub News API]

    Binance --> Collector[Data Collector]
    Finnhub --> Collector
    Collector --> DB[(SQLite / PostgreSQL)]
```

The collector consists of independent modules for the different data sources. Enabled modules run asynchronously and store their data in a common database.

## Data Processing

The collected data is used by the separate [Orderflow Trading Research](https://github.com/Tibor0234/OrderflowTradingBOT) project, which replays the historical data for strategy research, backtesting, analysis and machine learning.

## Requirements

- Python
- Python packages listed in `requirements.txt`
- An available SQLite database file or PostgreSQL database
- A Finnhub API key if news collection is enabled

## Installation and Running

Install the required dependencies:

```bash
pip install -r requirements.txt
```

Create a `.env` file in the project root according to the selected database backend, then start the application:

```bash
python main.py
```

### PostgreSQL

```dotenv
POSTGRES_URL=postgresql://username:password@localhost:5432/database
FINNHUB_API_KEY=your_finnhub_api_key
```

### SQLite

```dotenv
SQLITE_PATH=./orderflow.db
FINNHUB_API_KEY=your_finnhub_api_key
```

`FINNHUB_API_KEY` is only required when the `news` module is enabled.

The `database.backend` setting determines whether the application uses `POSTGRES_URL` or `SQLITE_PATH`.

The PostgreSQL database must exist before starting the application. Required tables and indexes are created automatically on startup.

## Configuration

Pairs, database backend, retention limits and collection modules can be configured in `config.yaml`.

Example configuration:

```yaml
database:
  backend: postgres

retention:
  max_sessions: 7

pairs:
  - BTCUSDT
  - ETHUSDT

modules:
  trades: true
  orderbook: true
  open_interest: true
  news: true
  ohlcv: true
```

The `backend` can be set to `postgres` or `sqlite`.

The switches under `modules` enable or disable individual data collection modules.

At startup, the retention mechanism removes the oldest sessions when the configured `max_sessions` limit is reached or exceeded.

## Collected Data

- **Trades:** Binance aggregated trade WebSocket stream.
- **Order Book:** Binance `depth20` WebSocket stream.
- **Open Interest:** Binance Futures API, queried once per minute.
- **News:** Cryptocurrency news from Finnhub, queried once per minute.
- **OHLCV:** Binance Futures klines; 4-hour candles for the previous 7 days and 30-minute candles for the previous 24 hours.
- **Instrument Metadata:** Binance Exchange Information API.

OHLCV data and instrument metadata are only requested when a new daily session is created. Other enabled collectors run continuously while the application is running.

Session, pair and data collection records are stored in the database. Application logs are written to the `logs/` directory.

## Planned Improvements

- Add automated tests for the main components.
- Abstract the data collection layer to support additional market data providers and exchanges instead of being limited to Binance Futures.
- Improve data validation and recovery handling for interrupted collection sessions.
