# Amazon MCP Server

This server allows you to interact with Amazon's services using the MCP (Model Context Protocol) framework. This lets you use your Amazon account through ChatGPT or Claude AI interfaces.

## Features

- **Product search**: Search for products on Amazon
- **Product details**: Retrieve detailed information about a specific product on Amazon
- **Product variants**: List a product's sizes/colors with the price, stock and delivery date of each, optionally filtered (e.g. size "XX-Large")
- **Cart management**: Add or remove single items, or clear your Amazon cart
- **Orders history**: Retrieve your recent Amazon orders details

There is no purchase tool: checkout happens in your own browser.

## Demo

Simple demo, showcasing a quick product search and purchase. (Recorded with the original fake purchase tool, which has since been removed: the connector now stops at the cart.)

![Demo GIF video](./demo.gif)

## Full Demo

Another more complex demo with products search, leveraging Claude AI recommendations to compare and make a decision, then purchase (also recorded with the removed fake purchase tool).

It showcases how natural and powerful the Amazon MCP integration could be inside a conversation

Video: https://www.youtube.com/watch?v=xas2CLkJDYg

## Install

Install dependencies

```sh
npm install -D
```

Build the project

```sh
npm run build
```

## Amazon login (cookies)

The server uses your Amazon session cookies, saved in `amazonCookies.json` in the project root. This file is gitignored: never commit or share it.

Create it with the login helper, which opens a browser for you to sign in (finish any CAPTCHA or OTP there) and saves the cookies:

```sh
node login_and_save_cookies.cjs
```

Or export your cookies from a logged-in browser with an extension like Cookie-Editor (format: see `amazonCookies.example.json`). The Amazon site (amazon.com, amazon.de, ...) is detected from the cookies.

When the server says you need to be logged in, refresh the cookies the same way (see `RELOGIN.md`).

## Claude Desktop Integration

Add the server to Claude Desktop's config file:

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "amazon": {
      "command": "node",
      "args": ["/Users/admin/dev/mcp-server-amazon/build/index.js"]
    }
  }
}
```

On Windows, use forward slashes in the path, e.g. `"C:/Users/you/mcp-server-amazon/build/index.js"`.

Fully quit and reopen Claude Desktop to apply the changes (on Windows, quit from the tray icon; closing the window is not enough). Do the same after rebuilding or refreshing cookies: the server only loads them at startup. You should now see the Amazon MCP server listed in the Claude Desktop app.

|                                  |                                    |
| :------------------------------: | :--------------------------------: |
| ![screenshot](./screenshot.webp) | ![screenshot2](./screenshot2.webp) |

## Troubleshooting

The MCP server logs its output to a file. If you encounter any issues, you can check the log file for more information.

- macOS: `~/Library/Logs/Claude/mcp-server-amazon.log`
- Windows: `%APPDATA%\Claude\logs\mcp-server-amazon.log`

A single connection error is usually Amazon's bot detection, not a logout: retry once before refreshing cookies.

## License

[The MIT license](./LICENSE)
