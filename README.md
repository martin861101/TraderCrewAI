# Agentic Trading System

## Overview

The Agentic Trading System is a multi-agent AI framework designed for intraday day trading in Forex and commodities markets, inspired by collaborative intelligence principles from the "Agentic Frameworks Handbook" by Martin Schoeman. It leverages specialized AI agents to analyze market data, sentiment, technical indicators, and macro events, providing trade recommendations with risk management. The system decomposes complex trading tasks into sub-tasks assigned to agents, using tools like MCP servers for data retrieval and execution.

Built with LangGraph for orchestration, CrewAI for agent collaboration, and Docker for deployment, the system integrates RAG for up-to-date information and persistent memory via databases. It's cost-effective for personal use, running on open-source components with free tiers (e.g., Alpaca paper trading).

Key features:
- **Multi-Agent Collaboration**: Agents work together for comprehensive analysis (e.g., breakout entries during London-NY overlaps as per the "Day Trading Handbook").
- **MCP Servers**: Modular access to external resources (e.g., Alpaca for trading, Redis for caching).
- **LLM Integration**: Uses models like Mixtral-8x22b for reasoning.
- **Scalability**: Stateful workflows with human-in-the-loop oversight.

This project draws from advanced frameworks like LangGraph, MCP, AutoGen, and CrewAI, as detailed in the "Advanced Agentic AI Frameworks Guide" and "Master Document".

## Folder Structure

```
agentic-trading-system/
├── src/
│   ├── agents/                  # Agent implementations
│   │   ├── macro_agent.py
│   │   ├── sentiment_agent.py
│   │   ├── technical_agent.py
│   │   ├── strategy_agent.py
│   │   ├── risk_agent.py
│   │   ├── visualizer_agent.py
│   ├── orchestrator/            # Orchestration logic
│   │   ├── orchestrator.py
│   ├── memory/                  # Memory and database integration
│   │   ├── db_config.py
│   ├── tools/                   # External tool integrations
│   │   ├── api_tools.py
│   ├── rag/                     # RAG implementation
│   │   ├── rag_engine.py
│   ├── utils/                   # Utility functions
│   │   ├── data_processing.py
│   │   ├── visualization.py
│   ├── main.py                  # Entry point
├── config/                      # Configuration files
│   ├── config.yaml
│   ├── docker-compose.yml
├── mcp-servers/                 # Cloned MCP servers
│   ├── postgres/
│   ├── redis/
│   ├── fetch/
│   ├── puppeteer/
│   ├── git/
│   ├── alpaca/
│   ├── time/
│   └── Dockerfile.mcp
├── requirements.txt             # Dependencies
├── Dockerfile                   # Docker configuration for app
├── README.md                    # This file
├── trading_system_flowchart.png # Generated flowchart
└── tests/                       # Unit tests
    ├── test_agents.py
    ├── test_orchestrator.py
```

## Dependencies

The system uses the following dependencies (listed in `requirements.txt`):

- `langchain==0.1.0`
- `crewai==0.165.0` (updated for compatibility)
- `openai==1.3.0`
- `requests==2.31.0`
- `pandas==2.0.3`
- `numpy==1.24.3`
- `matplotlib==3.7.1`
- `yfinance==0.2.31`
- `uv==0.1.0`
- `redis==5.0.1`
- `psycopg2-binary==2.9.9`
- `pydantic==2.8.2`

## Installation and Setup

1. **Clone the Repository** (if not already done):
   ```bash
   git clone <your-repo-url>
   cd agentic-trading-system
   ```

2. **Clone MCP Servers**:
   ```bash
   git clone https://github.com/modelcontextprotocol/servers mcp-servers
   cp -r mcp-servers/postgres mcp-servers/
   cp -r mcp-servers/redis mcp-servers/
   cp -r mcp-servers/fetch mcp-servers/
   cp -r mcp-servers/puppeteer mcp-servers/
   cp -r mcp-servers/git mcp-servers/
   cp -r mcp-servers/time mcp-servers/
   git clone https://github.com/alpacahq/alpaca-mcp-server mcp-servers/alpaca
   ```

3. **Update Configuration**:
   - Edit `config/config.yaml` with API keys (e.g., LLM provider, Alpaca).
   - Example:
     ```
     llm:
       provider: xai
       model: mixtral-8x22b
       api_key: your-api-key
     trading:
       alpaca_api_key: your-alpaca-api-key
     ```

4. **Build and Run with Docker**:
   ```bash
   docker-compose up --build
   ```

5. **Run Locally** (alternative):
   ```bash
   pip install -r requirements.txt
   python main.py
   ```

## Agents and How They Work

The system uses six specialized agents orchestrated by CrewAI and LangGraph. Each agent performs a specific role in the trading workflow, drawing from the "Day Trading Handbook" strategies (e.g., RSI, MACD, breakouts) and agentic principles from the "Agentic Frameworks Handbook" (e.g., decomposition into sub-tasks).

1. **MacroAgent**:
   - **Role**: Fetches economic events (e.g., NFP, CPI) using MCP Redis for caching and RAG for data retrieval.
   - **How It Works**: Queries macro calendars via APIs or MCP servers, analyzes impact on volatility (e.g., during London-NY overlap). Outputs: "High volatility expected at 08:30 UTC due to NFP."
   - **Integration**: Uses `time_series_db` for historical data.

2. **SentimentAgent**:
   - **Role**: Analyzes news and X sentiment using MCP Fetch for web data.
   - **How It Works**: Retrieves embeddings from vector DB, scores sentiment (e.g., positive USD on Fed news). Outputs: "Positive sentiment score: 0.75."
   - **Integration**: RAG for similarity searches on unstructured data.

3. **TechnicalAgent**:
   - **Role**: Applies indicators (RSI, MACD, ATR) using MCP Postgres for price data storage.
   - **How It Works**: Calculates metrics (e.g., RSI <30 for oversold), detects patterns like breakouts. Outputs: "RSI=28.50. Breakout above 150.00."
   - **Integration**: `time_series_db` for real-time price feeds via yfinance.

4. **StrategyAgent**:
   - **Role**: Combines inputs to propose trades (e.g., momentum entry on MACD cross).
   - **How It Works**: Aggregates macro, sentiment, and technical data for strategies like mean-reversion or news breaks. Outputs: "Long USD/JPY, stop at 149.50, target 151.00 (RR=2:1)."
   - **Integration**: No direct MCP, relies on agent collaboration.

5. **RiskAgent**:
   - **Role**: Manages risk (1-2% per trade) using MCP Alpaca for execution simulation.
   - **How It Works**: Calculates position size, stops based on ATR, checks correlations. Outputs: "Position size: 0.5 lots, 1% risk."
   - **Integration**: Integrates with Alpaca for paper trading.

6. **VisualizerAgent**:
   - **Role**: Generates charts using MCP Puppeteer for browser automation.
   - **How It Works**: Plots trends (e.g., candlestick with RSI) and saves images. Outputs: "Visualization: price_trend.png generated."
   - **Integration**: Uses `matplotlib` locally or Puppeteer for dynamic rendering.

### Workflow
1. User query enters `main.py`.
2. `TradingOrchestrator` routes to agents via CrewAI tasks.
3. Agents interact with MCP servers and databases for data.
4. Outputs aggregated into trade signals.
5. Human-in-the-loop for validation (e.g., news trades).

## Frontend Concept
The frontend is conceptual (not implemented) but could be a React dashboard integrated with the backend via Socket.IO for real-time streaming:
- **Components**:
  - **Dashboard**: Displays agent outputs, trade signals, and charts.
  - **AgentStream**: Real-time logs from agents (e.g., sentiment scores).
  - **MemoryPanel**: Visualizes persistent state (e.g., Redis cache).
  - **RouterGraph**: Shows agent orchestration flow.
- **UI**: Modern, dark theme with Tailwind CSS. Input field for queries, output panels for signals, and charts (e.g., candlestick via Chart.js).
- **Integration**: Backend streams data to frontend via WebSockets. Hypothetical URL: `http://localhost:5173`.
- **Example Look**: A split-screen layout with left sidebar for agents, center for charts, right for recommendations.

## Troubleshooting
- **Common Errors**: Ensure MCP servers have correct entry points. Update Python to 3.11 for compatibility.
- **Dependencies**: Run `pip install -r requirements.txt` locally.
- **Security**: Use environment variables for API keys.

## References
- "Agentic Frameworks: Collaborative Intelligence" by Martin Schoeman.
- "Day Trading Handbook (Forex & Commodities)".
- "Advanced Agentic AI Frameworks Guide" (grok.com).
- "AI Agents: The Ultimate Goal of Artificial Intelligence" (OpenAI resource).
- MCP Servers: https://github.com/modelcontextprotocol/servers.
- Alpaca MCP: https://github.com/alpacahq/alpaca-mcp-server.

For more details, see the generated flowchart: trading_system_flowchart.png.
