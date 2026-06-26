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

## Networks (external, must exist before `up`)
docker network create webnet          # create if missing
docker inspect poetry-gpt-api \
  --format '{{range $k,$v := .NetworkSettings.Networks}}{{$k}} {{end}}'   # list joined networks

## Health
curl -s http://localhost:3001/health
docker inspect --format '{{.State.Health.Status}}' poetry-gpt-api         # container healthcheck

## Smoke test (/generate returns a limerick)
curl -s http://localhost:3001/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Say hello in one sentence"}'

## Debug: hit the LiteLLM proxy directly (isolates litellm -> OpenAI)
curl -s http://localhost:4000/v1/chat/completions \
  -H "Authorization: Bearer <litellm-virtual-key>" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-5.4-nano","messages":[{"role":"user","content":"hi"}]}'
