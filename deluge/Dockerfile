FROM lscr.io/linuxserver/deluge
ARG CROSS_SEED_API_KEY
COPY --from=shared logging ./logging
COPY cross-seed ./cross-seed
RUN sed -i 's/YOUR_API_KEY/'"${CROSS_SEED_API_KEY}"'/g' ./cross-seed
RUN chmod a+x ./logging && chmod a+x ./cross-seed
