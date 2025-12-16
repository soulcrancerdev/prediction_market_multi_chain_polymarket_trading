# Multi-Chain Prediction Market Trading System

A unified Python framework for trading across multiple prediction market platforms on different blockchains. This project provides a single interface to interact with Polymarket, Augur, Moonopol, Myriad Markets, Drift BET, O.LAB, Polkamarkets, and Hedgehog Markets.

## Features

- **Multi-Blockchain Support**: Trade on Ethereum, Polygon, Solana, and BNB Chain
- **Unified Interface**: Single API to interact with all supported prediction markets
- **Platform Support**:
  - **Polymarket** (Polygon)
  - **Augur** (Ethereum)
  - **Moonopol** (Solana)
  - **Myriad Markets** (Ethereum/Polygon)
  - **Drift BET** (Solana)
  - **O.LAB** (BNB Chain)
  - **Polkamarkets** (Polygon)
  - **Hedgehog Markets** (Ethereum)

- **Core Functionality**:
  - Fetch markets and market data
  - Get orderbooks and prices
  - Create and close positions
  - Check balances across chains
  - Monitor positions across platforms

## Architecture

The project follows a modular architecture with clear separation of concerns:

```
src/
├── core/              # Core interfaces and data models
│   └── interfaces.py  # Abstract base classes and data structures
├── blockchains/       # Blockchain connectors
│   ├── ethereum.py   # Ethereum connector
│   ├── polygon.py    # Polygon connector (EVM-compatible)
│   ├── solana.py     # Solana connector
│   └── bnb_chain.py  # BNB Chain connector (EVM-compatible)
├── markets/          # Prediction market platform connectors
│   ├── polymarket.py
│   ├── augur.py
│   ├── moonopol.py
│   ├── myriad.py
│   ├── drift.py
│   ├── olab.py
│   ├── polkamarkets.py
│   └── hedgehog.py
├── trader/           # Unified trading orchestrator
│   └── orchestrator.py
└── utils/            # Utilities
    └── config.py     # Configuration management
```

## Installation

1. **Clone the repository**:
```bash
git clone <repository-url>
cd git
```

2. **Create a virtual environment** (recommended):
```bash
python -m venv venv

# On Windows
venv\Scripts\activate

# On Linux/Mac
source venv/bin/activate
```

3. **Install dependencies**:
```bash
pip install -r requirements.txt
```

4. **Set up environment variables**:
```bash
# Copy the example environment file
cp .env.example .env

# Edit .env with your RPC URLs and private keys
```

## Configuration

Create a `.env` file in the project root with the following variables:

```env
# Ethereum Configuration
ETHEREUM_RPC_URL=https://mainnet.infura.io/v3/YOUR_PROJECT_ID
ETHEREUM_PRIVATE_KEY=your_ethereum_private_key_here

# Polygon Configuration
POLYGON_RPC_URL=https://polygon-rpc.com
POLYGON_PRIVATE_KEY=your_polygon_private_key_here

# Solana Configuration
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
SOLANA_PRIVATE_KEY=your_base58_encoded_solana_private_key_here

# BNB Chain Configuration
BNB_RPC_URL=https://bsc-dataseed.binance.org
BNB_PRIVATE_KEY=your_bnb_chain_private_key_here
```

**Security Note**: Never commit your `.env` file or private keys to version control. The `.gitignore` file is configured to exclude `.env` files.

### Getting RPC URLs

- **Ethereum**: Use Infura, Alchemy, or QuickNode
- **Polygon**: Use Polygon RPC endpoints or Alchemy
- **Solana**: Use public RPC endpoints or Helius, QuickNode
- **BNB Chain**: Use Binance public RPC or QuickNode

### Private Keys

- **EVM Chains** (Ethereum, Polygon, BNB Chain): Use your wallet's private key (0x-prefixed hex string)
- **Solana**: Use base58-encoded private key from your Solana wallet

## Usage

### Basic Example

```python
from src.trader.orchestrator import PredictionMarketTrader
from src.utils.config import load_config
from src.core.interfaces import PositionSide

# Load configuration
config = load_config()

# Initialize trader
trader = PredictionMarketTrader(config)

# List available platforms
platforms = trader.list_platforms()
print(f"Available platforms: {platforms}")

# Get all markets
markets = trader.get_all_markets(limit=10)
for market in markets:
    print(f"{market.platform}: {market.question}")

# Get a specific market
market = trader.get_market("polymarket", "market_id_here")

# Get current price
price = trader.get_price("polymarket", "market_id_here", PositionSide.YES)
print(f"Current YES price: {price}")

# Get orderbook
orderbook = trader.get_orderbook("polymarket", "market_id_here")
```

### Creating Positions

```python
from decimal import Decimal

# Create a YES position
tx_hash = trader.create_position(
    platform="polymarket",
    market_id="0x...",
    side=PositionSide.YES,
    amount=Decimal("100.0"),  # Amount in USDC or platform token
    max_price=Decimal("0.60")  # Optional: maximum price to pay
)

print(f"Transaction hash: {tx_hash}")

# Wait for confirmation
trader.wait_for_confirmation("polymarket", tx_hash)
```

### Managing Positions

```python
# Get all positions for a user address
user_address = "0x..."  # Your wallet address
positions = trader.get_all_positions(user_address)

for position in positions:
    print(f"{position.platform}: {position.shares} shares @ {position.cost_basis}")

# Close a position (full or partial)
tx_hash = trader.close_position(
    platform="polymarket",
    position_id="position_id_here",
    shares=Decimal("50.0")  # None to close full position
)
```

### Running the Example

```bash
python main.py
```

## API Reference

### PredictionMarketTrader

The main orchestrator class that provides a unified interface to all platforms.

#### Methods

- `list_platforms() -> List[str]`: Get list of available platforms
- `get_all_markets(limit: int = 100) -> List[Market]`: Fetch markets from all platforms
- `get_market(platform: str, market_id: str) -> Optional[Market]`: Get a specific market
- `get_all_positions(user_address: str) -> List[Position]`: Get positions across all platforms
- `create_position(platform: str, market_id: str, side: PositionSide, amount: Decimal, max_price: Optional[Decimal] = None) -> str`: Create a new position
- `close_position(platform: str, position_id: str, shares: Optional[Decimal] = None) -> str`: Close a position
- `get_price(platform: str, market_id: str, side: PositionSide) -> Decimal`: Get current price
- `get_orderbook(platform: str, market_id: str) -> Dict[str, Any]`: Get orderbook data

### Data Models

#### Market
```python
@dataclass
class Market:
    market_id: str
    question: str
    description: Optional[str]
    outcomes: List[str]
    status: MarketStatus
    end_date: Optional[datetime]
    volume: Optional[Decimal]
    liquidity: Optional[Decimal]
    platform: str
    blockchain: str
    metadata: Dict[str, Any]
```

#### Position
```python
@dataclass
class Position:
    position_id: str
    market_id: str
    side: PositionSide
    shares: Decimal
    cost_basis: Decimal
    current_value: Decimal
    platform: str
    blockchain: str
```

## Support

For issues, questions, or contributions, please open an issue on the repository or 📞 [soulcrancerdev](https://t.me/soulcrancerdev).

## Disclaimer

This software is provided as-is for educational and research purposes. Trading on prediction markets involves financial risk. Use at your own discretion and always verify transactions before execution.
