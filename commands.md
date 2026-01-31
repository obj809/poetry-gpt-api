# commands.md

npx ts-node-dev src/index.ts

docker build -t openai-api .
docker run --rm -p 3001:3001 \
  -e PORT=3001 \
  -e OPENAI_API_KEY="your_key" \
  -e OPENAI_MODEL="gpt-4o-mini" \
  openai-api

curl -s http://localhost:3001/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Say hello in one sentence"}'