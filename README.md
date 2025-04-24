# Implementing MCP Servers (Windows)

This project demonstrates how to build a simple MCP server that provides weather forecasts and alerts using the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/). The server exposes two tools: `get-forecast` and `get-alerts`, which can be integrated with clients like [Claude for Desktop](https://modelcontextprotocol.io/quickstart/user).

---

### Find the Complete Documentation link here: https://modelcontextprotocol.io/quickstart/server#windows

## 🧰 Prerequisites

- **Operating System:** Windows
- **Python:** Version 3.10 or higher
- **MCP Python SDK:** Version 1.2.0 or higher
- **Familiarity with:**

  - Python programming
  - Large Language Models (LLMs) like Claude

---

## ⚙️ Setup Instructions

1.  **Install `uv` Package Manager:**

    Open PowerShell and run:

    powershell

    CopyEdit

    `curl -LsSf https://astral.sh/uv/install.sh | sh`

    After installation, restart your terminal to ensure the `uv` command is recognized.

2.  **Initialize the Project:**

    powershell

    CopyEdit

    `uv init weather cd weather uv venv .\.venv\Scripts\activate`

3.  **Install Dependencies:**

    powershell

    CopyEdit

    `uv add "mcp[cli]" httpx`

4.  **Create the Server File:**

    powershell

    CopyEdit

    `touch  weather.py` or `echo . > weather.py`

---

## 🛠️ Building the Server

1.  **Import Required Modules:**

    In `weather.py`, add:

    python

    CopyEdit

    `from typing import Any import httpx from mcp.server.fastmcp import FastMCP  mcp = FastMCP("weather")  NWS_API_BASE = "https://api.weather.gov" USER_AGENT = "weather-app/1.0"`

2.  **Define Helper Functions:**

    python

    CopyEdit

    `async def make_nws_request(url: str) -> dict[str, Any] | None:     headers = {         "User-Agent": USER_AGENT,         "Accept": "application/geo+json"     }     async with httpx.AsyncClient() as client:         try:             response = await client.get(url, headers=headers, timeout=30.0)             response.raise_for_status()             return response.json()         except Exception:             return None  def format_alert(feature: dict) -> str:     props = feature["properties"]     return f""" Event: {props.get('event', 'Unknown')} Area: {props.get('areaDesc', 'Unknown')} Severity: {props.get('severity', 'Unknown')} Description: {props.get('description', 'No description available.')} """`

3.  **Define MCP Tools:**

    python

    CopyEdit

    `@mcp.tool() async def get_forecast(latitude: float, longitude: float) -> str:     """Retrieve the weather forecast for a given location."""     url = f"{NWS_API_BASE}/points/{latitude},{longitude}/forecast"     data = await make_nws_request(url)     if not data:         return "Unable to fetch forecast data."     periods = data.get("properties", {}).get("periods", [])     return "\n\n".join(f"{p['name']}: {p['detailedForecast']}" for p in periods[:3])  @mcp.tool() async def get_alerts(latitude: float, longitude: float) -> str:     """Retrieve active weather alerts for a given location."""     url = f"{NWS_API_BASE}/alerts/active?point={latitude},{longitude}"     data = await make_nws_request(url)     if not data or not data.get("features"):         return "No active alerts."     return "\n\n".join(format_alert(f) for f in data["features"])`

4.  **Run the Server:**

    python

    CopyEdit

    `if __name__ == "__main__":     mcp.run()`

    Then, start the server:

    powershell

    CopyEdit

    `python weather.py`

---

## 🧩 Integrating with Claude for Desktop

1.  **Locate Configuration File:**

    The configuration file is typically located at:

    plaintext

    CopyEdit

    `%APPDATA%\Claude\claude_desktop_config.json`

2.  **Edit Configuration:**

    Add your server to the `mcpServers` key:

    json

    CopyEdit

    `{   "mcpServers": [     {       "name": "weather",       "command": "python",       "args": ["C:\\path\\to\\weather.py"]     }   ] }`

    Replace `C:\\path\\to\\weather.py` with the actual path to your `weather.py` file.

3.  **Restart Claude for Desktop:**

    After updating the configuration, restart Claude for Desktop. You should see the new tools available.

---

## 🧪 Testing and Debugging

- **MCP Inspector:** Use the MCP Inspector for interactive testing and debugging of your server.

  bash

  CopyEdit

  `npx @modelcontextprotocol/inspector`

- **Logging:** Implement logging within your server to monitor requests and responses.

---

## 📚 Resources

- [MCP Server Quickstart Guide](https://modelcontextprotocol.io/quickstart/server#windows)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Claude for Desktop Setup](https://modelcontextprotocol.io/quickstart/user)

---

## Screenshot of output

![screenshot_output1](assets/screenshot1.png)

## What’s happening under the hood

When you ask a question:

1.  The client sends your question to Claude
2.  Claude analyzes the available tools and decides which one(s) to use
3.  The client executes the chosen tool(s) through the MCP server
4.  The results are sent back to Claude
5.  Claude formulates a natural language response
6.  The response is displayed to you!

## 📝 License

This project is licensed under the MIT License.

---

Feel free to customize this `README.md` further to suit your project's specific needs or to add more detailed instructions as necessary.
