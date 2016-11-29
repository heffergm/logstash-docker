## Overview

Docker compose setup to allow running of the ELK stack using a Redis transport. Running the stack will start the following components:

- Elasticsearch
- Redis
- Kibana
- Logstash (client)
- Logstash (worker)

The Logstash worker's job is simply to watch Redis for new data and output it to Elasticsearch.
The Logstash client is configured to pick up anything with a .log suffix in the `/logs` on the container and output anything it finds there into the Redis transport. Logstash on its own will reload its configuration file (`/client.conf` or `/worker.conf`) every three seconds.

Both /logs and the logstash client and worker config files are mounted as volumes from the local host, meaning you can place log files directly in the logs/ directory locally and they will get picked up by logstash inside the container. Similarly, you can edit the client or worker configs locally and the changes will be picked up by the running processes.

Therefore, the workflow would be along the lines of:

Start the stack. Echo some test data to a file: `echo "FINDME" >> logs/test.log`. You should see output from docker stdout similar to the following:

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

You can now modify the client or worker configurations (`./logstash_{client,worker}/{client,worker}.conf`) to test whatever you'd like. You will see a message similar to the following when the configuration is reloaded:

```
logstash_client_1  | 19:50:30.202 [Ruby-0-Thread-3: /usr/share/logstash/vendor/bundle/jruby/1.9/gems/stu
d-0.0.22/lib/stud/task.rb:22] WARN  logstash.agent - fetched new config for pipeline. upgrading.. {:pipe
line=>"main", :config=>"input {\n  file {\n    type => \"logstash-client-logs\"\n    path => [ \"/logs/*
.log\" ]\n    tags => [ \"logstash-client-logs\" ]\n  }\n}\n\noutput {\n  stdout {\n    codec => rubydeb
ug\n  }\n\n  redis {\n    host      => \"${LOGSTASH_REDIS_HOST:redis}\"\n    key       => \"${LOGSTASH_R
EDIS_KEY:logstash}\"\n    data_type => \"${LOGSTASH_DATA_TYPE:list}\"\n  }\n}\n\n"}
```

## Running

`docker-compose up`

## Access Points

The following ports are forwarded locally for your convenience:

- Redis: localhost:5601
- [Kibana](http://localhost:5601): localhost:5601
- [Elasticsearch](http://localhost:9200): localhost:9200
