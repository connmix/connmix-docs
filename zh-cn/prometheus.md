# Prometheus 监控

Prometheus 是一个数据监控的解决方案，CONNMIX 内置了 Prometheus 的指标接口 `/metrics`，让我们能随时掌握系统运行的状态，快速定位问题和排除故障。

> 该功能只对企业版开放

## `center` 节点监控

在本节点 `apiServer` 的默认端口 `6787` 可以查看全部指标：

- `URL` http://127.0.0.1:6787/metrics

```
# HELP connmix_center_registry_channels_total Total number of registry channels
# TYPE connmix_center_registry_channels_total counter
connmix_center_registry_channels_total 2
# HELP connmix_center_registry_clients Number of registry clients
# TYPE connmix_center_registry_clients gauge
connmix_center_registry_clients 100
# HELP connmix_center_registry_engine_nodes Number of registry engine nodes
# TYPE connmix_center_registry_engine_nodes gauge
connmix_center_registry_engine_nodes 2
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 4.7792e-05
go_gc_duration_seconds{quantile="0.25"} 0.000196667
go_gc_duration_seconds{quantile="0.5"} 0.000382584
go_gc_duration_seconds{quantile="0.75"} 0.000477708
go_gc_duration_seconds{quantile="1"} 0.002006333
go_gc_duration_seconds_sum 0.009689124
go_gc_duration_seconds_count 22
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 29
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.17.5"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 4.306576e+06
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 1.7615544e+07
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 4819
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 148579
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 5.034368e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 4.306576e+06
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 6.447104e+06
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 5.152768e+06
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 47067
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 6.152192e+06
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 1.1599872e+07
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.676880689782481e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 195646
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 9600
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 16384
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 136816
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 147456
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 5.252704e+06
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 1.386173e+06
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 950272
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 950272
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 1.9139344e+07
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 14
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 1
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
```

## `engine` 节点监控

在本节点 `apiServer` 的默认端口 `6789` 可以查看全部指标：

- `URL` http://127.0.0.1:6789/metrics

```
# HELP connmix_engine_servers_clients Number of servers clients
# TYPE connmix_engine_servers_clients gauge
connmix_engine_servers_clients 100
# HELP connmix_engine_servers_read_messages_total Total number of servers read messages
# TYPE connmix_engine_servers_read_messages_total counter
connmix_engine_servers_read_messages_total{protocol="socket"} 0
connmix_engine_servers_read_messages_total{protocol="websocket"} 790599
# HELP connmix_engine_servers_send_duration_seconds A summary of the duration of servers send a message to client
# TYPE connmix_engine_servers_send_duration_seconds summary
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="forward",quantile="0"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="forward",quantile="0.25"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="forward",quantile="0.5"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="forward",quantile="0.75"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="forward",quantile="1"} NaN
connmix_engine_servers_send_duration_seconds_sum{protocol="socket",scenario="forward"} 0
connmix_engine_servers_send_duration_seconds_count{protocol="socket",scenario="forward"} 0
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="local",quantile="0"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="local",quantile="0.25"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="local",quantile="0.5"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="local",quantile="0.75"} NaN
connmix_engine_servers_send_duration_seconds{protocol="socket",scenario="local",quantile="1"} NaN
connmix_engine_servers_send_duration_seconds_sum{protocol="socket",scenario="local"} 0
connmix_engine_servers_send_duration_seconds_count{protocol="socket",scenario="local"} 0
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="forward",quantile="0"} NaN
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="forward",quantile="0.25"} NaN
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="forward",quantile="0.5"} NaN
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="forward",quantile="0.75"} NaN
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="forward",quantile="1"} NaN
connmix_engine_servers_send_duration_seconds_sum{protocol="websocket",scenario="forward"} 0
connmix_engine_servers_send_duration_seconds_count{protocol="websocket",scenario="forward"} 0
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="local",quantile="0"} 3.958e-06
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="local",quantile="0.25"} 0.000411041
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="local",quantile="0.5"} 0.0007815
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="local",quantile="0.75"} 0.001544833
connmix_engine_servers_send_duration_seconds{protocol="websocket",scenario="local",quantile="1"} 0.465281458
connmix_engine_servers_send_duration_seconds_sum{protocol="websocket",scenario="local"} 478.65057337999986
connmix_engine_servers_send_duration_seconds_count{protocol="websocket",scenario="local"} 47257
# HELP connmix_engine_servers_send_errors_total Total number of servers send errors
# TYPE connmix_engine_servers_send_errors_total counter
connmix_engine_servers_send_errors_total{protocol="socket"} 0
connmix_engine_servers_send_errors_total{protocol="websocket"} 0
# HELP connmix_engine_servers_send_messages_total Total number of servers send messages
# TYPE connmix_engine_servers_send_messages_total counter
connmix_engine_servers_send_messages_total{protocol="socket"} 0
connmix_engine_servers_send_messages_total{protocol="websocket"} 788801
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.9959e-05
go_gc_duration_seconds{quantile="0.25"} 0.000117667
go_gc_duration_seconds{quantile="0.5"} 0.000395416
go_gc_duration_seconds{quantile="0.75"} 0.001455083
go_gc_duration_seconds{quantile="1"} 0.006818834
go_gc_duration_seconds_sum 0.255898744
go_gc_duration_seconds_count 297
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 235
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.17.5"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 3.5784752e+07
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 3.760261536e+09
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 4819
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 3.0613714e+07
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 5.536944e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 3.5784752e+07
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 1.138688e+07
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 4.0812544e+07
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 89166
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 3.2768e+06
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 5.2199424e+07
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.676880692344567e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 3.070288e+07
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 9600
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 16384
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 436288
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 507904
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 3.9108304e+07
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 1.309581e+06
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 2.162688e+06
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 2.162688e+06
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 6.1737744e+07
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 20
# HELP promhttp_metric_handler_requests_in_flight Current number of scrapes being served.
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# HELP promhttp_metric_handler_requests_total Total number of scrapes by HTTP status code.
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 12
promhttp_metric_handler_requests_total{code="500"} 0
promhttp_metric_handler_requests_total{code="503"} 0
```
