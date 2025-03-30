# Perun Trading System

This project implements an automated trading system, named Perun, that leverages Large Language Models (LLMs) to analyze market data, generate trading signals, manage a portfolio, and execute trades via a brokerage API. It features a modular architecture with distinct services for orchestration, AI processing, execution, memory management, and optimization.

## Features

*   **LLM-Powered Analysis:** Utilizes LLMs (like OpenAI's GPT or Google's Gemini) to interpret market news, data, and potentially generate trading insights or signals.
*   **Automated Trading Cycle:** Orchestrates the entire process from data fetching, analysis, signal generation, order placement, and portfolio monitoring.
*   **Brokerage Integration:** Connects to brokerage platforms (e.g., Alpaca) to fetch market data, get account information, and execute trades.
*   **Memory System:** Maintains a persistent memory of past actions, market observations, and generated insights to inform future decisions.
*   **Notification System:** Sends updates and alerts via configured channels (e.g., Mattermost).
*   **Configuration Driven:** System behavior is controlled via environment variables.
*   **Modular Services:** Built with distinct, decoupled services for better maintainability and testability.
*   **Optimization Capabilities:** Includes components for analyzing system performance and potentially optimizing parameters (e.g., trading frequency).

## Architecture Overview

The system is composed of several core services orchestrated by a central daemon:

1.  **Orchestration Service (`OrchestrationDaemon`):** The main control loop. It schedules and coordinates the tasks performed by other services based on market hours and system state.
2.  **AI Service (`AIProcessor`):** Interacts with the configured LLM. It processes market data, news, and memory context to generate trading signals or analysis.
3.  **Execution Service (`ExecutionManager`):** Manages interactions with the brokerage API. It places orders based on signals from the AI Service, monitors order statuses, and updates portfolio information.
4.  **Memory Service (`MemoryStorage`, `MemoryOrganizer`):** Stores and retrieves relevant information, such as historical market data, past trades, LLM insights, and system logs. The Organizer helps structure and query this memory effectively.
5.  **Optimization Service (`OptimizationEngine`, `FrequencyAnalyzer`):** Analyzes system performance and potentially adjusts parameters like trading frequency or strategy parameters based on historical data and results.
6.  **Interfaces:** Abstract layers for interacting with external systems:
    *   `BrokerageInterface`: Defines interactions with the trading platform.
    *   `LLMInterface`: Defines interactions with the chosen Large Language Model.
    *   `NotificationInterface`: Defines how notifications are sent.
    *   `WebDataInterface`: (Implied) Defines how external web data/news might be fetched.
7.  **Models:** Data structures used throughout the system (e.g., `Order`, `Signal`, `Portfolio`, `MarketData`, `MemoryEntry`).

## Setup

1.  **Clone the repository:**
    ```bash
    git clone <your-repo-url>
    cd trading_system
    ```

2.  **Create and activate a virtual environment:**
    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```
    *(On Windows, use `.venv\Scripts\activate`)*

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure Environment Variables:**
    Create a `.env` file in the `trading_system` root directory by copying the example or creating it from scratch. Fill in the required API keys and configuration settings.

    **Example `.env` structure (Check `src/config.py` for all required variables):**
    ```dotenv
    # --- Brokerage Configuration (Required) ---
    ALPACA_API_KEY=YOUR_ALPACA_KEY_ID
    ALPACA_SECRET_KEY=YOUR_ALPACA_SECRET_KEY
    ALPACA_BASE_URL=https://paper-api.alpaca.markets # Use paper trading URL for testing or https://api.alpaca.markets for live

    # --- LLM Configuration (Required: Task-Specific Models, Optional: API Keys) ---
    OPENAI_API_KEY=YOUR_OPENAI_KEY_IF_USING_OPENAI # Needed if any model is from OpenAI
    GEMINI_API_KEY=YOUR_GOOGLE_KEY_IF_USING_GEMINI   # Needed if any model is from Google

    # Specify the models for each task (MUST BE SET)
    TRADING_ANALYSIS_LLM_MODEL="gpt-4o"
    MEMORY_ORGANIZATION_LLM_MODEL="gpt-4o" # Or a cheaper model like gpt-3.5-turbo
    OPTIMIZATION_LLM_MODEL="gpt-4o"

    # --- Notification Configuration (Optional, but required if enabled) ---
    # Mattermost (Set MATTERMOST_ENABLED=true to enable)
    MATTERMOST_ENABLED=false
    MATTERMOST_URL=https://your.mattermost.instance.com
    MATTERMOST_TOKEN=YOUR_MATTERMOST_BOT_TOKEN
    MATTERMOST_TEAM_ID=YOUR_MATTERMOST_TEAM_ID
    MATTERMOST_CHANNEL_ID=YOUR_TARGET_CHANNEL_ID

    # Email (Set EMAIL_ENABLED=true to enable)
    EMAIL_ENABLED=false
    SMTP_SERVER=your_smtp_server
    SMTP_PORT=587 # Default, change if needed
    SMTP_USERNAME=your_email@example.com
    SMTP_PASSWORD=your_email_password
    ADMIN_EMAIL=admin@example.com # Email address to send notifications to

    # --- File Paths (Required) ---
    MEMDIR_PATH=data/memdir
    LOG_PATH=data/logs
    PROMPTS_PATH=prompts

    # --- Trading Parameters (Required) ---
    DEFAULT_SYMBOLS=AAPL,MSFT,GOOG # Comma-separated list
    MAX_POSITION_SIZE=10000 # Maximum value per position in USD
    MAX_TOTAL_POSITIONS=5 # Maximum number of concurrent positions
    RISK_LIMIT_PERCENT=0.02 # Max risk per trade as a percentage of portfolio equity

    # --- Logging Configuration (Optional - Defaults provided in code) ---
    LOG_LEVEL_CONSOLE=INFO # DEBUG, INFO, WARNING, ERROR, CRITICAL
    LOG_LEVEL_FILE=DEBUG
    LOG_FILE_NAME=trading_system.log

    # --- Optimization Parameters (Required if OPTIMIZATION_ENABLED=true) ---
    OPTIMIZATION_ENABLED=true # Default, set to false to disable
    OPTIMIZATION_SCHEDULE=daily # e.g., daily, weekly, or cron expression
    OPTIMIZATION_PROMPT_THRESHOLD=0.05 # Minimum performance improvement (e.g., 5%) to accept a new prompt
    OPTIMIZATION_MIN_FREQUENCY=60 # Minimum trading frequency in seconds (e.g., 1 minute)
    OPTIMIZATION_FREQUENCY_BUFFER_FACTOR=1.5 # Safety buffer for frequency calculation
    OPTIMIZATION_MEMORY_QUERY_DAYS=30 # How many days of history to query for optimization

    # --- Memory Service Configuration (Required) ---
    MEMDIR_PRUNE_MAX_AGE_DAYS=90 # Delete memory files older than this
    MEMDIR_PRUNE_MAX_COUNT=100000 # Keep only the latest N memory files if pruning by count
    MEMDIR_ORGANIZER_MODEL=sentence-transformers/all-MiniLM-L6-v2 # Example lightweight model for tagging

    # --- Orchestration Service (Required) ---
    MAIN_LOOP_SLEEP_INTERVAL=1 # Default sleep interval in seconds when idle
    LIQUIDATE_ON_CLOSE=false # Default, set to true to liquidate EOD
    ```

## Usage

Run the main orchestration daemon from the `trading_system` directory:

```bash
python main.py
```

The system will start, initialize the services, and begin its trading cycle based on the configured frequency and market hours. Logs will be output to the console and potentially to files in the `data/logs` directory (if configured).

## Directory Structure

```
trading_system/
├── .env                # Environment variables (sensitive, DO NOT COMMIT)
├── .gitignore          # Files ignored by Git
├── main.py             # Main entry point for the application
├── README.md           # This file
├── requirements.txt    # Python dependencies
├── data/               # Data storage (logs, memory)
│   ├── logs/           # Log files
│   └── memdir/         # Persistent memory storage
├── docs/               # Detailed documentation for concepts/components
├── prompts/            # LLM prompts
├── scripts/            # Utility scripts (e.g., check_market_hours.py)
├── src/                # Source code
│   ├── __init__.py
│   ├── config.py       # Configuration loading logic
│   ├── interfaces/     # Interfaces to external services (Brokerage, LLM, etc.)
│   ├── models/         # Data models (Order, Signal, Portfolio, etc.)
│   ├── services/       # Core logic services (AI, Execution, Memory, etc.)
│   └── utils/          # Utility functions (Logging, Exceptions, Constants)
└── tests/              # Unit and integration tests
    ├── __init__.py
    ├── interfaces/
    ├── services/
    └── utils/
```

## Development & Testing

*   Ensure test dependencies are installed (`pip install -r requirements.txt` includes them).
*   Run tests using pytest from the `trading_system` directory:
    ```bash
    pytest
    ```

## Contributing

(Add contribution guidelines if applicable)

## License

(Add license information if applicable)
