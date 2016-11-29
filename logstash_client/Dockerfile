FROM logstash:5

COPY client.conf /logstash.conf

CMD ["-f", "/logstash.conf", "--config.reload.automatic"]
