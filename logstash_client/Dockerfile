FROM logstash:5

RUN mkdir /logs
RUN echo \
      TEST_DATA > /logs/test.log

COPY client.conf /logstash.conf

CMD ["-f", "/logstash.conf"]
