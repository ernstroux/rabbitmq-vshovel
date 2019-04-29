## RabbitMQ Variable Shovel

RabbitMQ vShovel is a WAN-friendly tool for moving messages from
a queue residing on an AMQP source broker, to a remote endpoint of any desired type, such as an HTTP endpoint.

Each endpoint is an implementation of the `rabbit_vshovel_endpoint` behaviour. The following message shovelling operations and protocol endpoints are currently supported, with more to be introduced as the plugin further grows:

- [AMQP 0.9.1](https://www.rabbitmq.com/amqp-0-9-1-reference.html) (RabbitMQ) to an AMQP 0.9.1 endpoint.
- AMQP 0.9.1 to an [HTTP 1.1](https://www.w3.org/Protocols/rfc2616/rfc2616.html) endpoint.


## Supported RabbitMQ Versions

This plugin is compatible with RabbitMQ 3.5.x and 3.6.x.


## Usage

### Packaging

Download pre-compiled versions from https://github.com/cloudamqp/rabbitmq-vshovel/releases

### Build

Clone and build the plugin by executing `make`. To create a package, execute `make dist` and find the `.ez` package file in the plugins directory.

## Configuration

### 1. Dynamic configuration

#### Unix

```
rabbitmqctl set_parameter vshovel my-vshovel \
  '{"src-uri": "amqp://fred:secret@host1.domain/my_vhost",
    "src-queue": "my-queue", "dest-type":"http",
    "dest-uri":"http://remote-host1.domain", "dest-vsn": "1.1",
    "dest-args": {"method": "post", "keep_alive_timeout" : 10000}}'
```

#### Windows

```
rabbitmqctl set_parameter vshovel my-vshovel ^
  "{""src-uri"": ""amqp://fred:secret@host1.domain/my_vhost"", ^
    ""src-queue"": ""my-queue"", ""dest-type"":""http"", ^
    ""dest-uri"":""http://remote-host1.domain"", ""dest-vsn"": ""1.1"", ^
    ""dest-args"": {""method"": ""post"", ""keep_alive_timeout"":10000}}"
```


### 2. Static configuration

## SMPP
```
[{rabbitmq_vshovel,
     [{vshovels,
         [{smpp_vshovel,
              [{sources,
                  [{brokers,
                      ["amqp://guest:guest@localhost:5672"]},
                   {arguments,
                      [{declarations,
                          [{'queue.declare',
                              [{queue, <<"deliver_sm">>}]}]}]}]},


                {destinations,
                  [
                    {protocol, smpp},
                    {arguments,
                    [
                        {host, {127,0,0,1}},
                        {port, 2775},
                        {password, <<"password">>},
                        {data_coding, 0},
                        {system_id, <<"smppclient1">>},
                        {interface_version, "3.4"},
                        {enquire_timeout, 30},
                        {submit_timeout, 60},
                        {system_type, ""},
                        {service_type, ""},
                        {addr_ton, 5},
                        {addr_npi, 0},
                        {source_addr, <<"123">>},
                        {source_addr_ton, 5},
                        {source_addr_npi, 0},
                        {dest_addr, <<"456">>},
                        {dest_addr_ton, 1},
                        {dest_addr_npi, 1},
                        {handler, rabbit_vshovel_endpoint_smpp},
                        {mode, transceiver},
                        {deliver_sm_exchange, <<"">>},
                        {deliver_sm_queue, <<"deliver_sm">>}]}]},

                {queue, <<"hello">>},
                {prefetch_count, 10},
                {reconnect_delay, 5}]}
          ]}]},
 {amqp_client,
     [{gen_server_call_timeout, 900000}]}].
```
## HTTP
```
[{rabbitmq_vshovel,
     [{vshovels,
         [{my_first_vshovel,
              [{sources,
                  [{brokers,
                      ["amqp://harry:secret@host1.domain/my_vhost",
                       "amqp://john:secret@host2.domain/my_vhost"]},
                   {arguments,
                      [{declarations,
                          [{'exchange.declare',
                              [{exchange, <<"my_fanout">>},
                               {type,      <<"fanout">>},
                                durable]},
                           {'queue.declare',
                              [{arguments,
                                    [{<<"x-message-ttl">>, long, 60000}]}]},
                           {'queue.bind',
                              [{exchange, <<"my_direct">>},
                               {queue,    <<>>}]}
                           ]}]}]},
              {destinations,
                  [{protocol,  http},
                   {address,   "https://56.12.331.123/server"},
                   {arguments, [{keepalive, 10},
                   {method,    post},
                   {ssl,       []}]}]},
              {queue,               <<>>},
              {prefetch_count,      10},
              {ack_mode,            on_confirm},
              {publish_properties,  [{delivery_mode, 2}]},
              {add_forward_headers, true},
              {publish_fields,      [{exchange,   <<"my_direct">>},
                                     {routing_key, <<"from_shovel">>}]},
              {reconnect_delay,     5}]}
          ]}
]}].

```

### 3. Verification

Once configured, verify status of the shovel using the following command (on a Unix-like OS):

```
rabbitmqctl eval 'rabbit_vshovel_status:status().'
```

and on Windows;

```
rabbitmqctl eval "rabbit_vshovel_status:status()."
```

## License and Copyright

(c) 84codes AB and Erlang Solutions Ltd. 2016-2017

https://www.cloudamqp.com
https://www.erlang-solutions.com/
