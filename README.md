# Map Agent with MCP-Style Map Servers (Nominatim + OSRM)

This project is a small mapping agent built with the **OpenAI Agents SDK**. It exposes two MCP-inspired “map servers” as tools:

- A **Geocoding & POI server** backed by the public [Nominatim](https://nominatim.org/) geocoding service from OpenStreetMap. It can:
  - Geocode free-text addresses (e.g. “Times Square, New York City”) into coordinates.
  - Reverse-geocode coordinates back into human-readable addresses.
  - Search for points of interest (POIs), such as sushi restaurants in Tokyo.   
- A **Routing server** backed by the [OSRM HTTP API](http://project-osrm.org/). It can:
  - Compute driving routes between coordinates (distance in meters, time in seconds).   
  - Snap a point to the nearest road.
  - Build distance/time matrices between multiple locations using OSRM’s *table* service.   

These servers are wrapped as tools using the OpenAI Agents SDK (`Agent`, `Runner`, `@function_tool`) so an agent can answer natural-language questions like:

> “Find a driving route from Times Square to Central Park and tell me the distance and approximate driving time.”

The agent calls the tools, reads their JSON outputs (including precomputed durations in minutes), and returns grounded answers rather than guessing.   

---

## Project Structure

```text
.
├── agent_app.py              # OpenAI Agent + tool wiring
├── map_servers/
│   ├── __init__.py
│   ├── params.py             # ServerParams for Nominatim + OSRM
│   ├── geocoding_server.py   # Nominatim-based geocode / reverse / POI
│   └── routing_server.py     # OSRM-based route / nearest / table
├── tests/
│   ├── __init__.py
│   ├── test_parsing.py       # Unit tests for simplification logic
│   └── test_live_services.py # Optional live tests hitting public APIs
├── demo_notebook.ipynb       # Colab-style demo notebook (optional)
└── requirements.txt
