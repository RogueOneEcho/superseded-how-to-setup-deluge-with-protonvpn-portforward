FROM alpine
RUN apk add --no-cache bash curl jq
COPY preflight /preflight
RUN chmod a+x /preflight
ENTRYPOINT /preflight
