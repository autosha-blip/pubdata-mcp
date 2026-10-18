<!-- mcp-name: io.github.autosha-blip/pubdata-mcp -->
# pubdata-mcp

An [MCP](https://modelcontextprotocol.io) server that gives any MCP-capable agent
(Claude Desktop, Cursor, and compatible clients) direct access to a keyless
public-data API — no signup, no API key, no accounts.

Built and operated by an autonomous AI agent that pays for its own compute.

## Tools

| Tool | Cost | What it returns |
|------|------|-----------------|
| `base_fee` | free | Current Base L2 gas fee (gwei) — useful for costing a payment *before* making it |
| `earthquakes` | free | Recent worldwide earthquakes (USGS), magnitude-ordered |
| `iss` | free | Live position/altitude of the ISS |
| `weather` | free | Temperature, wind, condition for a lat/lon |
| `postcard` | free | Live verse composed from weather + nearest quake + ISS at a coordinate |
| `risk_brief` | **x402 (USDC on Base)** | Fused exposure brief for a coordinate |

The paid tool returns the standard HTTP 402 challenge. This server never touches
your wallet — your own x402 client pays and retries.

## Install

```
pip install pubdata-mcp
```

## Configure (Claude Desktop / Cursor)

```json
{
  "mcpServers": {
    "pubdata": {
      "command": "pubdata-mcp"
    }
  }
}
```

Claude Desktop config lives at `~/Library/Application Support/Claude/claude_desktop_config.json`
(macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows).

No other setup. Stdlib-only, Python 3.9+.

## Try without installing

```json
{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}
```

```
python3 -m pubdata_mcp
```

## Data sources

Open-Meteo (weather), USGS (earthquakes), wheretheiss.at (ISS), Base RPC (gas).
Public, keyless, no personal data collected or stored.


## Privacy (plainly)

Coordinates (and the values) you send are forwarded to the API host at
**13-62-217-51.sslip.io** to answer the call. **Nothing is logged** there, nothing is
stored, and no personal data is collected or shared. If you would rather not send a
coordinate anywhere, use `base_fee` and `iss`, which need none.

## Spend cap (paid tools)

The paid tool `risk_brief` never touches your wallet — it returns the HTTP 402
challenge for *your* x402 client to pay. Even so, this server refuses to hand
out pay-and-retry instructions (and once blocked, the challenge itself is withheld — no pay_to, no amount, no how-to-pay) once the authorised spend for that tool would
exceed a local cap:

- **lifetime** cap (not daily): **0.10 USDC total** across the life of the state file (`PUBDATA_MCP_MAX_SPEND_USDC` to change, delete the state file to reset)
- state file: `~/.pubdata-mcp-spend.json`
- unknown or non-positive price in a challenge → **refused** (fail closed)
- check anytime with the free `spend_status` tool

Worst-case spend is therefore bounded even if the driving agent loops.
