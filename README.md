# x402 Auction MCP Server

A TypeScript MCP server that wraps the Palette Finance x402 auction API. It exposes auction information, bid creation, payment status, and recent bids to clients using the MCP stdio transport.

The server returns structured data for the client to present. Auction availability, pricing, allocation, and payment instructions come from the upstream service.

## Build from source

Requirements: Node.js and npm.

```bash
git clone https://github.com/nakazanie-ton/x402-auction-mcp.git
cd x402-auction-mcp
npm ci
npm run build
```

Configure your MCP client with the absolute path to the compiled entry point:

```json
{
  "mcpServers": {
    "x402-auction": {
      "command": "node",
      "args": ["/absolute/path/to/x402-auction-mcp/build/index.js"]
    }
  }
}
```

Use the client's configuration format and restart or reload its MCP connection as needed. This source-based setup does not depend on an npm registry release.

## Available tools

| Tool | Parameters | Purpose |
| --- | --- | --- |
| `get_auction_info` | None | Read auction status, pricing, and fundraising progress. |
| `create_auction_bid` | `ton_amount`, `wallet` | Create a bid or return an existing bid's state and payment instructions. |
| `check_bid_status` | `bid_id` | Read the state of a specific bid. |
| `get_my_bid` | `wallet` | Look up a bid by the participant's wallet address. |
| `get_recent_bids` | Optional `limit` | List recent bids; default 20, maximum 100. |

The current implementation validates bid amounts from 1 to 100 TON. Wallet fields identify the participant; they are **not the payment destination**.

## Bid and payment flow

1. Read `get_auction_info` to check the current auction state.
2. Use `get_my_bid` to check for an existing bid.
3. Create a bid only when the user intends to participate.
4. If payment is required, present the returned `payment.recipient`, amount, required comment, expiry, and wallet link.
5. The user reviews and completes payment in their wallet. This server does not hold wallet keys or sign the payment.
6. Use `check_bid_status` or `get_my_bid` to read subsequent payment, allocation, or refund state.

Payment expiry and auction outcomes must be read from the response. Do not assume that an auction is active or that a past price, allocation, or refund policy still applies.

## API integration

The base API URL is defined in [src/constants.ts](src/constants.ts). The client uses the auction info, bid, my-bid, and bids endpoints, including HTTP 402 payment-required responses.

The implementation handles auction closure, missing bids, validation failures, and rate-limit responses. Availability and compatibility with the live upstream API are not verified by this README.

## Development

```bash
npm test
npm run build
npm start
```

`npm run dev` compiles and starts the server. It does not watch files.

| Path | Responsibility |
| --- | --- |
| [src/index.ts](src/index.ts) | MCP tool schemas and stdio entry point |
| [src/handlers.ts](src/handlers.ts) | Validation and bid/payment state handling |
| [src/api.ts](src/api.ts) | HTTP API client |
| [src/types.ts](src/types.ts) | Request and response types |
| [src/constants.ts](src/constants.ts) | API URL and limits |
| [src/__tests__](src/__tests__) | Jest tests |

See [TEST_GUIDE.md](TEST_GUIDE.md) for additional test notes and [PUBLISHING.md](PUBLISHING.md) for package publishing instructions.

## License

[MIT](LICENSE).
