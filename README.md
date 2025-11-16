# Map Agent Demo – OpenAI Agents SDK + Open Map Servers

This project is a small end-to-end example of a **tool-using agent** for map tasks.

It combines:

- A **geocoding + POI “map server”** built on top of the public Nominatim API from OpenStreetMap (for address lookup, reverse geocoding, and place search).   
- A **routing “map server”** built on top of the OSRM HTTP API (for routes, nearest road snapping, and distance matrices).   
- An **OpenAI Agent** wired to these servers as tools using the official Agents SDK.   

The agent can answer questions like:

- “How far is it from Times Square to Central Park, and how long does it take to drive?”
- “Find sushi restaurants in Tokyo and give me their coordinates.”
- “Between Big Ben, Buckingham Palace, and Tower Bridge, which pair is closest in travel time?”

---

## Features

### Geocoding & POI Map Server (Nominatim-based)

Backed by the public Nominatim API, which provides open geocoding on OpenStreetMap data.   

Exposed operations (via tools):

- **Forward geocoding** – Convert a free-text address or place name to coordinates.
- **Reverse geocoding** – Convert coordinates to a human-readable address.
- **POI search** – Search for points of interest (e.g., “coffee shop”, “sushi restaurant”) optionally constrained by city.

Internally this wraps Nominatim’s `/search` and `/reverse` endpoints and normalizes the responses into small, stable dictionaries.

> **Important:** The public Nominatim service restricts usage to *no more than 1 request per second* and requires a meaningful `User-Agent` that identifies your application.   

---

### Routing Map Server (OSRM-based)

Backed by the OSRM HTTP API, which uses OpenStreetMap data for routing.   

Exposed operations (via tools):

- **Route** – Fastest route between two coordinates (distance in meters, travel time in seconds and minutes).
- **Nearest** – Snap a coordinate to the nearest road segment.
- **Distance matrix** – Travel time and distance between multiple coordinates (OSRM `/table` service).   

The project converts OSRM durations from seconds to minutes in the tools themselves so the agent can reuse those values without having to do the conversion.

> **Note:** The public OSRM demo server is intended for testing and small experiments only; for production use you should host your own OSRM backend or use a third-party provider.   

---

### Agent (OpenAI Agents SDK)

The agent is built using the OpenAI Agents SDK for Python.   

Key points:

- **Tools:** Each map operation (geocode, reverse, POI, route, nearest, matrix) is wrapped as a `@function_tool`.  
- **Instructions:** The agent is told to:
  - Use tools whenever it needs real geographic data.
  - Prefer precomputed minute values (`duration_min`, `durations_min`) instead of converting seconds to minutes itself.
- **Runner:** A small async helper `run_map_agent_async()` is provided so you can call the agent from scripts or notebooks without event-loop issues.

---

## Project Structure

```text
.
├── agent_app.py                # Agent definition and tool wiring
├── map_servers/
│   ├── __init__.py
│   ├── params.py               # ServerParams for geocoding & routing servers
│   ├── geocoding_server.py     # Nominatim-based map server
│   └── routing_server.py       # OSRM-based map server
├── tests/
│   ├── __init__.py
│   ├── test_parsing.py         # Unit tests for parsing/simplification logic
│   └── test_live_services.py   # Optional live tests hitting Nominatim/OSRM
├── requirements.txt
└── README.md                   # This file
