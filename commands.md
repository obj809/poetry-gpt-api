# commands.md

## Local dev
npx ts-node-dev src/index.ts

## Docker (via docker-compose)
docker compose up -d --build      # build and start in the background
docker compose logs -f            # follow logs
docker compose down               # stop and remove

## Smoke test
curl -s http://localhost:3001/health

curl -s http://localhost:3001/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Say hello in one sentence"}'
