FROM alpine
RUN apk add --no-cache bash curl jq
COPY monitor /root/monitor
RUN chmod +x /root/monitor
ENTRYPOINT /root/monitor
