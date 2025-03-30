# ✨ Perun Trading System ✨

Welcome to **Perun**, an automated trading system designed to leverage the power of Large Language Models (LLMs) for market analysis and trade execution. Perun analyzes market data, generates trading signals, manages a portfolio, and interacts with brokerage APIs, all orchestrated within a modular and configurable framework. For a deeper dive into the system's concepts and workflow, see the [🌌 Conceptual Overview](./docs/system_overview.md).

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) <!-- Example Badge -->

---

## 🚀 Features

*   🧠 **LLM-Powered Analysis:** Utilizes LLMs (OpenAI, Gemini) for deep market analysis and signal generation.
*   🔍 **Market Research:** Integrates with Perplexity AI for real-time news and sentiment analysis.
*   🤖 **Automated Trading Cycle:** Full automation from data fetching to order execution and portfolio monitoring.
*   🏦 **Brokerage Integration:** Connects seamlessly with Alpaca for market data, account management, and trading.
*   💾 **Persistent Memory:** Maintains a history of actions, observations, and insights to inform future decisions.
*   📢 **Notification System:** Configurable alerts via Mattermost and Email.
*   ⚙️ **Configuration Driven:** Easily customize system behavior via environment variables.
*   🧱 **Modular Architecture:** Decoupled services for enhanced maintainability, testability, and extensibility.
*   📈 **Optimization Ready:** Includes components for performance analysis and potential parameter tuning.

---

## 🏗️ Architecture Overview

Perun employs a service-oriented architecture, coordinated by a central daemon:

*   **Orchestration Service (`OrchestrationDaemon`):** 🕰️ The main control loop, scheduling tasks based on market hours and system state. [More Details](./docs/orchestration_service.md)
*   **AI Service (`AIServiceProcessor`):** 🤖 Interacts with LLMs (OpenAI/Gemini) and Perplexity to analyze data and generate trading signals. [More Details](./docs/ai_service.md)
*   **Execution Service (`ExecutionManager`):** 💼 Manages all interactions with the brokerage (Alpaca), handling orders and portfolio updates. [More Details](./docs/execution_service.md)
*   **Memory Service (`MemoryStorage`, `MemoryOrganizer`):** 📚 Stores and retrieves system memory (trades, signals, logs, analysis). [More Details](./docs/memory_service.md)
*   **Optimization Service (`OptimizationEngine`, `FrequencyAnalyzer`):** 🛠️ Analyzes performance and suggests parameter adjustments. [More Details](./docs/optimization_service.md)
*   **Interfaces:** 🔌 Abstract layers for external communication:
    *   `BrokerageInterface`: Alpaca interactions. [Details](./docs/brokerage_interface.md)
    *   `LLMInterface`: OpenAI/Gemini interactions. [Details](./docs/llm_interface.md)
    *   `PerplexityInterface`: Perplexity AI interactions. (See `src/interfaces/perplexity.py`)
    *   `NotificationInterface`: Mattermost/Email interactions. [Details](./docs/notification_interface.md)
    *   `WebDataInterface`: (Future) Fetching external web data. [Details](./docs/web_data_interface.md)
*   **Models:** 🧱 Core data structures (`Order`, `Signal`, `Portfolio`, etc.).

[General Interface Concepts](./docs/interfaces.md)

---

## 🛠️ Setup & Configuration

Follow these steps to get Perun up and running:

**1. Clone the Repository:**
```bash
git clone https://github.com/david-strejc/perun.git
cd perun # Note: The repo was created as 'perun', containing the 'trading_system' files directly
```

**2. Create & Activate Virtual Environment:**
```bash
# Linux/macOS
python3 -m venv .venv
source .venv/bin/activate

# Windows (Git Bash/WSL)
python3 -m venv .venv
source .venv/Scripts/activate

# Windows (Command Prompt)
python -m venv .venv
.venv\Scripts\activate.bat

# Windows (PowerShell)
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**3. Install Dependencies:**
```bash
pip install -r requirements.txt
```

**4. Configure Environment Variables (`.env`):**

Create a `.env` file in the project root (`perun/`). This file stores your API keys and configuration settings. **Do not commit this file to Git.**

```dotenv
#####################################################
# Perun Trading System Environment Configuration    #
#####################################################

# --- Brokerage: Alpaca (Required) ---
# 🔑 Get keys from: https://app.alpaca.markets/paper/dashboard/overview (Paper) or https://app.alpaca.markets/live/dashboard/overview (Live)
ALPACA_API_KEY=YOUR_ALPACA_KEY_ID
ALPACA_SECRET_KEY=YOUR_ALPACA_SECRET_KEY
ALPACA_BASE_URL=https://paper-api.alpaca.markets # Use paper trading URL for testing
# ALPACA_BASE_URL=https://api.alpaca.markets # Uncomment for live trading

# --- LLM & Research APIs (Optional Keys, Required Models) ---
# 🔑 OpenAI: https://platform.openai.com/api-keys (Needed if using OpenAI models)
OPENAI_API_KEY=YOUR_OPENAI_KEY_IF_USING_OPENAI

# 🔑 Google Gemini: https://aistudio.google.com/app/apikey (Needed if using Gemini models)
GEMINI_API_KEY=YOUR_GOOGLE_KEY_IF_USING_GEMINI

# 🔑 Perplexity AI: https://docs.perplexity.ai/docs/getting-started (Needed for market research feature)
PERPLEXITY_API_KEY=YOUR_PERPLEXITY_API_KEY

# Specify the models for each task (MUST BE SET - Choose models accessible with your keys)
# Example models: "gpt-4o", "gpt-3.5-turbo", "gemini-1.5-flash", "gemini-pro"
TRADING_ANALYSIS_LLM_MODEL="gpt-4o"
MEMORY_ORGANIZATION_LLM_MODEL="gpt-3.5-turbo" # Can use a cheaper/faster model
OPTIMIZATION_LLM_MODEL="gpt-4o"

# --- Notifications (Optional) ---
# Mattermost (Set MATTERMOST_ENABLED=true to enable)
# 🔑 Create a Bot Account: System Console -> Integrations -> Bot Accounts
MATTERMOST_ENABLED=false
MATTERMOST_URL=https://your.mattermost.instance.com # Your Mattermost server URL
MATTERMOST_TOKEN=YOUR_MATTERMOST_BOT_TOKEN
MATTERMOST_TEAM_ID=YOUR_MATTERMOST_TEAM_ID # Find in URL or via API
MATTERMOST_CHANNEL_ID=YOUR_TARGET_CHANNEL_ID # Find in URL or via API

# Email (Set EMAIL_ENABLED=true to enable)
# 🔑 Use your email provider's SMTP details. For Gmail, you might need an "App Password".
EMAIL_ENABLED=false
SMTP_SERVER=smtp.example.com # e.g., smtp.gmail.com
SMTP_PORT=587 # Common ports: 587 (TLS), 465 (SSL)
SMTP_USERNAME=your_email@example.com
SMTP_PASSWORD=your_email_or_app_password
ADMIN_EMAIL=recipient_email@example.com # Email address to send notifications TO

# --- File Paths (Required - Relative to project root) ---
MEMDIR_PATH=data/memdir
LOG_PATH=data/logs
PROMPTS_PATH=prompts

# --- Trading Parameters (Required) ---
DEFAULT_SYMBOLS=AAPL,MSFT,GOOG # Comma-separated list of symbols to trade
MAX_POSITION_SIZE=10000 # Maximum value (USD) per position
MAX_TOTAL_POSITIONS=5 # Maximum number of concurrent open positions
RISK_LIMIT_PERCENT=0.02 # Max risk per trade as % of portfolio equity (e.g., 0.02 = 2%)

# --- Logging Configuration (Optional - Defaults provided) ---
LOG_LEVEL_CONSOLE=INFO # DEBUG, INFO, WARNING, ERROR, CRITICAL
LOG_LEVEL_FILE=DEBUG
LOG_FILE_NAME=trading_system.log # Log filename within LOG_PATH

# --- Optimization Parameters (Required if OPTIMIZATION_ENABLED=true) ---
OPTIMIZATION_ENABLED=true # Set to false to disable optimization runs
OPTIMIZATION_SCHEDULE=daily # How often to run optimization (e.g., 'daily', 'weekly', or cron: '0 3 * * 0' for 3 AM Sunday)
OPTIMIZATION_PROMPT_THRESHOLD=0.05 # Min performance improvement (e.g., 5%) to accept a new prompt
OPTIMIZATION_MIN_FREQUENCY=60 # Minimum trading frequency (seconds) allowed by optimization
OPTIMIZATION_FREQUENCY_BUFFER_FACTOR=1.5 # Safety buffer for frequency calculation
OPTIMIZATION_MEMORY_QUERY_DAYS=30 # How many days of history to query for optimization analysis

# --- Memory Service Configuration (Required) ---
MEMDIR_PRUNE_MAX_AGE_DAYS=90 # Delete memory files older than this (0 to disable age pruning)
MEMDIR_PRUNE_MAX_COUNT=100000 # Max memory files to keep (0 to disable count pruning)
MEMDIR_ORGANIZER_MODEL=sentence-transformers/all-MiniLM-L6-v2 # Model for memory tagging/similarity

# --- Orchestration Service (Required) ---
MAIN_LOOP_SLEEP_INTERVAL=1 # Sleep interval (seconds) when idle (e.g., outside market hours)
LIQUIDATE_ON_CLOSE=false # Set to true to liquidate all positions before market close
```

---

## ▶️ Usage

Ensure your virtual environment is active and the `.env` file is configured correctly. Run the main orchestration daemon from the project root (`perun/`):

```bash
python main.py
```

The system will initialize all services and begin its operational cycle. Monitor the console output and log files (`data/logs/trading_system.log`) for status updates and potential issues.

---

## 📁 Project Structure

```
perun/
├── .env                # Environment variables (sensitive, DO NOT COMMIT)
├── .git/               # Git repository data
├── .gitignore          # Files ignored by Git
├── .venv/              # Python virtual environment (ignored)
├── data/               # Data storage (logs, memory - ignored by default)
│   ├── logs/           # Log files (.gitkeep to keep dir)
│   └── memdir/         # Persistent memory storage (.gitkeep to keep dir)
├── docs/               # Detailed documentation for concepts/components
│   ├── ai_service.md
│   ├── brokerage_interface.md
│   ├── execution_service.md
│   ├── interfaces.md
│   ├── llm_interface.md
│   ├── memory_service.md
│   ├── notification_interface.md
│   ├── optimization_service.md
│   ├── orchestration_service.md
│   └── web_data_interface.md
├── prompts/            # LLM prompts
│   ├── evaluation/
│   ├── memory_organization/
│   ├── trading/
│   └── metadata.json
├── scripts/            # Utility scripts
│   └── check_market_hours.py
├── src/                # Source code
│   ├── __init__.py
│   ├── config.py       # Configuration loading
│   ├── interfaces/     # External service interfaces
│   ├── models/         # Data models (Pydantic)
│   ├── services/       # Core logic services
│   └── utils/          # Utility functions (Logging, Exceptions)
├── tests/              # Unit and integration tests
│   ├── __init__.py
│   ├── interfaces/
│   ├── services/
│   └── utils/
├── main.py             # Main application entry point
├── README.md           # This file
├── requirements.txt    # Python dependencies
└── repomix-output.txt  # (Optional) Output from code analysis tools
```

---

## 🧪 Development & Testing

*   Make sure development dependencies are installed (`pip install -r requirements.txt`).
*   Run tests using `pytest` from the project root (`perun/`):
    ```bash
    pytest
    ```
*   Consider using pre-commit hooks for code formatting and linting.

---

## 🤝 Contributing

Contributions are welcome! Please follow standard fork-and-pull-request workflows. Ensure tests pass and code adheres to project standards. (Further details can be added).

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details (assuming MIT, add a LICENSE file if needed).
