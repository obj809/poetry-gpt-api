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

Create a `.env` file in the project root (copy the template and fill in values):

```bash
cp .env.example .env
```

```bash
OPENAI_API_KEY=sk-...                       # required (OpenAI key, or a LiteLLM virtual key)
OPENAI_MODEL=gpt-5.4-nano                    # optional, defaults to gpt-5.4-nano
OPENAI_BASE_URL=https://api.openai.com/v1    # optional, defaults to OpenAI; see "Routing through LiteLLM"
CORS_ORIGIN=http://localhost:5173,http://localhost:3000  # comma-separated allowlist
```

`.env` is gitignored and should never be committed.

### Routing through LiteLLM

The service talks to any OpenAI-compatible endpoint via `OPENAI_BASE_URL`. To send
requests through a local [LiteLLM](https://github.com/obj809/litellm-docker-container)
proxy instead of OpenAI directly, set the base URL to the proxy and use a LiteLLM
virtual key as `OPENAI_API_KEY`:

```bash
OPENAI_API_KEY=sk-<litellm-virtual-key>
OPENAI_BASE_URL=http://litellm:4000/v1      # docker service name on the shared network
```

When running on the host with `npm run dev` (not inside Docker), the `litellm`
hostname won't resolve — use the published port instead:

```bash
OPENAI_BASE_URL=http://localhost:4000/v1 npm run dev
```

In Docker, `docker-compose.yml` attaches this service to the external
`litellm-docker-container_default` network so it can reach the proxy by name.

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

### Networks

`docker-compose.yml` attaches the container to two **external** Docker networks
in addition to the project's own `default` network:

| Network | Purpose |
| ------- | ------- |
| `litellm-docker-container_default` | Outbound — lets this service reach the LiteLLM proxy by name at `http://litellm:4000/v1` (see [Routing through LiteLLM](#routing-through-litellm)). |
| `webnet` | Inbound — lets other containers on `webnet` reach this service by name at `http://poetry-gpt-api:3001`. |

Because both are declared `external: true`, they must already exist before you run
`docker compose up`, otherwise compose will error. They are created by their
owning projects; to create `webnet` manually if needed:

```bash
docker network create webnet
```

Verify which networks the running container joined:

```bash
docker inspect poetry-gpt-api \
  --format '{{range $k,$v := .NetworkSettings.Networks}}{{$k}} {{end}}'
```

## Project layout

```
src/index.ts        # the entire application
Dockerfile          # multi-stage build (builder -> slim runtime)
docker-compose.yml  # build, healthcheck, and external network wiring (litellm, webnet)
```