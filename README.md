## Overview

Docker compose setup to allow running of the ELK stack using a Redis transport. Running the stack will start the following components:

- Elasticsearch
- Redis
- Kibana
- Logstash (client)
- Logstash (worker)

The Logstash worker's job is simply to watch Redis for new data and output it to Elasticsearch.
The Logstash client is configured to watch `/logs` on the container and output anything it finds there into the Redis transport. Logstash on its own will reload its configuration file (`/client.conf` or `/worker.conf`) every three seconds.

Therefore, the workflow would be along the lines of:

Start the stack. Log into the Logstash client machine (`docker exec -i -t ${docker_id} /bin/bash`) and echo some test data to a file: `echo "FINDME" >> a_test.log`. You should see output from docker stdout similar to the following:

```
logstash_client_1  | {
logstash_client_1  |           "path" => "/logs/test.log",
logstash_client_1  |     "@timestamp" => 2016-11-29T17:09:32.543Z,
logstash_client_1  |       "@version" => "1",
logstash_client_1  |           "host" => "a55aab70d5c0",
logstash_client_1  |        "message" => "stuff",
logstash_client_1  |           "type" => "logstash-client-logs",
logstash_client_1  |           "tags" => [
logstash_client_1  |         [0] "logstash-client-logs"
logstash_client_1  |     ]
logstash_client_1  | }

logstash_worker_1  | {
logstash_worker_1  |           "path" => "/logs/test.log",
logstash_worker_1  |     "@timestamp" => 2016-11-29T17:09:32.543Z,
logstash_worker_1  |       "@version" => "1",
logstash_worker_1  |           "host" => "a55aab70d5c0",
logstash_worker_1  |        "message" => "stuff",
logstash_worker_1  |           "type" => "logstash-client-logs",
logstash_worker_1  |           "tags" => [
logstash_worker_1  |         [0] "logstash-client-logs"
logstash_worker_1  |     ]
logstash_worker_1  | }
```

Navigate to Kibana (see Access Points below), and you should see your log event. Note that on initial startup, you'll need to set up the default mapping in Kibana. Accepting the defaults will accomplish this.

You can now modify the client configuration (`/client.conf`) to test whatever you'd like.

## Running

`docker-compose up`

## Access Points

The following ports are forwarded locally for your convenience:

- Redis: localhost:5601
- [Kibana](http://localhost:5601): localhost:5601
- [Elasticsearch](http://localhost:9200): localhost:9200
