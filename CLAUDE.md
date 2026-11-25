# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an Amazon MCP (Model Context Protocol) Server that enables AI assistants to interact with Amazon services through web scraping. The server uses Puppeteer for browser automation and exposes Amazon functionality through MCP tools.

## Development Commands

```bash
# Install dependencies (use -D flag for Puppeteer)
npm install -D

# Build TypeScript to JavaScript (also compiles the test scripts into build/)
npm run build

# Clean mock HTML files
npm run clean
```

## Architecture

### Core Components

- **MCP Server** (`src/index.ts`): Defines and exposes tools via the MCP protocol
- **Products** (`src/products.ts`): Product search, product details and product variants
- **Cart** (`src/cart.ts`): Cart content, add, remove and clear
- **Orders** (`src/orders.ts`): Orders history
- **Configuration** (`src/config.ts`): Settings, paths, cookie loading and Amazon domain detection
- **Browser Utils** (`src/utils.ts`): Puppeteer browser setup, cookie file loading and login detection

Each tool call launches its own headless browser, loads the page, then parses the HTML with Cheerio.

### Key Dependencies

- `@modelcontextprotocol/sdk`: MCP framework
- `puppeteer`: Browser automation
- `cheerio`: HTML parsing
- `zod`: Schema validation

### Authentication

The server requires Amazon cookies in `amazonCookies.json` in the project root (gitignored, never commit it):
- Create or refresh it with `node login_and_save_cookies.cjs`, which opens a browser for you to sign in and saves the cookies. See `RELOGIN.md`.
- Or export cookies from a logged-in browser with a cookie export extension (format: see `amazonCookies.example.json`).
- Cookies are read once at startup, so Claude Desktop must be fully quit and reopened after they change.
- The Amazon domain (amazon.com, amazon.de, ...) is detected from the cookies.

## Important Implementation Details

### Browser Automation
- Uses headless Chrome with specific flags to avoid detection
- Implements user agent spoofing
- Handles Amazon's anti-bot measures

### Error Handling
- Detects login page redirects and throws authentication errors
- Each tool returns errors as text instead of crashing the server
- There is no automatic retry; a single connection failure is usually Amazon bot detection, so retry once manually

### Mock Mode
- Set `USE_MOCKS = true` in `src/config.ts` to read pages from `mocks/` instead of Amazon
- `EXPORT_LIVE_SCRAPING_FOR_MOCKS = true` saves live pages to `mocks/` (they contain account data, so keep it off)
- Useful for development and testing without hitting Amazon

### Logging
- The server logs to stderr; Claude Desktop saves it to `%APPDATA%\Claude\logs\mcp-server-amazon.log` on Windows (`~/Library/Logs/Claude/mcp-server-amazon.log` on macOS)
- Check logs for debugging authentication or scraping issues

## MCP Tools Exposed

1. `search-products`: Search Amazon catalog
2. `get-product-details`: Get detailed product information
3. `get-product-variants`: List a product's variations (size/color → child ASIN) with price, stock and delivery per variation; optional `size`/`color` filters, fetches at most `limit` child pages (default 5, max 15)
4. `get-orders-history`: View past orders
5. `get-cart-content`: View current cart
6. `add-to-cart`: Add items to cart
7. `remove-from-cart`: Remove a single item from the cart by ASIN (leaves other items untouched)
8. `clear-cart`: Remove all items from cart

There is no purchase tool: checkout happens in a real browser.

## Testing Approach

There is no test framework. `src/*.test.ts` are scripts that `npm run build` compiles; run them with `node build/<name>.test.js`.
- `amazon.getProductVariants.test.js` is offline (parses `mocks/getProductDetails.html`) and safe to run anytime.
- The other test scripts hit live Amazon with your cookies. `addToCart`, `removeFromCart` and `clearCart` change your real cart.
- Manual testing through Claude Desktop, and log analysis for debugging.

## Common Issues

1. **Authentication failures**: Refresh cookies (see `RELOGIN.md`), then fully restart Claude Desktop
2. **Scraping failures**: Amazon HTML structure may have changed
3. **Rate limiting**: Add delays between requests if needed
