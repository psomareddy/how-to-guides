# Enable Process Level Metrics with Open Telemetry **hostmetrics** receiver

## Open Telemetry Collector Configuration

Edit the /etc/otel/collector/splunk-otel-collector.conf file.

```cmd
sudo vi /etc/otel/collector/splunk-otel-collector.conf
```

1. Create a new receiver to scrape only the process metrics as we need to perform heavy filtering on this later and do not want the other metrics impacted
Copy the following snippet under "receivers:" section directly under the existing "hostmetrics:" entry.
    ```yaml
      hostmetrics/process_metrics:
        collection_interval: 10s
        scrapers:
          process:
            include:
              names: [java, node]
              match_type: regexp
            resource_attributes:
              process.owner:
                enabled: true
            metrics:
              process.memory.usage:
                enabled: true
            mute_process_name_error: true
            mute_process_exe_error: true
            mute_process_io_error: true
    ```

2. Reduce the number of process MTS (metric time series) by tracking only specific processes. Filtering out MTS for unwanted processes can have a tremendous impact on reducing the number of MTS. 
    
    ```yaml
      filter/filter_by_process_cmd:
        metrics:
          include:
            match_type: regexp
            metric_names:
              # only look at metrics that start with process
              - 'process.*'
            resource_attributes:
              - key: process.command_line
                #look for anything with anywhere in the command line value
                value: '(tomcat|node)'
    
      # Add a truncate command to make sure values like the Command Line are truncated below 256 so that o11y cloud can accept them          
      transform/truncate_metric_values:
        metric_statements:
          - context: resource
            statements:
              - truncate_all(attributes, 255)
    ```

3. We use a completely different pipeline for the metrics related to process as we want to filter only this data.
   Copy the following snippet under "service:pipelines:" section directly under "metrics:" pipeline.
    ```yaml
        metrics/process_metrics:
          receivers: [hostmetrics/process_metrics]
          processors: [memory_limiter, batch, resourcedetection, filter/filter_by_process_cmd, transform/truncate_metric_values]
          # this currently includes sending all metrics to the log file for testing/verification purposes
          exporters: [signalfx]
          # Use instead when sending to gateway
          #exporters: [otlp/gateway]      
    ```


## Restart Open Telemetry Collector

Restart the collector to apply the changes.

```cmd
sudo systemctl restart splunk-otel-collector
```
