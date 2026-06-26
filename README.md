# PoetryGPT API

A minimal Express + TypeScript service that proxies prompts to the OpenAI Chat Completions API. It exposes a single endpoint, `POST /generate`, that takes a text prompt and returns the model's response. The model is configured via `OPENAI_MODEL` (defaults to `gpt-5.4-nano`) — there is no prompt-shaping logic in the server itself, so it works as a generic OpenAI relay.

## Why a proxy?

The server keeps your `OPENAI_API_KEY` on the backend instead of shipping it to the browser, and applies a CORS allowlist so only approved frontends can call it.

## Requirements

- Node.js 20+
- An OpenAI API key

## Setup

```bash
npm install
```

Create a `.env` file in the project root:

```bash
OPENAI_API_KEY=sk-...                       # required
OPENAI_MODEL=gpt-5.4-nano                    # optional, defaults to gpt-5.4-nano
CORS_ORIGIN=http://localhost:5173,http://localhost:3000  # comma-separated allowlist
```

`.env` is gitignored and should never be committed.

## Running

```bash
npm run dev      # hot-reload dev server (ts-node-dev)
npm run build    # compile TypeScript to dist/
npm start        # run the compiled build
```

The server always listens on **port 3001**.

## API

### `POST /generate`

**Request body**

```json
{ "prompt": "Write a haiku about the sea" }
```

**Response**

```json
{ "text": "..." }
```

**Errors**

| Status | Body | Cause |
| ------ | ---- | ----- |
| 400 | `{ "error": "Invalid prompt" }` | `prompt` missing or not a string |
| 500 | `{ "error": "Server misconfigured" }` | `OPENAI_API_KEY` not set |
| 500 | `{ "error": "OpenAI request failed" }` | Upstream OpenAI call failed |

**Example**

```bash
curl -s http://localhost:3001/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Say hello in one sentence"}'
```

## Configuration notes

- **`CORS_ORIGIN`** is an allowlist of exact origins. If it is empty, no browser origin is permitted; requests without an `Origin` header (e.g. `curl`, server-to-server) are always allowed.
- **`PORT`** in `.env` is ignored — the listen port is hardcoded to `3001` in `src/index.ts`.

## Docker

The project ships with a `docker-compose.yml`. It builds the image, reads
configuration from `.env`, publishes port `3001`, and runs a healthcheck against
`/health`.

```bash
docker compose up -d --build   # build and start in the background
docker compose logs -f         # follow logs
docker compose down            # stop and remove
```

The service is then available at `http://localhost:3001`. Make sure `.env`
contains a valid `OPENAI_API_KEY` before starting — compose passes it into the
container via `env_file`.

## Project layout

```
src/index.ts        # the entire application
Dockerfile          # multi-stage build (builder -> slim runtime)
docker-compose.yml  # deployment behind a reverse proxy network
```