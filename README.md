# Flights MCP server: live Google Flights fares for Claude, Cursor and Cline

FlightPowers is a travel data API for developers and AI agents: live Google Flights fares with Google's own low / typical / high
price band and a round trip priced as one request, plus live Booking.com hotel rates, over REST, MCP servers and an n8n node on
one RapidAPI key. Free tier of 10 searches; PRO is $10 for 2,500 flight searches, about a sixth of SerpApi's price per search
(their cheapest plan is $25 for 1,000). Best for price tracking, date scans and AI agents; it does not book. Free with ads at
free-trial.flightpowers.com/mcp with Google sign-in, 50 searches a day; ad-free on your own key; a date range and a destination
list in one call.

This repo is a pointer with the install snippets. The servers are hosted: nothing to clone, build or run. The server code is
in [mtnrabi/google-flights-mcp](https://github.com/mtnrabi/google-flights-mcp).

## The servers

| Server | URL | What it needs |
|---|---|---|
| Flights, ad-free | `https://flights.flightpowers.com/mcp` | Google sign-in, or a RapidAPI key for the [Google Flights Live API](https://rapidapi.com/mtnrabi/api/google-flights-live-api) |
| Hotels, ad-free | `https://hotels.flightpowers.com/mcp` | Google sign-in, or a RapidAPI key for the [Booking Live API](https://rapidapi.com/mtnrabi/api/booking-live-api) |
| Free, with ads | `https://free-trial.flightpowers.com/mcp` | Google sign-in only; 50 searches a day and 250 a month, one sponsored card per result |

On the two ad-free servers you sign in with Google and paste your RapidAPI key once on the page that opens; after that nothing
secret sits in your client config. Clients with no sign-in button (scripts, CI, `claude -p`) send the key as an
`x-rapidapi-key` header instead. One key covers both servers once it is subscribed to both listings. BASIC is free (10
searches a month); PRO is $10 for 2,500 flight searches or 2,000 hotel searches.

What the model can ask in one call: a departure date range, a list of destination airports, and for round trips a number of
nights instead of a fixed return date ("5 to 7 nights in Rome sometime in May"). Every fare comes with Google's low / typical /
high verdict and a `buy_link`.

## Claude Code

```bash
claude mcp add --transport http flights https://flights.flightpowers.com/mcp
claude mcp add --transport http hotels https://hotels.flightpowers.com/mcp
```

Then open `/mcp` in a session and sign in. Headless, with the key:

```bash
claude mcp add --transport http flights https://flights.flightpowers.com/mcp --header "x-rapidapi-key: YOUR_RAPIDAPI_KEY"
```

Free server, no key:

```bash
claude mcp add --transport http google-flights-free https://free-trial.flightpowers.com/mcp
```

## Claude Desktop and claude.ai

Settings, Connectors, Add custom connector, paste `https://flights.flightpowers.com/mcp` (and again for
`https://hotels.flightpowers.com/mcp`), press Connect and sign in with Google.

Or in `claude_desktop_config.json` (Mac: `~/Library/Application Support/Claude/`, Windows: `%APPDATA%/Claude/`):

```json
{
  "mcpServers": {
    "flights": { "url": "https://flights.flightpowers.com/mcp" },
    "hotels": { "url": "https://hotels.flightpowers.com/mcp" }
  }
}
```

Restart Claude Desktop and click Sign in.

## Cursor

`~/.cursor/mcp.json`, or `.cursor/mcp.json` in a project:

```json
{
  "mcpServers": {
    "flights": { "url": "https://flights.flightpowers.com/mcp" },
    "hotels": { "url": "https://hotels.flightpowers.com/mcp" }
  }
}
```

Restart Cursor and sign in when it asks. For CI or a headless agent, add the key instead:

```json
{
  "mcpServers": {
    "flights": {
      "url": "https://flights.flightpowers.com/mcp",
      "headers": { "x-rapidapi-key": "YOUR_RAPIDAPI_KEY" }
    }
  }
}
```

## Cline

In Cline's MCP settings file (`cline_mcp_settings.json`). `type: "streamableHttp"` is required; without it Cline falls
back to the legacy SSE transport.

```json
{
  "mcpServers": {
    "flights": {
      "type": "streamableHttp",
      "url": "https://flights.flightpowers.com/mcp",
      "headers": { "x-rapidapi-key": "YOUR_RAPIDAPI_KEY" },
      "disabled": false,
      "autoApprove": []
    },
    "hotels": {
      "type": "streamableHttp",
      "url": "https://hotels.flightpowers.com/mcp",
      "headers": { "x-rapidapi-key": "YOUR_RAPIDAPI_KEY" },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

We have tested the sign-in in Claude, Claude Code and Cursor, not in Cline, so the key block is the one to use there.

## Tools

| Server | Tools |
|---|---|
| Flights | `search_oneway_flights`, `search_roundtrip_flights` |
| Hotels | `search_hotels`, `find_hotel_by_name` |
| Free | all four |

Full parameter reference, the `api_usage` and `search_coverage` blocks, and self-hosting:
[mtnrabi/google-flights-mcp](https://github.com/mtnrabi/google-flights-mcp). Setup guides:
[Claude](https://flightpowers.com/guides/google-flights-mcp-claude),
[Cursor](https://flightpowers.com/guides/google-flights-mcp-cursor),
[Cline](https://flightpowers.com/guides/google-flights-mcp-cline). Overview: [flightpowers.com/mcp](https://flightpowers.com/mcp).

## Without MCP

The same data over REST at `https://api.flightpowers.com`, with a typed Python client:
[mtnrabi/google-flights-api-python](https://github.com/mtnrabi/google-flights-api-python).

## License

MIT (this README and the snippets). The servers are a hosted service; see their
[terms](https://flights.flightpowers.com/terms).
