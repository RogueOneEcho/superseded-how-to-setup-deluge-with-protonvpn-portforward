FROM alpine
RUN apk add --no-cache bash curl jq
COPY --from=shared logging ./logging
COPY monitor ./monitor
RUN chmod a+x ./logging && chmod a+x ./monitor
ENTRYPOINT ./monitor
