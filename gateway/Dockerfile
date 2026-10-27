# JamAI Base OSS has no login: every request acts as the built-in admin. This
# gateway puts the UI behind a password and the API behind a Bearer token.
FROM caddy:2.11.7-alpine
COPY Caddyfile /etc/caddy/Caddyfile
CMD ["/bin/sh", "-c", "export JAMAI_PASSWORD_HASH=$(caddy hash-password --plaintext \"$JAMAI_PASSWORD\") && exec caddy run --config /etc/caddy/Caddyfile --adapter caddyfile"]
