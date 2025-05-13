# Creating Custom Toolsets for Miyoka AI Agents

This guide explains how to create custom toolsets that extend the capabilities of Miyoka AI agents. Toolsets are collections of methods that provide specific functionality to agents, allowing them to interact with external systems, APIs, blockchains, and more.

## Table of Contents

- [Overview](#overview)
- [Toolset Structure](#toolset-structure)
- [Creating a New Toolset](#creating-a-new-toolset)
- [Best Practices](#best-practices)
- [Examples](#examples)
- [Testing Your Toolset](#testing-your-toolset)
- [Contributing](#contributing)

## Overview

Miyoka AI agents can be enhanced with custom toolsets that provide domain-specific capabilities. When an agent is created with a toolset, it can use the methods in that toolset as tools to accomplish tasks. The agent automatically discovers the methods in the toolset and makes them available for use.

Key benefits of custom toolsets:

- **Extend agent capabilities** with domain-specific functionality
- **Connect to external systems** like APIs, databases, or blockchains
- **Encapsulate complex logic** in reusable components
- **Maintain separation of concerns** between agent logic and tool implementation

## Toolset Structure

A toolset is a Python class that inherits from the `ToolSet` base class. Each public method in the toolset becomes a tool that the agent can use. The method's docstring becomes the tool's description, and its parameters become the tool's inputs.

### Basic Structure

```python
from toolsets.base import ToolSet

class MyCustomToolset(ToolSet):
    def __init__(self, param1, param2=None):
        """Initialize the toolset with required parameters."""
        self.param1 = param1
        self.param2 = param2
        
    async def my_tool_method(self, input1: str, input2: int = 0) -> dict:
        """
        Description of what this tool does.
        
        Args:
            input1 (str): Description of input1.
            input2 (int, optional): Description of input2. Defaults to 0.
            
        Returns:
            dict: Description of the return value.
        """
        # Tool implementation
        result = {"output": f"Processed {input1} with {input2}"}
        return result
```

### Key Components

1. **Class Definition**: Inherit from `ToolSet` base class
2. **Initialization**: Set up any resources needed by the toolset
3. **Methods**: Define public methods that will become tools
4. **Docstrings**: Provide clear descriptions for methods and parameters
5. **Type Hints**: Use Python type hints for parameters and return values
6. **Async Support**: Use `async` methods for non-blocking operations

## Creating a New Toolset

Follow these steps to create a new toolset:

1. Create a new Python file in the `toolsets` directory
2. Import the `ToolSet` base class
3. Define your toolset class inheriting from `ToolSet`
4. Implement the `__init__` method to initialize your toolset
5. Add methods that will become tools for the agent
6. Document your methods with clear docstrings
7. Use type hints for parameters and return values

### Example: Creating a Weather Toolset

```python
# toolsets/weather.py
import aiohttp
import logging
from typing import Dict, Any, Optional
from toolsets.base import ToolSet

logger = logging.getLogger(__name__)

class WeatherToolset(ToolSet):
    def __init__(self, api_key: str):
        """Initialize the weather toolset with an API key."""
        self.api_key = api_key
        self.base_url = "https://api.weatherapi.com/v1"
        
    async def get_current_weather(self, location: str) -> Dict[str, Any]:
        """
        Get the current weather for a location.
        
        Args:
            location (str): The location to get weather for (city name, zip code, etc.)
            
        Returns:
            Dict with current weather information.
        """
        try:
            url = f"{self.base_url}/current.json"
            params = {"key": self.api_key, "q": location}
            
            async with aiohttp.ClientSession() as session:
                async with session.get(url, params=params) as resp:
                    if resp.status != 200:
                        raise Exception(f"API error: {resp.status}")
                    data = await resp.json()
                    
                    return {
                        "location": data["location"]["name"],
                        "country": data["location"]["country"],
                        "temperature_c": data["current"]["temp_c"],
                        "temperature_f": data["current"]["temp_f"],
                        "condition": data["current"]["condition"]["text"],
                        "humidity": data["current"]["humidity"],
                        "wind_kph": data["current"]["wind_kph"]
                    }
        except Exception as e:
            logger.error(f"Failed to get weather: {str(e)}")
            raise Exception(f"Failed to get weather: {str(e)}")
    
    async def get_forecast(self, location: str, days: int = 3) -> Dict[str, Any]:
        """
        Get a weather forecast for a location.
        
        Args:
            location (str): The location to get forecast for.
            days (int, optional): Number of days for the forecast (1-10). Defaults to 3.
            
        Returns:
            Dict with forecast information.
        """
        try:
            url = f"{self.base_url}/forecast.json"
            params = {"key": self.api_key, "q": location, "days": days}
            
            async with aiohttp.ClientSession() as session:
                async with session.get(url, params=params) as resp:
                    if resp.status != 200:
                        raise Exception(f"API error: {resp.status}")
                    data = await resp.json()
                    
                    forecast_days = []
                    for day in data["forecast"]["forecastday"]:
                        forecast_days.append({
                            "date": day["date"],
                            "max_temp_c": day["day"]["maxtemp_c"],
                            "min_temp_c": day["day"]["mintemp_c"],
                            "condition": day["day"]["condition"]["text"],
                            "chance_of_rain": day["day"]["daily_chance_of_rain"]
                        })
                    
                    return {
                        "location": data["location"]["name"],
                        "country": data["location"]["country"],
                        "forecast": forecast_days
                    }
        except Exception as e:
            logger.error(f"Failed to get forecast: {str(e)}")
            raise Exception(f"Failed to get forecast: {str(e)}")
```

## Best Practices

Follow these best practices when creating toolsets:

### 1. Method Design

- **Use async methods** for non-blocking operations, especially for API calls
- **Keep methods focused** on a single responsibility
- **Return structured data** (dictionaries or objects) rather than strings
- **Handle errors gracefully** with try/except blocks
- **Log errors and important information** for debugging

### 2. Documentation

- **Write clear docstrings** that explain what the method does
- **Document all parameters** with types and descriptions
- **Document return values** with types and descriptions
- **Include examples** in docstrings when helpful

### 3. Type Hints

- **Use type hints** for all parameters and return values
- **Use appropriate types** from the `typing` module for complex types
- **Be specific with types** to help the agent understand the expected inputs

### 4. Error Handling

- **Catch and handle exceptions** appropriately
- **Provide meaningful error messages** that help diagnose issues
- **Log errors** with sufficient context for debugging
- **Raise exceptions** with clear messages when necessary

### 5. Resource Management

- **Initialize resources** in the `__init__` method
- **Clean up resources** when they're no longer needed
- **Use context managers** (`with` statements) for resource management
- **Avoid global state** that could cause conflicts between agents

## Examples

Here are examples of toolsets for different domains:

### Blockchain Toolset (Solana)

```python
# toolsets/solana_trading.py (simplified example)
from toolsets.base import ToolSet
import aiohttp
from typing import Dict, Any

class SolanaTrading(ToolSet):
    def __init__(self, wallet):
        self.wallet = wallet
        self.connection = AsyncClient("https://api.mainnet-beta.solana.com")
        
    async def get_token_price(self, input_mint_address: str, output_mint_address: str = None) -> dict:
        """
        Retrieves the current price of a given token.

        Args:
            input_mint_address (str): The mint address of the token.
            output_mint_address (str, optional): The mint address of the token to compare against.

        Returns:
            dict: JSON data containing the price information.
        """
        token_prices_url = "https://api.jup.ag/price/v2?ids=" + input_mint_address
        if output_mint_address:
            token_prices_url += "&vsToken=" + output_mint_address
        async with aiohttp.ClientSession() as session:
            async with session.get(token_prices_url) as resp:
                data = await resp.json()
                if "error" in data:
                    print(f"[ERROR] {data['error']}")
                    return []
            return data
```

### NFT Analytics Toolset

```python
# toolsets/nft_analytics.py (simplified example)
from toolsets.base import ToolSet
import aiohttp
from typing import Dict, Any

class NFTAnalytics(ToolSet):
    def __init__(self, user_id: str, agent_id: str):
        self.user_id = user_id
        self.agent_id = agent_id
        self.magic_eden_url = "https://api-mainnet.magiceden.dev/v2"
        
    async def get_floor_price(self, collection_symbol: str) -> Dict[str, Any]:
        """
        Fetch the floor price of an NFT collection on Solana via Magic Eden.

        Args:
            collection_symbol (str): Collection identifier (e.g., "degods").

        Returns:
            Dict with floor price in SOL or error.
        """
        try:
            url = f"{self.magic_eden_url}/collections/{collection_symbol}/stats"
            async with aiohttp.ClientSession() as session:
                async with session.get(url) as resp:
                    if resp.status != 200:
                        raise Exception(f"Magic Eden API error: {resp.status}")
                    data = await resp.json()
                    floor_price = data.get("floorPrice", 0) / 1_000_000_000  # Lamports to SOL
                    return {"floor_price": floor_price, "symbol": collection_symbol}
        except Exception as e:
            raise Exception(f"Failed to fetch floor price: {str(e)}")
```

## Testing Your Toolset

Before integrating your toolset with an agent, test it independently:

1. Create a simple script that instantiates your toolset
2. Call its methods with test inputs
3. Verify the outputs match your expectations
4. Test error handling by providing invalid inputs

Example test script:

```python
import asyncio
from toolsets.weather import WeatherToolset

async def test_weather_toolset():
    # Initialize the toolset
    weather = WeatherToolset(api_key="your_api_key")
    
    # Test get_current_weather
    try:
        result = await weather.get_current_weather("London")
        print("Current weather in London:")
        print(result)
    except Exception as e:
        print(f"Error getting current weather: {e}")
    
    # Test get_forecast
    try:
        result = await weather.get_forecast("New York", days=5)
        print("\nForecast for New York:")
        print(result)
    except Exception as e:
        print(f"Error getting forecast: {e}")

if __name__ == "__main__":
    asyncio.run(test_weather_toolset())
```

## Contributing

When contributing toolsets to the Miyoka AI platform:

1. Follow the best practices outlined in this guide
2. Ensure your toolset is well-documented
3. Include tests for your toolset
4. Submit a pull request with your toolset

Your toolset should be placed in the `toolsets` directory and follow the naming convention of `<domain>_<purpose>.py`.

---

By following this guide, you can create powerful toolsets that extend the capabilities of Miyoka AI agents, enabling them to perform a wide range of tasks across different domains.
