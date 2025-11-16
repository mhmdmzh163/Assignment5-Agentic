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
```
## Configuration

1. OpenAI API Key
   Set your OpenAI API key so the agent can run:
   ```bash
   export OPENAI_API_KEY="sk-..."
   ```
2. Nominatim (Geocoding & POI server)
   By default, the project uses the public Nominatim instance at:
   ```text
   https://nominatim.openstreetmap.org
   ```
Nominatim’s usage policy requires that clients:
     - Use a custom User-Agent that clearly identifies your application (not the default from your HTTP library).
     - Keep requests to a low rate (roughly 1 request per second or less) on the public instance.
     - Provide proper attribution to OpenStreetMap where results are used.
In this project, these details are configured via GeocodingServerParams in map_servers/params.py:
     - base_url – the Nominatim endpoint.
     - user_agent – the custom User-Agent string.
     - timeout – HTTP timeout for requests.
If you want to use this beyond a small demo, consider:
     - Reducing request frequency and adding caching.
     - Running your own Nominatim instance for higher-volume workloads.
3. OSRM (Routing server)
The routing server uses an OSRM HTTP endpoint, also configured in map_servers/params.py via RoutingServerParams:
     - base_url – the OSRM server URL (for example, a public demo server or your own instance).
     - profile – routing profile (e.g. "driving").
     - timeout – HTTP timeout for OSRM requests.
According to the OSRM HTTP API documentation:
     - distance values are returned in meters.
     - duration values are returned in seconds.
In this project, routing_server.py:
     - Keeps OSRM’s raw units (meters and seconds).
     - Adds convenience fields: duration_min for single routes and durations_min for the distance matrix.
You can point RoutingServerParams.base_url to:
     - A public OSRM demo server (suitable only for very light testing), or
     - Your own OSRM instance (recommended if you need reliability, custom data, or higher request volume). 
