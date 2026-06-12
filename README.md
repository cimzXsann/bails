
# cimzXsann

A high-performance, production-grade WhatsApp automation library built for reliability and scalability. YakuzaXsilence provides a robust abstraction layer over the Baileys protocol, ensuring seamless connectivity and advanced pairing mechanisms.

## Author

**Cimz**

## Core Capabilities

YakuzaXsilence is engineered to address common pain points in WhatsApp bot development, specifically session stability and authentication complexity. The library offers:

- **Full Node.js Compatibility:** Explicit support for the latest Node.js versions (v18, v20, v22, and beyond) without dependency conflicts or native module warnings.
- **Custom Pairing Interface:** A streamlined, programmable pairing method that bypasses QR code scanning, enabling deployment in headless server environments.
- **Anti-Error Connection Logic:** Proprietary retry algorithms and state management that prevent common disconnection errors (`Connection Closed`, `Stream Errored`) and handle session restoration automatically.

## Key Features

- **Modern Runtime Support:** Optimized for current LTS and Current releases of Node.js.
- **Advanced Authentication:** Support for standard QR pairing and custom code-based pairing.
- **Resilient Socket Management:** Automated reconnection handling with exponential backoff to mitigate rate-limiting and network jitter.
- **TypeScript First:** Fully typed with comprehensive type definitions for an enhanced developer experience.

## Installation

```bash
npm install yakuza-xsilence
```

## Usage

### Standard QR Authentication

```typescript
import YakuzaXsilence from 'yakuza-xsilence';

async function startBot() {
    const client = new YakuzaXsilence({
        printQRInTerminal: true
    });

    client.ev.on('connection.update', (update) => {
        const { connection } = update;
        if (connection === 'open') {
            console.log('Connection established.');
        }
    });

    await client.connect();
}

startBot();
```

### Custom Pairing (Anti-Error Mode)

This method is recommended for server deployments where scanning a QR code is not feasible. It utilizes the `customPairing` configuration to inject the code directly into the authentication flow.

```typescript
import YakuzaXsilence from 'yakuza-xsilence';

async function startWithPairingCode() {
    const phoneNumber = '6281234567890'; // International format without '+'

    const client = new YakuzaXsilence({
        customPairing: {
            enabled: true,
            phoneNumber: phoneNumber,
            showCode: (code) => {
                // This code is typically 8 digits. Log it or send it to a frontend.
                console.log(`[YAKUZA] Pairing Code: ${code}`);
            }
        },
        // Anti-error configuration (default: true)
        retryRequestDelayMs: 3000,
        maxRetries: 10
    });

    client.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update;
        if (connection === 'close') {
            // YakuzaXsilence will automatically attempt to reconnect using the saved session.
            console.log('Connection closed. Reconnecting automatically...');
        } else if (connection === 'open') {
            console.log('Client is ready.');
        }
    });

    await client.connect();
}

startWithPairingCode();
```

## Configuration Options

| Option | Type | Description | Default |
| :--- | :--- | :--- | :--- |
| `customPairing.enabled` | `boolean` | Activates the custom pairing flow (anti-error). | `false` |
| `customPairing.phoneNumber` | `string` | The target WhatsApp Business/User number. | `null` |
| `customPairing.showCode` | `function` | Callback function that receives the pairing string. | `null` |
| `printQRInTerminal` | `boolean` | Displays QR code in terminal for standard authentication. | `true` |
| `authDir` | `string` | Custom directory path for session credential storage. | `./auth_info_yakuza` |
| `retryRequestDelayMs` | `number` | Delay between reconnection attempts in milliseconds. | `3000` |
| `maxRetries` | `number` | Maximum attempts to reconnect before throwing a fatal error. | `10` |
| `browser` | `string[]` | Browser identifier sent to WhatsApp servers. | `['YakuzaXsilence', 'Chrome', '1.0.0']` |

## Session Persistence

YakuzaXsilence automatically manages credential storage. To maintain state across server restarts, ensure the working directory is writable.

```typescript
const client = new YakuzaXsilence({
    authDir: './auth_state' // Custom directory for session storage
});
```

## Event Handling

YakuzaXsilence extends the standard Baileys event emitter. Common events include:

```typescript
client.ev.on('messages.upsert', async (m) => {
    // Handle incoming messages
    console.log('New message received:', m.messages[0].key.remoteJid);
});

client.ev.on('creds.update', (creds) => {
    // Credentials have been updated (automatically saved)
    console.log('Session credentials refreshed.');
});
```

## Error Handling and Anti-Error Mechanism

The anti-error mechanism is enabled by default and provides:

1.  **Automatic Session Refresh:** Detects expired credentials and attempts renewal.
2.  **Connection Keep-Alive:** Sends periodic pings to prevent socket timeouts.
3.  **Graceful Reconnection:** Implements exponential backoff strategy to avoid server-side rate limiting.
4.  **Custom Pairing Fallback:** If QR scanning fails, the library can seamlessly switch to code-based pairing.

## Node.js Version Policy

| Version | Status |
| :--- | :--- |
| 22.x (Current) | Fully Supported |
| 20.x (LTS) | Fully Supported |
| 18.x (Maintenance) | Fully Supported |
| 16.x | Limited Support (Deprecated) |
| < 16.x | Unsupported |

## Common Issues and Solutions

| Issue | Solution |
| :--- | :--- |
| `Connection Closed` loop | Ensure `retryRequestDelayMs` is set to at least `3000` and `maxRetries` is sufficient. |
| Pairing code not received | Verify phone number format (international, no '+' symbol) and check spam folder if using SMS. |
| Session deleted on restart | Verify `authDir` path is absolute or relative to the correct working directory. |
| `TypeError` on Node.js 16 | Upgrade Node.js to v18 or higher. YakuzaXsilence relies on modern `fetch` and `crypto` APIs. |

## Development

```bash
# Clone the repository
git clone https://github.com/OmhcSilence/yakuza-xsilence.git
cd yakuza-xsilence

# Install dependencies
npm install

# Build the project
npm run build

# Run tests
npm test
```

## Contributing

Contributions are welcome. Please ensure any pull requests maintain compatibility with the latest Node.js LTS release and include appropriate tests for the anti-error pairing mechanism. Submit issues and feature requests via the GitHub issue tracker.

## Security

YakuzaXsilence stores credentials locally in the `authDir` specified. It is the responsibility of the developer to secure this directory on production servers and to never commit `creds.json` files to public version control.

## License

MIT License

Copyright (c) 2019 OmhcSilence

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Acknowledgments

YakuzaXsilence is built upon the foundational work of the Baileys library and the open-source WhatsApp protocol research community.

## FORK AJA DASAR DEV RENAME BUDAK AI AWOKWOKWOWKWOWKOK
