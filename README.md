# kubectl-curl-minimal

Kubectl plugin to run curl commands against kubernetes pods (minimal version)

## Motivation

Focus on this fork is the possibility to query Prometheus metrics and the
Prometheus database via the API. Usually this database is running in a POD
of a Kubernetes cluster and without exposed Ingress this service is not
reachable from outside the cluster. For a faster access such kind of data,
`kubectl curl` could help.

## Installation

If `$GOPATH/bin` is in the `PATH`, the plugin can be installed with:
```
$ go install github.com/eumel8/kubectl-curl@latest
```

If it was installed properly, it will be visibile when listing kubectl plugins:
```
$ kubectl plugin list
The following compatible plugins are available:

/.../kubectl-curl
```

## Usage

```
kubectl curl [options] URL [container]
```

* In the URL, the host part must be the name of the pod to send the request to.
* If no port number is specified, the request will be sent to a `http` port.
* If there are multiple containers with a `http` port, the name of the container
  to send to the request to must be specified after the URL.

## Examples

```bash
$ kubectl curl -n cattle-monitoring-system http://prometheus-rancher-monitoring-prometheus-0:9090/api/v1/query?query=kube_pod_container_info | jq '.data.result[].metric|select(.namespace=="demoapp")|.pod'

"alertmanager-project-monitoring-alertmanager-0"
"prometheus-project-monitoring-prometheus-0"
"alertmanager-project-monitoring-alertmanager-0"
"curl-client-0"
"demoapp-57bf45f76-bgkwb"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"prometheus-project-monitoring-prometheus-0"
```

```bash
$ kubectl curl -n cattle-monitoring-system http://prometheus-rancher-monitoring-prometheus-0:9090/api/v1/query?query=container_memory_usage_bytes | jq '.data.result[]|select(.metric.namespace=="demoapp")|.metric.pod,.value[1]'
"alertmanager-project-monitoring-alertmanager-0"
"23601152"
"alertmanager-project-monitoring-alertmanager-0"
"4415488"
"prometheus-project-monitoring-prometheus-0"
"6987776"
"curl-client-0"
"3043328"
"demoapp-57bf45f76-bgkwb"
"3813376"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"59322368"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"5423104"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"1069416448"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"70479872"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"65568768"
"prometheus-project-monitoring-prometheus-0"
"215195648"
"alertmanager-project-monitoring-alertmanager-0"
"28397568"
"alertmanager-project-monitoring-alertmanager-0"
"344064"
"demoapp-57bf45f76-bgkwb"
"4059136"
"demoapp-57bf45f76-bgkwb"
"208896"
"prometheus-project-monitoring-prometheus-0"
"223219712"
"prometheus-project-monitoring-prometheus-0"
"204800"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"1270480896"
"project-monitoring-grafana-8d98ffcbb-q7wc5"
"204800"
"curl-client-0"
"3284992"
"curl-client-0"
"217088"
```

```bash
$ kubectl curl -n cattle-monitoring-system http://prometheus-rancher-monitoring-prometheus-0:9090/api/v1/query?query=fluentbit_uptime | jq .
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "__name__": "fluentbit_uptime",
          "container": "fluent-bit",
          "endpoint": "http-metrics",
          "instance": "10.42.0.167:2020",
          "job": "logging-example-fluentbit-monitor",
          "namespace": "kube-logging",
          "pod": "logging-example-fluentbit-97gjj",
          "service": "logging-example-fluentbit-monitor"
        },
        "value": [
          1693512106.679,
          "1976"
        ]
      },
      {
        "metric": {
          "__name__": "fluentbit_uptime",
          "container": "fluent-bit",
          "endpoint": "http-metrics",
          "instance": "10.42.1.127:2020",
          "job": "logging-example-fluentbit-monitor",
          "namespace": "kube-logging",
          "pod": "logging-example-fluentbit-m6w6q",
          "service": "logging-example-fluentbit-monitor"
        },
        "value": [
          1693512106.679,
          "1982"
        ]
      }
    ]
  }
}
```

```bash
$ kubectl curl -n kube-logging http://kube-logging-logging-operator-f874c54f8-wcd6g:8080/metrics
# HELP certwatcher_read_certificate_errors_total Total number of certificate read errors
# TYPE certwatcher_read_certificate_errors_total counter
certwatcher_read_certificate_errors_total 0
# HELP certwatcher_read_certificate_total Total number of certificate reads
# TYPE certwatcher_read_certificate_total counter
certwatcher_read_certificate_total 0
# HELP controller_runtime_active_workers Number of currently used workers per controller
# TYPE controller_runtime_active_workers gauge
controller_runtime_active_workers{controller="eventtailer"} 0
controller_runtime_active_workers{controller="hosttailer"} 0
controller_runtime_active_workers{controller="logging"} 0
# HELP controller_runtime_max_concurrent_reconciles Maximum number of concurrent reconciles per controller
# TYPE controller_runtime_max_concurrent_reconciles gauge
controller_runtime_max_concurrent_reconciles{controller="eventtailer"} 1
controller_runtime_max_concurrent_reconciles{controller="hosttailer"} 1
controller_runtime_max_concurrent_reconciles{controller="logging"} 1
# HELP controller_runtime_reconcile_errors_total Total number of reconciliation errors per controller
# TYPE controller_runtime_reconcile_errors_total counter
controller_runtime_reconcile_errors_total{controller="eventtailer"} 0
controller_runtime_reconcile_errors_total{controller="hosttailer"} 0
controller_runtime_reconcile_errors_total{controller="logging"} 20
# HELP controller_runtime_reconcile_time_seconds Length of time per reconciliation per controller
# TYPE controller_runtime_reconcile_time_seconds histogram
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.005"} 9
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.01"} 11
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.025"} 18
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.05"} 28
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.1"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.15"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.2"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.25"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.3"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.35"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.4"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.45"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.5"} 30
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.6"} 31
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.7"} 31
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.8"} 31
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="0.9"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="1"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="1.25"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="1.5"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="1.75"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="2"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="2.5"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="3"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="3.5"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="4"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="4.5"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="5"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="6"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="7"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="8"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="9"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="10"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="15"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="20"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="25"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="30"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="40"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="50"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="60"} 33
controller_runtime_reconcile_time_seconds_bucket{controller="logging",le="+Inf"} 33
controller_runtime_reconcile_time_seconds_sum{controller="logging"} 2.887466626
controller_runtime_reconcile_time_seconds_count{controller="logging"} 33
# HELP controller_runtime_reconcile_total Total number of reconciliations per controller
# TYPE controller_runtime_reconcile_total counter
controller_runtime_reconcile_total{controller="eventtailer",result="error"} 0
controller_runtime_reconcile_total{controller="eventtailer",result="requeue"} 0
controller_runtime_reconcile_total{controller="eventtailer",result="requeue_after"} 0
controller_runtime_reconcile_total{controller="eventtailer",result="success"} 0
controller_runtime_reconcile_total{controller="hosttailer",result="error"} 0
controller_runtime_reconcile_total{controller="hosttailer",result="requeue"} 0
controller_runtime_reconcile_total{controller="hosttailer",result="requeue_after"} 0
controller_runtime_reconcile_total{controller="hosttailer",result="success"} 0
controller_runtime_reconcile_total{controller="logging",result="error"} 20
controller_runtime_reconcile_total{controller="logging",result="requeue"} 1
controller_runtime_reconcile_total{controller="logging",result="requeue_after"} 6
controller_runtime_reconcile_total{controller="logging",result="success"} 6
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 2.0933e-05
go_gc_duration_seconds{quantile="0.25"} 6.5873e-05
go_gc_duration_seconds{quantile="0.5"} 8.5912e-05
go_gc_duration_seconds{quantile="0.75"} 0.000176668
go_gc_duration_seconds{quantile="1"} 0.004291975
go_gc_duration_seconds_sum 0.011985725
go_gc_duration_seconds_count 35
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 314
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.20.6"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.
# TYPE go_memstats_alloc_bytes gauge
go_memstats_alloc_bytes 2.261408e+07
# HELP go_memstats_alloc_bytes_total Total number of bytes allocated, even if freed.
# TYPE go_memstats_alloc_bytes_total counter
go_memstats_alloc_bytes_total 3.19551592e+08
# HELP go_memstats_buck_hash_sys_bytes Number of bytes used by the profiling bucket hash table.
# TYPE go_memstats_buck_hash_sys_bytes gauge
go_memstats_buck_hash_sys_bytes 1.575384e+06
# HELP go_memstats_frees_total Total number of frees.
# TYPE go_memstats_frees_total counter
go_memstats_frees_total 2.433461e+06
# HELP go_memstats_gc_sys_bytes Number of bytes used for garbage collection system metadata.
# TYPE go_memstats_gc_sys_bytes gauge
go_memstats_gc_sys_bytes 9.766672e+06
# HELP go_memstats_heap_alloc_bytes Number of heap bytes allocated and still in use.
# TYPE go_memstats_heap_alloc_bytes gauge
go_memstats_heap_alloc_bytes 2.261408e+07
# HELP go_memstats_heap_idle_bytes Number of heap bytes waiting to be used.
# TYPE go_memstats_heap_idle_bytes gauge
go_memstats_heap_idle_bytes 2.015232e+07
# HELP go_memstats_heap_inuse_bytes Number of heap bytes that are in use.
# TYPE go_memstats_heap_inuse_bytes gauge
go_memstats_heap_inuse_bytes 3.2473088e+07
# HELP go_memstats_heap_objects Number of allocated objects.
# TYPE go_memstats_heap_objects gauge
go_memstats_heap_objects 97714
# HELP go_memstats_heap_released_bytes Number of heap bytes released to OS.
# TYPE go_memstats_heap_released_bytes gauge
go_memstats_heap_released_bytes 1.6449536e+07
# HELP go_memstats_heap_sys_bytes Number of heap bytes obtained from system.
# TYPE go_memstats_heap_sys_bytes gauge
go_memstats_heap_sys_bytes 5.2625408e+07
# HELP go_memstats_last_gc_time_seconds Number of seconds since 1970 of last garbage collection.
# TYPE go_memstats_last_gc_time_seconds gauge
go_memstats_last_gc_time_seconds 1.6935122425080535e+09
# HELP go_memstats_lookups_total Total number of pointer lookups.
# TYPE go_memstats_lookups_total counter
go_memstats_lookups_total 0
# HELP go_memstats_mallocs_total Total number of mallocs.
# TYPE go_memstats_mallocs_total counter
go_memstats_mallocs_total 2.531175e+06
# HELP go_memstats_mcache_inuse_bytes Number of bytes in use by mcache structures.
# TYPE go_memstats_mcache_inuse_bytes gauge
go_memstats_mcache_inuse_bytes 4800
# HELP go_memstats_mcache_sys_bytes Number of bytes used for mcache structures obtained from system.
# TYPE go_memstats_mcache_sys_bytes gauge
go_memstats_mcache_sys_bytes 15600
# HELP go_memstats_mspan_inuse_bytes Number of bytes in use by mspan structures.
# TYPE go_memstats_mspan_inuse_bytes gauge
go_memstats_mspan_inuse_bytes 407040
# HELP go_memstats_mspan_sys_bytes Number of bytes used for mspan structures obtained from system.
# TYPE go_memstats_mspan_sys_bytes gauge
go_memstats_mspan_sys_bytes 652800
# HELP go_memstats_next_gc_bytes Number of heap bytes when next garbage collection will take place.
# TYPE go_memstats_next_gc_bytes gauge
go_memstats_next_gc_bytes 4.5846456e+07
# HELP go_memstats_other_sys_bytes Number of bytes used for other system allocations.
# TYPE go_memstats_other_sys_bytes gauge
go_memstats_other_sys_bytes 897328
# HELP go_memstats_stack_inuse_bytes Number of bytes in use by the stack allocator.
# TYPE go_memstats_stack_inuse_bytes gauge
go_memstats_stack_inuse_bytes 1.900544e+06
# HELP go_memstats_stack_sys_bytes Number of bytes obtained from system for stack allocator.
# TYPE go_memstats_stack_sys_bytes gauge
go_memstats_stack_sys_bytes 1.900544e+06
# HELP go_memstats_sys_bytes Number of bytes obtained from system.
# TYPE go_memstats_sys_bytes gauge
go_memstats_sys_bytes 6.7433736e+07
# HELP go_threads Number of OS threads created.
# TYPE go_threads gauge
go_threads 9
# HELP leader_election_master_status Gauge of if the reporting system is master of the relevant lease, 0 indicates backup, 1 indicates master. 'name' is the string used to identify the lease. Please make sure to group by name.
# TYPE leader_election_master_status gauge
leader_election_master_status{name="logging-operator.logging.banzaicloud.io"} 1
# HELP logging_resource_problems
# TYPE logging_resource_problems gauge
logging_resource_problems{kind="ClusterFlow",name="allout",namespace="cattle-logging-system"} 0
logging_resource_problems{kind="ClusterOutput",name="allout",namespace="cattle-logging-system"} 0
logging_resource_problems{kind="ClusterOutput",name="eaas",namespace="cattle-logging-system"} 0
# HELP logging_resource_state
# TYPE logging_resource_state gauge
logging_resource_state{kind="ClusterFlow",name="allout",namespace="cattle-logging-system",status="active"} 1
logging_resource_state{kind="ClusterFlow",name="allout",namespace="cattle-logging-system",status="inactive"} 0
logging_resource_state{kind="ClusterOutput",name="allout",namespace="cattle-logging-system",status="active"} 1
logging_resource_state{kind="ClusterOutput",name="allout",namespace="cattle-logging-system",status="inactive"} 0
logging_resource_state{kind="ClusterOutput",name="eaas",namespace="cattle-logging-system",status="active"} 0
logging_resource_state{kind="ClusterOutput",name="eaas",namespace="cattle-logging-system",status="inactive"} 1
# HELP process_cpu_seconds_total Total user and system CPU time spent in seconds.
# TYPE process_cpu_seconds_total counter
process_cpu_seconds_total 5.22
# HELP process_max_fds Maximum number of open file descriptors.
# TYPE process_max_fds gauge
process_max_fds 1.048576e+06
# HELP process_open_fds Number of open file descriptors.
# TYPE process_open_fds gauge
process_open_fds 11
# HELP process_resident_memory_bytes Resident memory size in bytes.
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 6.6342912e+07
# HELP process_start_time_seconds Start time of the process since unix epoch in seconds.
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.69351003211e+09
# HELP process_virtual_memory_bytes Virtual memory size in bytes.
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 7.71555328e+08
# HELP process_virtual_memory_max_bytes Maximum amount of virtual memory available in bytes.
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes 1.8446744073709552e+19
# HELP rest_client_requests_total Number of HTTP requests, partitioned by status code, method, and host.
# TYPE rest_client_requests_total counter
rest_client_requests_total{code="200",host="10.43.0.1:443",method="DELETE"} 13
rest_client_requests_total{code="200",host="10.43.0.1:443",method="GET"} 1283
rest_client_requests_total{code="200",host="10.43.0.1:443",method="PATCH"} 14
rest_client_requests_total{code="200",host="10.43.0.1:443",method="PUT"} 1097
rest_client_requests_total{code="201",host="10.43.0.1:443",method="POST"} 28
rest_client_requests_total{code="409",host="10.43.0.1:443",method="POST"} 18
rest_client_requests_total{code="422",host="10.43.0.1:443",method="PUT"} 8
# HELP workqueue_adds_total Total number of adds handled by workqueue
# TYPE workqueue_adds_total counter
workqueue_adds_total{name="eventtailer"} 0
workqueue_adds_total{name="hosttailer"} 0
workqueue_adds_total{name="logging"} 33
# HELP workqueue_depth Current depth of workqueue
# TYPE workqueue_depth gauge
workqueue_depth{name="eventtailer"} 0
workqueue_depth{name="hosttailer"} 0
workqueue_depth{name="logging"} 0
# HELP workqueue_longest_running_processor_seconds How many seconds has the longest running processor for workqueue been running.
# TYPE workqueue_longest_running_processor_seconds gauge
workqueue_longest_running_processor_seconds{name="eventtailer"} 0
workqueue_longest_running_processor_seconds{name="hosttailer"} 0
workqueue_longest_running_processor_seconds{name="logging"} 0
# HELP workqueue_queue_duration_seconds How long in seconds an item stays in workqueue before being requested
# TYPE workqueue_queue_duration_seconds histogram
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="1e-08"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="1e-07"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="1e-06"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="9.999999999999999e-06"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="9.999999999999999e-05"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="0.001"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="0.01"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="0.1"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="1"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="10"} 0
workqueue_queue_duration_seconds_bucket{name="eventtailer",le="+Inf"} 0
workqueue_queue_duration_seconds_sum{name="eventtailer"} 0
workqueue_queue_duration_seconds_count{name="eventtailer"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="1e-08"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="1e-07"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="1e-06"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="9.999999999999999e-06"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="9.999999999999999e-05"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="0.001"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="0.01"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="0.1"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="1"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="10"} 0
workqueue_queue_duration_seconds_bucket{name="hosttailer",le="+Inf"} 0
workqueue_queue_duration_seconds_sum{name="hosttailer"} 0
workqueue_queue_duration_seconds_count{name="hosttailer"} 0
workqueue_queue_duration_seconds_bucket{name="logging",le="1e-08"} 0
workqueue_queue_duration_seconds_bucket{name="logging",le="1e-07"} 0
workqueue_queue_duration_seconds_bucket{name="logging",le="1e-06"} 0
workqueue_queue_duration_seconds_bucket{name="logging",le="9.999999999999999e-06"} 16
workqueue_queue_duration_seconds_bucket{name="logging",le="9.999999999999999e-05"} 21
workqueue_queue_duration_seconds_bucket{name="logging",le="0.001"} 21
workqueue_queue_duration_seconds_bucket{name="logging",le="0.01"} 22
workqueue_queue_duration_seconds_bucket{name="logging",le="0.1"} 28
workqueue_queue_duration_seconds_bucket{name="logging",le="1"} 33
workqueue_queue_duration_seconds_bucket{name="logging",le="10"} 33
workqueue_queue_duration_seconds_bucket{name="logging",le="+Inf"} 33
workqueue_queue_duration_seconds_sum{name="logging"} 2.513555746999999
workqueue_queue_duration_seconds_count{name="logging"} 33
# HELP workqueue_retries_total Total number of retries handled by workqueue
# TYPE workqueue_retries_total counter
workqueue_retries_total{name="eventtailer"} 0
workqueue_retries_total{name="hosttailer"} 0
workqueue_retries_total{name="logging"} 27
# HELP workqueue_unfinished_work_seconds How many seconds of work has been done that is in progress and hasn't been observed by work_duration. Large values indicate stuck threads. One can deduce the number of stuck threads by observing the rate at which this increases.
# TYPE workqueue_unfinished_work_seconds gauge
workqueue_unfinished_work_seconds{name="eventtailer"} 0
workqueue_unfinished_work_seconds{name="hosttailer"} 0
workqueue_unfinished_work_seconds{name="logging"} 0
# HELP workqueue_work_duration_seconds How long in seconds processing an item from workqueue takes.
# TYPE workqueue_work_duration_seconds histogram
workqueue_work_duration_seconds_bucket{name="eventtailer",le="1e-08"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="1e-07"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="1e-06"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="9.999999999999999e-06"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="9.999999999999999e-05"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="0.001"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="0.01"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="0.1"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="1"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="10"} 0
workqueue_work_duration_seconds_bucket{name="eventtailer",le="+Inf"} 0
workqueue_work_duration_seconds_sum{name="eventtailer"} 0
workqueue_work_duration_seconds_count{name="eventtailer"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="1e-08"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="1e-07"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="1e-06"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="9.999999999999999e-06"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="9.999999999999999e-05"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="0.001"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="0.01"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="0.1"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="1"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="10"} 0
workqueue_work_duration_seconds_bucket{name="hosttailer",le="+Inf"} 0
workqueue_work_duration_seconds_sum{name="hosttailer"} 0
workqueue_work_duration_seconds_count{name="hosttailer"} 0
workqueue_work_duration_seconds_bucket{name="logging",le="1e-08"} 0
workqueue_work_duration_seconds_bucket{name="logging",le="1e-07"} 0
workqueue_work_duration_seconds_bucket{name="logging",le="1e-06"} 0
workqueue_work_duration_seconds_bucket{name="logging",le="9.999999999999999e-06"} 0
workqueue_work_duration_seconds_bucket{name="logging",le="9.999999999999999e-05"} 0
workqueue_work_duration_seconds_bucket{name="logging",le="0.001"} 0
workqueue_work_duration_seconds_bucket{name="logging",le="0.01"} 11
workqueue_work_duration_seconds_bucket{name="logging",le="0.1"} 30
workqueue_work_duration_seconds_bucket{name="logging",le="1"} 33
workqueue_work_duration_seconds_bucket{name="logging",le="10"} 33
workqueue_work_duration_seconds_bucket{name="logging",le="+Inf"} 33
workqueue_work_duration_seconds_sum{name="logging"} 2.8878778179999998
workqueue_work_duration_seconds_count{name="logging"} 33
```
