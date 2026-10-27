# JamAI Base's SvelteKit frontend, built from the same pinned commit as Owl
# (docker/Dockerfile.frontend upstream).
FROM alpine/git:v2.49.1 AS src
ARG JAMAI_COMMIT=834459fcfca7458c03841c40ea1f7b80cc373e21
RUN git init -q /src && cd /src \
    && git fetch -q --depth 1 https://github.com/EmbeddedLLM/JamAIBase.git "$JAMAI_COMMIT" \
    && git checkout -q FETCH_HEAD

FROM node:20-alpine
ENV PUBLIC_ADMIN_ORGANIZATION_ID="0"
WORKDIR /app
COPY --from=src /src/services/app .
RUN rm -f .env && npm ci \
    && PUBLIC_JAMAI_URL= PUBLIC_IS_SPA=false CHECK_ORIGIN=false npx vite build \
    && mv temp build
ENV NODE_ENV=production \
    BODY_SIZE_LIMIT=Infinity \
    FRONTEND_PORT=4000
CMD ["node", "server"]
