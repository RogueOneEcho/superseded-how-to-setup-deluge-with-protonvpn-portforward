FROM alpine
RUN apk add --no-cache bash curl jq
COPY monitor /root/monitor
ENTRYPOINT /root/monitor
