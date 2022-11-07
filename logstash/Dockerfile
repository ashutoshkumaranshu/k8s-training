FROM docker.elastic.co/logstash/logstash-oss:8.5.0

RUN rm -f /usr/share/logstash/pipeline/logstash.conf && \
  bin/logstash-plugin install logstash-output-loki && \
  bin/logstash-plugin install logstash-filter-kubernetes
