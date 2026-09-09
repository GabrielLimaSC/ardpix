# ArdPix

A payment-confirmation display: an ESP8266 microcontroller polls a small Flask API for an
account's balance and shows the amount just received on an LCD screen, so a seller gets visual
confirmation a payment landed without checking a phone.

## Stack

- **API** (`api/main.py`): Python, Flask, `waitress` as the production server
- **Device** (`ardpix.ino`): ESP8266 (Arduino), `ESP8266WiFi`, `ESP8266HTTPClient`,
  `WiFiClientSecureBearSSL`, `LiquidCrystal_I2C` for a 16x2 I2C LCD

## Implementation notes

- The API wraps the [Asaas](https://www.asaas.com/) payment platform's balance endpoint
  (`GET /api/v3/finance/balance`), keeps the last seen balance in memory (`saldos_anteriores`),
  and on each poll returns the delta since the previous read as `R$X.XX recebido!` — this is what
  turns "current balance" into "amount just received" for the display.
- The device connects to Wi-Fi, polls the API's root endpoint over HTTPS on a loop (1s delay),
  and prints whatever text comes back straight to the LCD — the device has no logic of its own,
  it's a dumb display for whatever the API returns.
- `client->setInsecure()` skips TLS certificate validation on the ESP8266 side — acceptable for a
  personal/hobby device on a trusted network, not for anything handling sensitive data.

## How to run

**API:**
```bash
cd api
pip install flask requests waitress
# edit main.py: set your Asaas access_token
python main.py
```

**Device:** open `ardpix.ino` in the Arduino IDE with ESP8266 board support installed, set `ssid`,
`password`, and the API URL in `https.begin(*client, "your_api")`, then flash to the board.

## Limitations

- Balance state is in-memory only (`saldos_anteriores` list) — restarting the API loses the
  reference point and the next poll will report the full balance as "received."
- No auth on the Flask endpoint itself; anyone who can reach it can read the balance delta.
- Wi-Fi credentials and the API token are hardcoded placeholders meant to be edited directly in
  source, not loaded from environment/config.

<!-- TODO GABRIEL: is this still running anywhere, or was it a one-off build? -->
