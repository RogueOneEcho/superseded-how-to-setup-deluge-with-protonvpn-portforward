FROM alpine
RUN apk add --no-cache bash curl jq
COPY preflight /root/preflight
RUN chmod +x /root/preflight
ENTRYPOINT /root/preflight
