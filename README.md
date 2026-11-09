# WhatsApp Desk Starter

A small, clean WhatsApp Cloud API webhook server. It verifies with Meta, checks every
event's signature, answers fast, skips duplicate deliveries, and logs to stdout.

This is the Part 1 base for the course. It uses a made-up business called DemoDesk.

## What it does

- `GET /health` returns `{ "ok": true }`
- `GET /webhook` answers Meta's verification request
- `POST /webhook` accepts events, rejects bad signatures, replies to text and files
- Logs one JSON line per event to stdout (no log files, on purpose)

## Run it locally

```bash
npm install
cp .env.example .env     # fill in your own values
npm start
```

Check each step:

```bash
curl localhost:3000/health
curl "localhost:3000/webhook?hub.mode=subscribe&hub.verify_token=YOUR_VERIFY_TOKEN&hub.challenge=123"
npm run event                       # signed fake event, expect 200
npm run event -- --bad-signature    # expect 401
npm run event -- --repeat           # second one logs duplicate_skipped
```

Meta cannot reach localhost. Use a tunnel such as ngrok to test with the real dashboard.

## Environment variables

| Variable | What it is |
|---|---|
| VERIFY_TOKEN | A string you invent. Must match the Verify token in the Meta dashboard |
| APP_SECRET | Meta app secret, used to check the signature header |
| WHATSAPP_TOKEN | Permanent System User token, not the 24 hour test token |
| PHONE_NUMBER_ID | ID of the phone number that sends messages |
| WABA_ID | ID of the WhatsApp Business Account. Different from the phone number ID |
| GRAPH_VERSION | Current Graph API version, from Meta's docs |
| TEMPLATE_LANGUAGE | Must match your approved templates exactly (en or en_US) |

`.env.example` stays in git and holds placeholders only. `.env` is never committed.

## Deploy (Render)

1. Push this repo to GitHub, private.
2. Create a Web Service, build command `npm install`, start command `npm start`.
3. Add the variables above in the Render dashboard, pasted cleanly with no quotes or spaces.
4. In Meta, set the callback URL to `https://YOUR_APP.onrender.com/webhook` and your verify token.
5. Subscribe the `messages` field.
6. Run `npm run waba -- subscribe` once, then `npm run waba -- check`.

Free instances can sleep when idle, so the first request after a pause may be slow.
Check Render's current free tier behavior before relying on it for production.

## When nothing works, check in this order

1. Is the server up? Open `/health`.
2. Did Meta verify the callback URL?
3. Is your app subscribed to the WABA? `npm run waba -- check`
4. Is the `messages` field subscribed in the dashboard?
5. Do the logs show incoming POSTs? Look for `message_received`.
6. Is the token valid, with the right permissions?
7. Is the phone number registered and not restricted?
8. Is the template approved, and does its language code match?

"Verified" only proves the URL works. It does not prove events are being sent to it.

## What is not here yet

Database, saved messages, step-by-step conversation flow, staff dashboard, fraud checks
and any AI layer come in later modules. Nothing here has been tested against live Meta
traffic yet, so run through the checks above on your own test number first.
