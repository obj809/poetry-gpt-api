# commands.md

## Local dev
npx ts-node-dev src/index.ts

# Local dev routed through a host-published LiteLLM proxy:
OPENAI_BASE_URL=http://localhost:4000/v1 npm run dev

## Docker (via docker-compose)
docker compose up -d --build          # build and start in the background
docker compose up -d --force-recreate # recreate to pick up .env changes
docker compose logs -f                # follow logs
docker compose down                   # stop and remove

## Production deploy (VPS, via docker-compose.prod.yml)
git clone git@github.com:obj809/poetry-gpt-api.git ~/poetry-gpt-api
cd ~/poetry-gpt-api
cp .env.example .env                  # then edit: LiteLLM virtual key + CORS origins
docker network create webnet
docker compose -f docker-compose.prod.yml up -d --build
docker compose -f docker-compose.prod.yml logs -f
docker compose -f docker-compose.prod.yml down

## Networks (external, must exist before `up`)
docker network create webnet          # create if missing
docker inspect poetry-gpt-api \
  --format '{{range $k,$v := .NetworkSettings.Networks}}{{$k}} {{end}}'   # list joined networks

## Health
curl -s http://localhost:3001/health
docker inspect --format '{{.State.Health.Status}}' poetry-gpt-api

## Smoke test (/generate returns a limerick)
curl -s http://localhost:3001/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Say hello in one sentence"}'

## Debug: hit the LiteLLM proxy directly (isolates litellm -> OpenAI)
curl -s http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer <litellm-virtual-key>" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-5.4-nano","messages":[{"role":"user","content":"hi"}]}'
