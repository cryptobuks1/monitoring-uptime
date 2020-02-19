# Pools uptime monitoring system.
Uptime monitoring system contains a two component: [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) as agent and [Prometheus](https://prometheus.io/) as server. Agents are placed on small VPS servers in three locations: Europe, Asia, USA. Configuratiion of agents contain input plugins for every monitoring pool. Two plugins are used:
* http_response for site and api endpoints probe;
* net_response for stratum endpoint tcp probe.
For one kind of each metric all input plugins contains a additional tags. This made a grouping metrics by labels and make a common endpoint status. The output uses the worst status of endpoint.

# Metrics
I'm showing only a important fields of each metric
* http_response_content_length{host="proxy-eu",location="cn",pool="btc.com",type="api"}
* http_response_http_response_code{host="proxy-eu",location="cn",pool="btc.com",type="api"}
* http_response_response_string_match{host="proxy-eu",location="cn",pool="btc.com",type="api"}
* http_response_response_time{host="proxy-eu",location="cn",pool="btc.com",type="api"}
* http_response_result_code{host="proxy-eu",location="cn",pool="btc.com",type="api"}

In consolidated graph metrics using the last metric - http_response_result_code. Her value is equal 0 when isn't access problems, and > 0 in all other cases.
* net_response_response_time{host="proxy-eu",location="br",pool="nicehash.com",port="3334",server="sha256.br.nicehash.com",type="stratum"}
* net_response_result_code{host="proxy-eu",location="br",pool="nicehash.com",port="3334",server="sha256.br.nicehash.com",type="stratum"}

In consolidated graph metrics using the last metric - net_response_result_code. Her behaviour similar a http_response_result_code.

# Computed metrics.
## Site and api metrics.
In graph using consolidated metrics. Consolidation doing by Prometheus server during a query metrics. These are the requests:
```
max(http_response_result_code{pool=~".+"}) by (pool, location, type)
```
We taking a max value of metric from all locations for viewing worstly value.

```
sum(count_over_time(http_response_result_code{pool =~ ".+", result="success"}[1d])) without (host, instance, job, method, result_type, server, status_code) / ignoring(result) group_left sum(count_over_time(http_response_result_code{pool =~ ".+"}[1d])) without (host, instance, job, method, result, result_type, server, status_code) * 100
```
Calculating a uptime percent by last 24 hours.

```
sum(count_over_time(http_response_result_code{pool =~ ".+", result="success"}[1d] offset 50000s)) without (host, instance, job, method, result_type, server, status_code) / ignoring(result) group_left sum(count_over_time(http_response_result_code{pool =~ ".+"}[1d] offset 50000s)) without (host, instance, job, method, result, result_type, server, status_code) * 100
```
Calculating a uptime percent from day start. Start position set as seconds offset in two places of query.

## Stratum metrics
Stratum metrics computed similar a http metrics.
```
max(net_response_result_code{pool=~".+"}) by (pool, location, type)

sum(count_over_time(net_response_result_code{type="stratum",result="success"}[1d])) without (port, server, host, instance, protocol, job, type) / ignoring(result) group_left sum(count_over_time(net_response_result_code{type="stratum"}[1d])) without (port, server, host, instance, protocol, job, type, result) * 100

sum(count_over_time(net_response_result_code{type="stratum",result="success"}[1d] offset 50000s)) without (port, server, host, instance, protocol, job, type) / ignoring(result) group_left sum(count_over_time(net_response_result_code{type="stratum"}[1d] offset 50000s)) without (port, server, host, instance, protocol, job, type, result) * 100
```
