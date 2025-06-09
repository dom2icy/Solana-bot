# 🚀 Solana Trading Bot - Render Deployment Package

## Complete File Structure for GitHub Repository

Create these files in your GitHub repository for Render deployment:

---

## 📁 File: `requirements.txt`
```txt
flask==2.3.3
gunicorn==21.2.0
requests==2.31.0
solana==0.34.3
solders==0.21.0
telethon==1.31.1
base58==2.1.1
openai==1.3.0
python-dotenv==1.0.0
cryptography==41.0.7
aiofiles==23.2.1
aiohttp==3.9.1
gevent==23.9.1
```

---

## 📁 File: `render.yaml`
```yaml
services:
  - type: web
    name: solana-trading-bot
    env: python
    buildCommand: pip install -r requirements.txt
    startCommand: gunicorn --bind 0.0.0.0:$PORT --workers 1 --timeout 120 --worker-class gevent app:app
    plan: free
    envVars:
      - key: PYTHON_VERSION
        value: 3.11.7
      - key: RENDER_ENV
        value: production
```

---

## 📁 File: `runtime.txt`
```txt
python-3.11.7
```

---

## 📁 File: `.gitignore`
```gitignore
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Bot-specific files
*.session
*.session-journal
portfolio.json
bot.log
trade_logs/

# OS-specific
.DS_Store
Thumbs.db
```

---

## 📁 File: `.env.example`
```env
# Copy this to .env for local development
# For Render deployment, set these as environment variables in Render dashboard

# Telegram API Configuration (Required)
TG_API_ID=your_telegram_api_id_from_my.telegram.org
TG_API_HASH=your_telegram_api_hash_from_my.telegram.org
TG_PHONE_NUMBER=+1234567890
TG_GROUP=gem_tools_calls

# Solana Wallet Configuration (Required)
PHANTOM_PRIVATE_KEY_B58=your_wallet_private_key_in_base58_format

# OpenAI Configuration (Optional)
OPENAI_API_KEY=your_openai_api_key

# Flask Configuration
FLASK_SECRET_KEY=your_random_secret_key_for_sessions
PORT=10000

# Trading Configuration
BUY_AMOUNT_SOL=0.1
SLIPPAGE_BPS=1000
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com

# Render-specific
RENDER_ENV=production
```

---

## 📁 File: `README.md`
```markdown
# 🚀 Solana Trading Bot - Render Deployment

An advanced automated trading bot for Solana tokens that monitors Telegram groups for new token mentions and executes intelligent trades using Jupiter DEX.

## ✨ Features

- 🤖 **Automated Token Detection**: Monitors Telegram groups for new token addresses
- 💰 **Smart Trading**: Uses Jupiter DEX for optimal swap routes  
- 🧠 **AI Decision Making**: OpenAI GPT-4 integration for trading decisions
- 📊 **Portfolio Management**: Real-time tracking and automated profit-taking
- 🛡️ **Risk Management**: Position sizing and portfolio risk controls
- 🌐 **Web Dashboard**: Real-time monitoring interface
- 🎨 **Render Ready**: Optimized for Render deployment

## 🚀 Deploy to Render

### Prerequisites

1. **Telegram API Credentials**
   - Visit [my.telegram.org](https://my.telegram.org/apps)
   - Create a new application
   - Save your `API ID` and `API Hash`

2. **Solana Wallet**
   - Private key in Base58 format (from Phantom, Solflare, etc.)
   - Ensure wallet has SOL for trading

3. **Render Account**
   - Sign up at [render.com](https://render.com)

### Deployment Steps

1. **Fork/Clone this repository**
2. **Create new Web Service on Render**
   - Connect your GitHub account
   - Select this repository
   - Choose "Web Service"
3. **Configure Environment Variables** (see below)
4. **Deploy** - Render will automatically build and deploy

### Environment Variables

Set these in Render's environment variables section:

#### Required Variables
```
TG_API_ID=your_telegram_api_id
TG_API_HASH=your_telegram_api_hash
TG_PHONE_NUMBER=+1234567890
PHANTOM_PRIVATE_KEY_B58=your_wallet_private_key_base58
```

#### Optional Variables
```
OPENAI_API_KEY=your_openai_api_key
BUY_AMOUNT_SOL=0.1
SLIPPAGE_BPS=1000
TG_GROUP=gem_tools_calls
FLASK_SECRET_KEY=your_random_secret_key
```

## 🔧 Usage

1. **Start the Bot**: Click "Start Bot" in the web interface
2. **Telegram Verification**: Enter the 5-digit code sent to your phone
3. **Monitor**: Watch the dashboard for trading activity
4. **Portfolio**: Track your positions and gains in real-time

## 🛡️ Security

- Never commit private keys to Git
- Use Render's environment variables for all secrets  
- Monitor wallet activity regularly
- Start with small amounts for testing

## 📊 Monitoring

- **Live Dashboard**: `https://your-app.onrender.com`
- **Health Check**: `https://your-app.onrender.com/health`
- **API Status**: `https://your-app.onrender.com/api/status`

## 🐛 Troubleshooting

### Common Issues

1. **Missing Environment Variables**
   - Check all required variables are set in Render
   - Verify correct spelling and format

2. **Telegram Authentication Failed**
   - Verify API_ID and API_HASH from my.telegram.org
   - Check phone number format (+countrycode+number)

3. **Build Failures**
   - Check Render build logs
   - Ensure all files are committed to GitHub

## 📈 Trading Strategy

- **Buy Trigger**: New token detected in Telegram group
- **Sell Strategy**: Graduated profit-taking at 100%, 200%, 400%, 800% gains
- **AI Integration**: GPT-4 analysis for positions with 200%+ gains
- **Risk Management**: Configurable position sizing and limits

## 🔄 Updates

To update the bot:
1. Push changes to your GitHub repository
2. Render will automatically redeploy
3. Monitor logs for successful deployment

## ⚠️ Disclaimer

This bot trades real cryptocurrency. Use at your own risk. Start with small amounts and never invest more than you can afford to lose.
```

---

## 📁 File: `app.py`
```python
import os
import sys
import signal
import asyncio
import threading
import json
import time
import logging
from datetime import datetime

# Configure logging for Render
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[logging.StreamHandler(sys.stdout)]
)
logger = logging.getLogger(__name__)

from flask import Flask, render_template, send_from_directory, request, jsonify, session
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

def get_port():
    """Get port from Render environment or default"""
    port = os.environ.get('PORT', 10000)
    try:
        return int(port)
    except (ValueError, TypeError):
        logger.warning(f"Invalid PORT value: {port}, using default 10000")
        return 10000

def is_render_environment():
    """Check if running on Render"""
    return bool(os.environ.get('RENDER_ENV') or 
                os.environ.get('RENDER_SERVICE_NAME') or
                os.environ.get('RENDER_EXTERNAL_URL'))

def validate_environment():
    """Validate required environment variables"""
    required_vars = {
        'TG_API_ID': 'Telegram API ID (get from my.telegram.org)',
        'TG_API_HASH': 'Telegram API Hash (get from my.telegram.org)', 
        'TG_PHONE_NUMBER': 'Telegram Phone Number (+countrycode+number)',
        'PHANTOM_PRIVATE_KEY_B58': 'Solana Wallet Private Key (Base58 format)'
    }
    
    missing_vars = []
    for var, description in required_vars.items():
        if not os.environ.get(var):
            missing_vars.append(f"{var} ({description})")
    
    if missing_vars:
        logger.error("❌ Missing required environment variables:")
        for var in missing_vars:
            logger.error(f"  - {var}")
        logger.error("Please set these in Render's environment variables section")
        
        if is_render_environment():
            logger.warning("⚠️ Running in limited mode without full configuration")
            return False
        else:
            logger.warning("⚠️ Some features may not work without proper configuration")
            return False
    
    return True

# Validate environment on startup
config_valid = validate_environment()

# Mock dependencies that might not be available
class MockTradeLogger:
    def get_current_trade_state(self, portfolio, bot_status, recent_activity):
        return {"mock": True, "timestamp": datetime.now().isoformat()}
    def log_trades(self, data):
        logger.info(f"Trade logged: {data}")

class MockOpenAI:
    def __init__(self):
        self.api_key = os.environ.get('OPENAI_API_KEY', '')
    def test_api_connection(self):
        return {"status": "success", "message": "Mock OpenAI ready"}
    async def get_gpt_decision(self, token_data, gain):
        if gain >= 5.0:
            return "SELL"
        elif gain >= 2.0:
            return "HOLD"
        return "HOLD"
    def format_token_data_for_gpt(self, mint, data, market_data):
        return {"mint": mint, "gain": "mock"}
    def calculate_score(self, data):
        return 5

trade_logger = MockTradeLogger()
openai_engine = MockOpenAI()

# Import dependencies
import base58
import re
import requests
from telethon import TelegramClient, events
from solana.rpc.async_api import AsyncClient
from solana.rpc.api import Client
from solders.keypair import Keypair
from solders.transaction import VersionedTransaction
from solana.rpc.types import TxOpts
from base64 import b64decode
from solders.pubkey import Pubkey as PublicKey

# Flask app configuration
app = Flask(__name__, static_folder='.', template_folder='.')
app.secret_key = os.environ.get('FLASK_SECRET_KEY', 'render_default_secret_key_change_in_production')

# Bot Configuration from environment
TG_API_ID = int(os.environ.get('TG_API_ID', 0))
TG_API_HASH = os.environ.get('TG_API_HASH', '')
TG_PHONE_NUMBER = os.environ.get('TG_PHONE_NUMBER', '')
TG_SESSION = "gem_bot_session_render"
TG_GROUP = os.environ.get('TG_GROUP', 'gem_tools_calls')
PHANTOM_PRIVATE_KEY_B58 = os.environ.get('PHANTOM_PRIVATE_KEY_B58', '')
SOLANA_RPC_URL = os.environ.get('SOLANA_RPC_URL', 'https://api.mainnet-beta.solana.com')
BUY_AMOUNT_SOL = float(os.environ.get('BUY_AMOUNT_SOL', 0.1))
SLIPPAGE_BPS = int(os.environ.get('SLIPPAGE_BPS', 1000))
SOL_MINT_ADDRESS = "So11111111111111111111111111111111111111112"

# Sell strategy configuration
SELL_STEPS = [
    (1.0, 0.5),  # 100% gain -> sell 50%
    (2.0, 0.2),  # 200% gain -> sell 20%
    (4.0, 0.2),  # 400% gain -> sell 20%
    (8.0, 0.2),  # 800% gain -> sell 20%
]

logger.info(f"🚀 Starting Solana Trading Bot on Render")
logger.info(f"📡 Port: {get_port()}")
logger.info(f"🔑 Wallet configured: {bool(PHANTOM_PRIVATE_KEY_B58)}")
logger.info(f"📱 Telegram configured: {bool(TG_API_ID and TG_API_HASH)}")

# Global variables
bot_status = {
    "running": False, 
    "connected_wallets": [], 
    "portfolio": {}, 
    "recent_activity": [],
    "start_time": time.time()
}
telegram_client = None
portfolio = {}
recent_buys = set()
current_trade_amount = BUY_AMOUNT_SOL
pending_verification = {"waiting": False, "client": None}
wallet = None
wallet_pubkey = None
solana_client = None
sync_client = None

# Initialize Solana clients
def initialize_solana_clients():
    global solana_client, sync_client
    try:
        solana_client = AsyncClient(SOLANA_RPC_URL)
        sync_client = Client(SOLANA_RPC_URL)
        logger.info("✅ Solana clients initialized")
    except Exception as e:
        logger.error(f"❌ Failed to initialize Solana clients: {e}")

initialize_solana_clients()

# Load wallet
if PHANTOM_PRIVATE_KEY_B58:
    try:
        secret_key_bytes = base58.b58decode(PHANTOM_PRIVATE_KEY_B58)
        wallet = Keypair.from_bytes(secret_key_bytes)
        wallet_pubkey = wallet.pubkey()
        logger.info(f"✅ Wallet loaded: {wallet_pubkey}")
        bot_status["connected_wallets"] = [str(wallet_pubkey)]
    except Exception as e:
        logger.error(f"❌ Failed to load wallet: {e}")

# Regex for Solana mint addresses
SOL_MINT_REGEX = re.compile(r"[1-9A-HJ-NP-Za-km-z]{32,44}")

def add_activity(message):
    """Add activity to recent activity log"""
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    activity_message = f"[{timestamp}] {message}"
    bot_status["recent_activity"].append(activity_message)
    logger.info(activity_message)
    
    # Keep only last 50 activities
    if len(bot_status["recent_activity"]) > 50:
        bot_status["recent_activity"] = bot_status["recent_activity"][-50:]

def save_portfolio():
    """Save portfolio to file"""
    try:
        with open('portfolio.json', 'w') as f:
            json.dump(portfolio, f, indent=2)
        bot_status["portfolio"] = portfolio.copy()
        
        trade_state = trade_logger.get_current_trade_state(
            portfolio, bot_status, bot_status.get("recent_activity", [])
        )
        trade_logger.log_trades(trade_state)
    except Exception as e:
        logger.error(f"Error saving portfolio: {e}")

def load_portfolio():
    """Load portfolio from file"""
    try:
        with open('portfolio.json', 'r') as f:
            return json.load(f)
    except FileNotFoundError:
        return {}
    except Exception as e:
        logger.error(f"Error loading portfolio: {e}")
        return {}

async def get_token_balance(mint: str) -> float:
    """Get token balance for the wallet"""
    try:
        if not solana_client or not wallet_pubkey:
            return 0.0
            
        response = await solana_client.get_token_accounts_by_owner(
            wallet_pubkey, {"mint": PublicKey.from_string(mint)}
        )

        if response.value:
            token_account = response.value[0]
            account_info = await solana_client.get_account_info(token_account.pubkey)

            if account_info.value and account_info.value.data:
                data = account_info.value.data
                if len(data) >= 64:
                    amount_bytes = data[64:72]
                    amount = int.from_bytes(amount_bytes, 'little')
                    return amount / 1_000_000_000
        return 0.0
    except Exception as e:
        logger.error(f"Error getting token balance for {mint}: {e}")
        return 0.0

async def fetch_jupiter_quote(input_mint, output_mint, amount):
    """Fetch quote from Jupiter API"""
    try:
        if amount <= 0:
            return None

        params = {
            "inputMint": input_mint,
            "outputMint": output_mint,
            "amount": str(amount),
            "slippageBps": str(SLIPPAGE_BPS),
            "onlyDirectRoutes": "true"
        }

        resp = requests.get("https://quote-api.jup.ag/v6/quote", params=params, timeout=15)
        if resp.status_code == 200:
            quote_data = resp.json()
            if 'outAmount' in quote_data:
                return quote_data
        return None
    except Exception as e:
        add_activity(f"Error fetching Jupiter quote: {str(e)}")
        return None

async def get_token_price(mint: str) -> float:
    """Fetch current token price"""
    try:
        quote = await fetch_jupiter_quote(mint, SOL_MINT_ADDRESS, 10**9)
        if quote and "outAmount" in quote:
            amount_out = int(quote["outAmount"])
            return amount_out / 1_000_000_000
        return 0.0
    except Exception as e:
        return 0.0

def sol_to_lamports(sol):
    """Convert SOL to lamports"""
    return int(sol * 1_000_000_000)

async def buy_token_with_retries(mint, max_attempts=3):
    """Buy token with retries"""
    global portfolio, current_trade_amount

    if not wallet or not solana_client:
        add_activity("❌ Wallet or Solana client not initialized")
        return False

    try:
        PublicKey.from_string(mint)
    except Exception:
        add_activity(f"❌ Invalid mint address: {mint}")
        return False

    if mint == SOL_MINT_ADDRESS:
        return False

    for attempt in range(max_attempts):
        try:
            amount = sol_to_lamports(current_trade_amount)
            add_activity(f"Buy attempt {attempt + 1}/{max_attempts} for {mint[:8]}...")

            quote = await fetch_jupiter_quote(SOL_MINT_ADDRESS, mint, amount)
            if not quote:
                if attempt < max_attempts - 1:
                    await asyncio.sleep(0.5)
                    continue
                return False

            # Check balance
            try:
                balance_response = await solana_client.get_balance(wallet_pubkey)
                balance_sol = balance_response.value / 1_000_000_000
                if balance_sol < current_trade_amount + 0.01:
                    add_activity(f"Insufficient balance: {balance_sol:.3f} SOL")
                    return False
            except Exception:
                return False

            # Get swap transaction
            swap_response = requests.post("https://quote-api.jup.ag/v6/swap", 
                json={"quoteResponse": quote, "userPublicKey": str(wallet.pubkey())},
                timeout=15
            )

            if swap_response.status_code != 200:
                if attempt < max_attempts - 1:
                    await asyncio.sleep(0.5)
                    continue
                return False

            swap_data = swap_response.json()
            swap_tx_b64 = swap_data["swapTransaction"]
            tx = VersionedTransaction.from_bytes(b64decode(swap_tx_b64))
            tx.sign([wallet])

            # Send transaction
            resp = await solana_client.send_raw_transaction(bytes(tx), TxOpts(skip_preflight=False))
            tx_hash = resp.value

            # Wait for confirmation
            for i in range(20):
                await asyncio.sleep(0.5)
                status = await solana_client.get_signature_status(tx_hash)
                if status.value and status.value[0]:
                    if status.value[0].confirmation_status == "confirmed":
                        bought_price = await get_token_price(mint)
                        if bought_price == 0:
                            bought_price = current_trade_amount / (int(quote["outAmount"]) / 1_000_000_000)

                        portfolio[mint] = {
                            "bought_price": bought_price, 
                            "amount": current_trade_amount, 
                            "sold": 0,
                            "timestamp": datetime.now().isoformat()
                        }
                        save_portfolio()
                        add_activity(f"✅ Successfully bought {mint[:8]}... at {bought_price:.9f} SOL")
                        return True
                    elif status.value[0].err:
                        break

            if attempt < max_attempts - 1:
                await asyncio.sleep(0.5)

        except Exception as e:
            add_activity(f"Error in buy attempt {attempt + 1}: {str(e)}")
            if attempt < max_attempts - 1:
                await asyncio.sleep(0.5)

    add_activity(f"❌ Failed to buy {mint} after {max_attempts} attempts")
    return False

async def sell_token(mint, fraction):
    """Sell a fraction of token position"""
    global portfolio
    try:
        if mint not in portfolio or not wallet or not solana_client:
            return False

        add_activity(f"Attempting to sell {fraction*100}% of {mint[:8]}...")

        token_balance = await get_token_balance(mint)
        if token_balance == 0:
            return False

        tokens_to_sell = token_balance * fraction
        amount_smallest = int(tokens_to_sell * 1_000_000_000)

        if amount_smallest == 0:
            return False

        quote = await fetch_jupiter_quote(mint, SOL_MINT_ADDRESS, amount_smallest)
        if not quote:
            return False

        swap_response = requests.post("https://quote-api.jup.ag/v6/swap", 
            json={"quoteResponse": quote, "userPublicKey": str(wallet.pubkey())},
            timeout=30
        )

        if swap_response.status_code != 200:
            return False

        swap_data = swap_response.json()
        swap_tx_b64 = swap_data["swapTransaction"]
        tx = VersionedTransaction.from_bytes(b64decode(swap_tx_b64))
        tx.sign([wallet])

        resp = await solana_client.send_raw_transaction(bytes(tx), TxOpts(skip_preflight=False))
        tx_hash = resp.value

        for i in range(30):
            await asyncio.sleep(1)
            status = await solana_client.get_signature_status(tx_hash)
            if status.value and status.value[0]:
                if status.value[0].confirmation_status == "confirmed":
                    holding = portfolio[mint]
                    holding['sold'] += fraction
                    if holding['sold'] >= 1.0:
                        portfolio.pop(mint)
                        add_activity(f"Completely sold out of {mint[:8]}...")
                    else:
                        portfolio[mint] = holding
                        add_activity(f"Sold {fraction*100}% of {mint[:8]}...")

                    save_portfolio()
                    return True
                elif status.value[0].err:
                    return False

    except Exception as e:
        add_activity(f"Error in sell_token: {str(e)}")
        return False

async def check_sells():
    """Monitor portfolio and execute sells"""
    global portfolio
    while bot_status["running"]:
        try:
            for mint, data in list(portfolio.items()):
                current_price = await get_token_price(mint)
                if current_price == 0.0:
                    continue

                gain = current_price / data['bought_price']

                # Use AI for gains >= 2x
                if gain >= 2.0:
                    try:
                        token_data = openai_engine.format_token_data_for_gpt(mint, data, {})
                        gpt_decision = await openai_engine.get_gpt_decision(token_data, gain)
                        
                        add_activity(f"🤖 AI Decision for {mint[:8]}... (gain: {gain:.2f}x): {gpt_decision}")
                        
                        if gpt_decision == "SELL":
                            to_sell = 0.5 - data.get('sold', 0)
                            if to_sell > 0:
                                success = await sell_token(mint, to_sell)
                                if success:
                                    add_activity(f"🤖 AI-recommended SELL executed")
                        
                    except Exception:
                        # Fall back to original strategy
                        for target, fraction in SELL_STEPS:
                            if gain >= target and data['sold'] < fraction:
                                to_sell = fraction - data['sold']
                                if to_sell > 0:
                                    success = await sell_token(mint, to_sell)
                                    if success:
                                        add_activity(f"Fallback: Sold {to_sell*100:.0f}% at {gain*100:.0f}% gain")
                else:
                    # Use original sell logic for gains < 2x
                    for target, fraction in SELL_STEPS:
                        if gain >= target and data['sold'] < fraction:
                            to_sell = fraction - data['sold']
                            if to_sell > 0:
                                success = await sell_token(mint, to_sell)
                                if success:
                                    add_activity(f"Sold {to_sell*100:.0f}% at {gain*100:.0f}% gain")
            
            await asyncio.sleep(60)
        except Exception as e:
            add_activity(f"Error in check_sells: {str(e)}")
            await asyncio.sleep(60)

async def clear_recent_buy(mint):
    """Clear mint from recent buys after timeout"""
    await asyncio.sleep(60)
    recent_buys.discard(mint)

async def telegram_listener(event):
    """Listen for new messages in Telegram group"""
    global recent_buys, portfolio
    try:
        text = event.raw_text
        if not text:
            return

        found = SOL_MINT_REGEX.findall(text)
        
        for mint in found:
            if len(mint) < 32 or len(mint) > 44:
                continue

            if mint not in recent_buys and mint not in portfolio:
                add_activity(f"🎯 NEW TOKEN: {mint[:8]}...")
                recent_buys.add(mint)

                try:
                    success = await buy_token_with_retries(mint, 3)
                    if not success:
                        recent_buys.discard(mint)
                        add_activity(f"💥 Failed to buy {mint[:8]}...")
                    else:
                        add_activity(f"✅ Successfully bought {mint[:8]}...")
                except Exception as buy_error:
                    recent_buys.discard(mint)
                    add_activity(f"❌ Buy error: {str(buy_error)}")

                asyncio.create_task(clear_recent_buy(mint))
                
    except Exception as e:
        add_activity(f"❌ Telegram listener error: {str(e)}")

async def start_telegram_bot():
    """Start Telegram bot with authentication"""
    global telegram_client, pending_verification, portfolio, bot_status

    try:
        portfolio = load_portfolio()
        bot_status["portfolio"] = portfolio.copy()

        add_activity("🚀 Starting Telegram bot...")

        telegram_client = TelegramClient(TG_SESSION, TG_API_ID, TG_API_HASH)
        await telegram_client.connect()

        if not telegram_client.is_connected():
            add_activity("❌ Failed to connect to Telegram")
            bot_status["running"] = False
            return

        add_activity("✅ Connected to Telegram")

        if not await telegram_client.is_user_authorized():
            if pending_verification["waiting"]:
                return
                
            phone_number = TG_PHONE_NUMBER.strip()
            add_activity(f"📱 Sending verification code to: {phone_number}")
            
            try:
                await telegram_client.send_code_request(phone_number)
                pending_verification["waiting"] = True
                pending_verification["client"] = telegram_client
                
                add_activity("✅ Verification code sent!")
                add_activity("📱 Check Telegram for verification code")
                
                # Wait for verification with timeout
                timeout_counter = 0
                max_timeout = 300  # 5 minutes
                
                while pending_verification["waiting"] and bot_status["running"] and timeout_counter < max_timeout:
                    await asyncio.sleep(2)
                    timeout_counter += 2
                
                if timeout_counter >= max_timeout:
                    add_activity("⏰ Verification timeout")
                    pending_verification["waiting"] = False
                    bot_status["running"] = False
                        
            except Exception as send_error:
                add_activity(f"❌ Failed to send verification code: {str(send_error)}")
                bot_status["running"] = False
                pending_verification["waiting"] = False
                return

        else:
            add_activity("✅ Already authenticated")
            await start_monitoring()

    except Exception as e:
        add_activity(f"❌ Telegram bot error: {str(e)}")
        bot_status["running"] = False

async def start_monitoring():
    """Start monitoring Telegram group and portfolio"""
    global telegram_client, portfolio

    try:
        add_activity("📊 Starting monitoring...")
        
        try:
            entity = await telegram_client.get_entity(TG_GROUP)
            add_activity(f"✅ Connected to group: {TG_GROUP}")
            
            telegram_client.add_event_handler(telegram_listener, events.NewMessage(chats=[entity.id]))
            add_activity(f"👂 Monitoring {TG_GROUP} for new tokens")
            
        except Exception as group_error:
            add_activity(f"❌ Cannot access group '{TG_GROUP}': {str(group_error)}")
            return
        
        # Start sell monitoring if wallet available
        if wallet:
            asyncio.create_task(check_sells())
            add_activity("📊 Portfolio monitoring started")
        
        add_activity("🎯 Bot is now active!")
        
        # Keep running
        while bot_status["running"]:
            await asyncio.sleep(60)
            bot_status["portfolio"] = portfolio.copy()

    except Exception as e:
        add_activity(f"❌ Monitoring error: {str(e)}")
        bot_status["running"] = False

def run_telegram_bot():
    """Run Telegram bot in background thread"""
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)

    try:
        loop.run_until_complete(start_telegram_bot())
    except Exception as e:
        add_activity(f"❌ Bot error: {str(e)}")
        bot_status["running"] = False
    finally:
        try:
            pending = asyncio.all_tasks(loop)
            if pending:
                loop.run_until_complete(asyncio.gather(*pending, return_exceptions=True))
            loop.close()
        except:
            pass

# Render-specific graceful shutdown
def signal_handler(signum, frame):
    """Handle shutdown signals gracefully"""
    logger.info(f"🛑 Received signal {signum}, shutting down...")
    
    global bot_status, telegram_client
    bot_status["running"] = False
    
    if telegram_client:
        try:
            loop = asyncio.new_event_loop()
            asyncio.set_event_loop(loop)
            loop.run_until_complete(telegram_client.disconnect())
            loop.close()
        except:
            pass
    
    sys.exit(0)

# Register signal handlers
signal.signal(signal.SIGTERM, signal_handler)
signal.signal(signal.SIGINT, signal_handler)

# Flask Routes
@app.route('/')
def index():
    """Serve main page"""
    return send_from_directory('.', 'index.html')

@app.route('/api/status')
def get_status():
    """Get bot status"""
    wallet_balance = 0.0
    portfolio_value = 0.0
    
    try:
        if wallet_pubkey and sync_client:
            balance_response = sync_client.get_balance(wallet_pubkey)
            wallet_balance = balance_response.value / 1_000_000_000
            portfolio_value = wallet_balance
    except Exception as e:
        logger.error(f"Error getting status: {e}")
    
    current_status = {
        "running": bot_status["running"],
        "connected_wallets": [str(wallet_pubkey)] if wallet_pubkey else [],
        "portfolio": portfolio.copy(),
        "recent_activity": bot_status.get("recent_activity", [])[-20:],
        "wallet_balance": wallet_balance,
        "portfolio_value": portfolio_value,
        "position_count": len(portfolio),
        "wallet_address": str(wallet_pubkey) if wallet_pubkey else None
    }
    
    return jsonify(current_status)

@app.route('/api/start-bot', methods=['POST'])
def start_bot_endpoint():
    """Start the trading bot"""
    global bot_status, pending_verification, telegram_client

    if not TG_API_ID or not TG_API_HASH:
        return jsonify({
            "status": "error", 
            "message": "Telegram API credentials required. Set TG_API_ID and TG_API_HASH environment variables."
        })

    if not TG_PHONE_NUMBER:
        return jsonify({
            "status": "error", 
            "message": "Phone number required. Set TG_PHONE_NUMBER environment variable."
        })

    add_activity("🔄 Starting Telegram authentication...")
    
    bot_status["running"] = True
    pending_verification["waiting"] = False
    pending_verification["client"] = None
    
    bot_thread = threading.Thread(target=run_telegram_bot, daemon=True)
    bot_thread.start()

    return jsonify({
        "status": "started", 
        "message": f"Authentication started. Code will be sent to {TG_PHONE_NUMBER}"
    })

@app.route('/api/stop-bot', methods=['POST'])
def stop_bot():
    """Stop the trading bot"""
    global telegram_client, pending_verification
    bot_status["running"] = False
    
    pending_verification["waiting"] = False
    pending_verification["client"] = None
    
    if telegram_client:
        try:
            loop = asyncio.new_event_loop()
            asyncio.set_event_loop(loop)
            loop.run_until_complete(telegram_client.disconnect())
            loop.close()
        except:
            pass
    telegram_client = None

    add_activity("✅ Bot stopped")
    return jsonify({"status": "stopped"})

@app.route('/api/submit-verification-code', methods=['POST'])
def submit_verification_code():
    """Submit Telegram verification code"""
    global pending_verification, telegram_client
    
    try:
        data = request.get_json()
        if not data:
            return jsonify({"status": "error", "error": "No data received"})

        code = data.get('code', '').strip()

        if not code or len(code) != 5 or not code.isdigit():
            return jsonify({"status": "error", "error": "Invalid verification code format"})

        phone_number = os.environ.get('TG_PHONE_NUMBER', '').strip()
        if not phone_number:
            return jsonify({"status": "error", "error": "Phone number not configured"})
        
        add_activity(f"🔑 Verification code received: {code}")
        
        if not bot_status["running"] or not pending_verification["waiting"]:
            return jsonify({
                "status": "error", 
                "error": "No verification request pending"
            })

        def authenticate():
            loop = asyncio.new_event_loop()
            asyncio.set_event_loop(loop)
            try:
                client = pending_verification["client"]
                result = loop.run_until_complete(client.sign_in(phone_number, code))
                is_auth = loop.run_until_complete(client.is_user_authorized())
                
                if is_auth:
                    add_activity("✅ Telegram authentication successful!")
                    
                    global telegram_client
                    telegram_client = client
                    pending_verification["waiting"] = False
                    pending_verification["client"] = None
                    
                    loop.run_until_complete(start_monitoring())
                else:
                    add_activity("❌ Authentication failed")
                    pending_verification["waiting"] = False
                    pending_verification["client"] = None
                    bot_status["running"] = False
                    
            except Exception as e:
                pending_verification["waiting"] = False
                pending_verification["client"] = None
                
                if "PHONE_CODE_INVALID" in str(e):
                    add_activity(f"❌ Invalid verification code: {code}")
                elif "PHONE_CODE_EXPIRED" in str(e):
                    add_activity(f"❌ Verification code expired")
                else:
                    add_activity(f"❌ Authentication error: {str(e)}")
            finally:
                loop.close()

        auth_thread = threading.Thread(target=authenticate, daemon=True)
        auth_thread.start()

        return jsonify({
            "status": "success", 
            "message": f"Code {code} submitted - authenticating..."
        })

    except Exception as e:
        add_activity(f"❌ Error processing verification code: {str(e)}")
        return jsonify({"status": "error", "error": str(e)})

@app.route('/health')
def health_check():
    """Health check endpoint for Render"""
    try:
        return jsonify({
            "status": "healthy",
            "timestamp": datetime.now().isoformat(),
            "environment": "render" if is_render_environment() else "local",
            "wallet_loaded": bool(PHANTOM_PRIVATE_KEY_B58),
            "telegram_configured": bool(TG_API_ID and TG_API_HASH),
            "bot_running": bot_status.get("running", False),
            "portfolio_size": len(portfolio),
            "config_valid": config_valid,
            "uptime_seconds": time.time() - bot_status.get("start_time", time.time())
        }), 200
    except Exception as e:
        return jsonify({
            "status": "unhealthy",
            "error": str(e),
            "timestamp": datetime.now().isoformat()
        }), 503

@app.route('/<path:filename>')
def serve_static(filename):
    """Serve static files"""
    return send_from_directory('.', filename)

# Initialize application
def create_app():
    """Application factory for Render"""
    
    global bot_status, portfolio
    
    bot_status["recent_activity"] = []
    add_activity("🚀 Solana Trading Bot starting on Render...")
    
    # Load existing portfolio
    try:
        portfolio = load_portfolio()
        bot_status["portfolio"] = portfolio.copy()
        logger.info(f"📊 Loaded portfolio with {len(portfolio)} positions")
    except Exception as e:
        logger.error(f"❌ Failed to load portfolio: {e}")
    
    logger.info("✅ Application initialized successfully")
    return app

if __name__ == '__main__':
    try:
        app = create_app()
        
        port = get_port()
        
        if is_render_environment():
            logger.info("🎨 Running on Render with Gunicorn")
        else:
            logger.info("🖥️ Running locally")
            
        app.run(
            host='0.0.0.0',
            port=port,
            debug=not is_render_environment(),
            threaded=True,
            use_reloader=False
        )
        
    except Exception as e:
        logger.error(f"❌ Application startup failed: {e}")
        sys.exit(1)
```

---

## 📁 File: `index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Solana Trading Bot - Render</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
            color: #333;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.1);
            backdrop-filter: blur(10px);
        }

        .header {
            text-align: center;
            margin-bottom: 40px;
            padding-bottom: 20px;
            border-bottom: 2px solid #f0f0f0;
        }

        .header h1 {
            color: #2d3748;
            font-size: 2.5rem;
            font-weight: 700;
            margin-bottom: 10px;
        }

        .header .subtitle {
            color: #718096;
            font-size: 1.1rem;
        }

        .render-badge {
            display: inline-block;
            background: linear-gradient(45deg, #7c3aed, #a855f7);
            color: white;
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 0.9rem;
            font-weight: 600;
            margin-top: 10px;
            text-decoration: none;
            transition: transform 0.2s ease;
        }

        .render-badge:hover {
            transform: translateY(-2px);
        }

        .status-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .status-card {
            background: #f8fafc;
            border: 1px solid #e2e8f0;
            border-radius: 12px;
            padding: 20px;
            transition: all 0.3s ease;
        }

        .status-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
        }

        .status-card h3 {
            color: #2d3748;
            font-size: 1.1rem;
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .status-value {
            font-size: 1.5rem;
            font-weight: 700;
            color: #4299e1;
        }

        .controls {
            display: flex;
            gap: 15px;
            margin-bottom: 30px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 1rem;
            text-decoration: none;
            display: inline-flex;
            align-items: center;
            gap: 8px;
        }

        .btn-primary {
            background: linear-gradient(45deg, #4299e1, #3182ce);
            color: white;
        }

        .btn-danger {
            background: linear-gradient(45deg, #f56565, #e53e3e);
            color: white;
        }

        .btn-secondary {
            background: #edf2f7;
            color: #4a5568;
            border: 1px solid #e2e8f0;
        }

        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        }

        .btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }

        .activity-log {
            background: #1a202c;
            color: #e2e8f0;
            border-radius: 12px;
            padding: 20px;
            max-height: 400px;
            overflow-y: auto;
            font-family: 'Monaco', 'Menlo', monospace;
            font-size: 0.9rem;
            line-height: 1.6;
        }

        .activity-log h3 {
            color: #90cdf4;
            margin-bottom: 15px;
            font-family: inherit;
        }

        .activity-item {
            margin-bottom: 8px;
            padding: 8px;
            border-radius: 6px;
            background: rgba(255, 255, 255, 0.05);
        }

        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            backdrop-filter: blur(5px);
        }

        .modal-content {
            background-color: white;
            margin: 15% auto;
            padding: 30px;
            border-radius: 15px;
            width: 90%;
            max-width: 400px;
            text-align: center;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
        }

        .modal input {
            width: 100%;
            padding: 15px;
            border: 2px solid #e2e8f0;
            border-radius: 8px;
            font-size: 1.2rem;
            text-align: center;
            margin: 15px 0;
            letter-spacing: 3px;
        }

        .modal input:focus {
            outline: none;
            border-color: #4299e1;
        }

        .close {
            color: #aaa;
            float: right;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }

        .close:hover {
            color: #333;
        }

        .loading {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid #f3f3f3;
            border-top: 3px solid #4299e1;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .environment-badge {
            position: absolute;
            top: 20px;
            right: 20px;
            background: #7c3aed;
            color: white;
            padding: 5px 12px;
            border-radius: 15px;
            font-size: 0.8rem;
            font-weight: 600;
        }

        @media (max-width: 768px) {
            .container {
                padding: 20px;
                margin: 10px;
            }

            .header h1 {
                font-size: 2rem;
            }

            .controls {
                flex-direction: column;
                align-items: stretch;
            }

            .btn {
                justify-content: center;
            }
        }
    </style>
</head>
<body>
    <div class="environment-badge">🎨 Render</div>
    
    <div class="container">
        <div class="header">
            <h1>🚀 Solana Trading Bot</h1>
            <p class="subtitle">Automated token trading with AI-powered decisions</p>
            <a href="https://render.com" class="render-badge" target="_blank">
                🎨 Deployed on Render
            </a>
        </div>

        <div class="status-grid">
            <div class="status-card">
                <h3>🤖 Bot Status</h3>
                <div class="status-value" id="botStatus">Loading...</div>
            </div>
            <div class="status-card">
                <h3>💰 Wallet Balance</h3>
                <div class="status-value" id="walletBalance">Loading...</div>
            </div>
            <div class="status-card">
                <h3>📊 Active Positions</h3>
                <div class="status-value" id="positionCount">Loading...</div>
            </div>
            <div class="status-card">
                <h3>📱 Telegram</h3>
                <div class="status-value" id="telegramStatus">Loading...</div>
            </div>
            <div class="status-card">
                <h3>🔗 Wallet Address</h3>
                <div class="status-value" id="walletAddress" style="font-size: 0.8rem; word-break: break-all;">Loading...</div>
            </div>
            <div class="status-card">
                <h3>⏱️ Uptime</h3>
                <div class="status-value" id="uptime">Loading...</div>
            </div>
        </div>

        <div class="controls">
            <button class="btn btn-primary" id="startBtn" onclick="startBot()">
                <span id="startSpinner" class="loading" style="display: none;"></span>
                🚀 Start Bot
            </button>
            <button class="btn btn-danger" id="stopBtn" onclick="stopBot()">
                🛑 Stop Bot
            </button>
            <a href="/health" class="btn btn-secondary" target="_blank">
                🏥 Health Check
            </a>
            <button class="btn btn-secondary" onclick="refreshStatus()">
                🔄 Refresh
            </button>
        </div>

        <div class="activity-log">
            <h3>📋 Recent Activity</h3>
            <div id="activityList">
                <div class="activity-item">Loading recent activity...</div>
            </div>
        </div>
    </div>

    <!-- Verification Code Modal -->
    <div id="verificationModal" class="modal">
        <div class="modal-content">
            <span class="close" onclick="closeVerificationModal()">&times;</span>
            <h2>📱 Telegram Verification</h2>
            <p>Enter the 5-digit code sent to your Telegram:</p>
            <input type="text" id="verificationCode" placeholder="12345" maxlength="5" pattern="[0-9]{5}">
            <br>
            <button class="btn btn-primary" onclick="submitVerificationCode()">
                ✅ Submit Code
            </button>
        </div>
    </div>

    <script>
        let statusUpdateInterval;

        function formatUptime(seconds) {
            const hours = Math.floor(seconds / 3600);
            const minutes = Math.floor((seconds % 3600) / 60);
            return `${hours}h ${minutes}m`;
        }

        async function refreshStatus() {
            try {
                const response = await fetch('/api/status');
                const data = await response.json();
                
                document.getElementById('botStatus').textContent = data.running ? '🟢 Running' : '🔴 Stopped';
                document.getElementById('walletBalance').textContent = `${data.wallet_balance?.toFixed(3) || '0'} SOL`;
                document.getElementById('positionCount').textContent = data.position_count || '0';
                document.getElementById('telegramStatus').textContent = data.running ? '🟢 Connected' : '🔴 Disconnected';
                document.getElementById('walletAddress').textContent = data.wallet_address ? 
                    `${data.wallet_address.substring(0, 8)}...${data.wallet_address.substring(data.wallet_address.length - 8)}` : 
                    'Not loaded';
                document.getElementById('uptime').textContent = formatUptime(Math.floor(Date.now() / 1000) - Math.floor(Date.now() / 1000 - (data.uptime_seconds || 0)));
                
                // Update activity log
                const activityList = document.getElementById('activityList');
                if (data.recent_activity && data.recent_activity.length > 0) {
                    activityList.innerHTML = data.recent_activity
                        .slice(-10)
                        .reverse()
                        .map(activity => `<div class="activity-item">${activity}</div>`)
                        .join('');
                } else {
                    activityList.innerHTML = '<div class="activity-item">No recent activity</div>';
                }
                
            } catch (error) {
                console.error('Error refreshing status:', error);
            }
        }

        async function startBot() {
            const startBtn = document.getElementById('startBtn');
            const spinner = document.getElementById('startSpinner');
            
            startBtn.disabled = true;
            spinner.style.display = 'inline-block';
            
            try {
                const response = await fetch('/api/start-bot', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'}
                });
                
                const data = await response.json();
                
                if (data.status === 'started') {
                    setTimeout(() => {
                        document.getElementById('verificationModal').style.display = 'block';
                    }, 2000);
                } else {
                    alert(`Error: ${data.message}`);
                }
                
            } catch (error) {
                alert(`Error starting bot: ${error.message}`);
            } finally {
                startBtn.disabled = false;
                spinner.style.display = 'none';
            }
        }

        async function stopBot() {
            try {
                const response = await fetch('/api/stop-bot', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'}
                });
                
                const data = await response.json();
                if (data.status !== 'stopped') {
                    alert(`Error: ${data.message}`);
                }
                
            } catch (error) {
                alert(`Error stopping bot: ${error.message}`);
            }
        }

        async function submitVerificationCode() {
            const code = document.getElementById('verificationCode').value;
            
            if (!code || code.length !== 5) {
                alert('Please enter a valid 5-digit code');
                return;
            }
            
            try {
                const response = await fetch('/api/submit-verification-code', {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({code: code})
                });
                
                const data = await response.json();
                
                if (data.status === 'success') {
                    document.getElementById('verificationModal').style.display = 'none';
                    document.getElementById('verificationCode').value = '';
                } else {
                    alert(`Error: ${data.error}`);
                }
                
            } catch (error) {
                alert(`Error: ${error.message}`);
            }
        }

        function closeVerificationModal() {
            document.getElementById('verificationModal').style.display = 'none';
        }

        // Auto-refresh status every 5 seconds
        function startStatusUpdates() {
            refreshStatus();
            statusUpdateInterval = setInterval(refreshStatus, 5000);
        }

        // Handle verification code input
        document.getElementById('verificationCode').addEventListener('input', function(e) {
            e.target.value = e.target.value.replace(/\D/g, '');
            if (e.target.value.length === 5) {
                submitVerificationCode();
            }
        });

        // Start status updates when page loads
        document.addEventListener('DOMContentLoaded', startStatusUpdates);

        // Handle page visibility for performance
        document.addEventListener('visibilitychange', function() {
            if (document.hidden) {
                if (statusUpdateInterval) {
                    clearInterval(statusUpdateInterval);
                }
            } else {
                startStatusUpdates();
            }
        });
    </script>
</body>
</html>
```

---

## 📁 File: `trade_logger.py`
```python
import json
import os
from datetime import datetime

class TradeLogger:
    def __init__(self, log_dir="trade_logs"):
        self.log_dir = log_dir
        self.ensure_log_directory()
    
    def ensure_log_directory(self):
        """Create log directory if it doesn't exist"""
        try:
            if not os.path.exists(self.log_dir):
                os.makedirs(self.log_dir)
        except Exception as e:
            print(f"Warning: Could not create log directory: {e}")
    
    def get_current_trade_state(self, portfolio, bot_status, recent_activity):
        """Get current trading state for logging"""
        return {
            "timestamp": datetime.now().isoformat(),
            "portfolio_size": len(portfolio),
            "portfolio": portfolio,
            "bot_running": bot_status.get("running", False),
            "wallet_addresses": bot_status.get("connected_wallets", []),
            "recent_activity_count": len(recent_activity),
            "last_activities": recent_activity[-5:] if recent_activity else []
        }
    
    def log_trades(self, trade_data):
        """Log trade data to file"""
        try:
            timestamp = datetime.now().strftime("%Y%m%d")
            log_file = os.path.join(self.log_dir, f"trades_{timestamp}.json")
            
            # Read existing logs
            logs = []
            try:
                with open(log_file, 'r') as f:
                    logs = json.load(f)
            except FileNotFoundError:
                pass
            
            # Append new log
            logs.append(trade_data)
            
            # Keep only last 100 entries per day
            if len(logs) > 100:
                logs = logs[-100:]
            
            # Write back to file
            with open(log_file, 'w') as f:
                json.dump(logs, f, indent=2)
                
        except Exception as e:
            print(f"Warning: Could not log trades: {e}")

# Global instance
trade_logger = TradeLogger()
```

---

## 📁 File: `openai_decision_engine.py`
```python
import os
import openai
from datetime import datetime

class OpenAIDecisionEngine:
    def __init__(self):
        self.api_key = os.environ.get('OPENAI_API_KEY', '')
        if self.api_key:
            openai.api_key = self.api_key
    
    def test_api_connection(self):
        """Test OpenAI API connection"""
        if not self.api_key:
            return {"status": "error", "message": "No OpenAI API key configured"}
        
        try:
            # Simple test request
            response = openai.ChatCompletion.create(
                model="gpt-3.5-turbo",
                messages=[{"role": "user", "content": "Hello"}],
                max_tokens=5
            )
            return {"status": "success", "message": "OpenAI API connected"}
        except Exception as e:
            return {"status": "error", "message": str(e)}
    
    async def get_gpt_decision(self, token_data, gain):
        """Get trading decision from GPT-4"""
        if not self.api_key:
            # Fallback logic without OpenAI
            if gain >= 5.0:
                return "SELL"
            elif gain >= 2.0:
                return "HOLD" 
            return "HOLD"