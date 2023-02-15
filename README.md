# docker-compose project for TIG stack.

## Usage

Set your desired values in .env.
```bash
# influxdb/telegraf
DOCKER_INFLUXDB_INIT_MODE=setup
DOCKER_INFLUXDB_INIT_USERNAME=admin
DOCKER_INFLUXDB_INIT_PASSWORD=changeme
DOCKER_INFLUXDB_INIT_ORG=tig_tutorial
DOCKER_INFLUXDB_INIT_BUCKET=tutorial
DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=test-bucket-token
DOCKER_INFLUXDB_INIT_RETENTION=30d
```

## Bring up the stack

```bash
docker compose up -d
```

Per default:
*   influxdb-v2 will be exposed at port 8086 and have a preconfigured bucket according to the "DOCKER_INFLUXDB_INIT_BUCKET" variable.
*   telegraf will have the "inputs_dns" as a preconfigured input which gathers dns query times in miliseconds - like Dig.
*   grafana will be exposed at port 3000 with anonymous login enabled i.e no login required.

## Add influxdb-v2 as a datasource in grafana
*   Head over to http://localhost:3000
*   Click gear icon -> Data sources -> Add data source
*   Select InfluxDB
    *   Name: friendly name
    *   Query Language: Flux
    *   URL: http://influxdb-v2:8086
    *   turn off Basic auth
    *   Organization: the value of DOCKER_INFLUXDB_INIT_ORG in .env
    *   Token: the value of DOCKER_INFLUXDB_INIT_ADMIN_TOKEN in .env
    *   Default bucket: the value of DOCKER_INFLUXDB_INIT_BUCKET in .env
    *   Save & test: You should see a message "datasource is working."

## Test our TIG stack by performing a flux query
*   Click compass icon -> Explore
*   Make sure the influxdb database is selected in the dropdown.
*   Paste the query below where "tutorial" represents your value of DOCKER_INFLUXDB_INIT_BUCKET in .env
    ```bash
    from(bucket: "tutorial")
      |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
      |> filter(fn: (r) => r["_measurement"] == "dns_query")
      |> filter(fn: (r) => r["_field"] == "query_time_ms")
      |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
      |> yield(name: "mean")
    ```