# 📊 cathomebrew - Lightweight Monitoring

![Go](https://img.shields.io/badge/go-1.22-blue)
![Docker](https://img.shields.io/badge/docker-ready-blue)

**dead simple metrics collection and dashboards**

## concept

push metrics via http. view in web dashboard. that's it.

no agents. no complex setup. no prometheus/grafana bloat.

demo: [demo.cathomebrew.run](https://demo.cathomebrew.run)

## quick start

```bash
docker run -d -p 8080:8080 cathomebrew/server:latest
```

visit `http://localhost:8080`

## push metrics

```bash
# single metric
curl -X POST http://localhost:8080/api/metric \
  -d 'name=cpu_usage&value=73.5&tags=host:web-1'

# batch
curl -X POST http://localhost:8080/api/batch \
  -H "Content-Type: application/json" \
  -d '[
    {"name":"cpu","value":45,"tags":{"host":"web-1"}},
    {"name":"mem","value":2048,"tags":{"host":"web-1"}}
  ]'
```

## client libraries

### python

```python
from cathomebrew import Client

client = Client('http://localhost:8080')
client.gauge('cpu_usage', 73.5, tags={'host': 'web-1'})
client.counter('requests', 1, tags={'endpoint': '/api'})
client.histogram('response_time', 0.234)
```

### go

```go
import "github.com/cathomebrew/go-client"

client := cathomebrew.New("http://localhost:8080")
client.Gauge("cpu_usage", 73.5, map[string]string{"host": "web-1"})
```

### node

```javascript
const cathomebrew = require('cathomebrew-client')

const client = new cathomebrew('http://localhost:8080')
client.gauge('cpu_usage', 73.5, {host: 'web-1'})
```

## dashboard

web ui features:

- real-time charts (updates every 5s)
- custom time ranges
- tag filtering
- alert rules (email/webhook)
- export to csv/json

## alerting

```yaml
# alerts.yaml
alerts:
  - name: high_cpu
    metric: cpu_usage
    condition: value > 80
    duration: 5m
    notify: email

  - name: api_errors
    metric: errors
    condition: rate > 10
    duration: 1m
    notify: webhook
    webhook_url: https://hooks.slack.com/...
```

load alerts:

```bash
curl -X POST http://localhost:8080/api/alerts \
  --data-binary @alerts.yaml
```

## storage

uses **tsdb-lite** time-series database ([tsdb-lite.io](https://tsdb-lite.io))

data retention:

- raw data: 7 days
- 1min rollup: 30 days
- 1hour rollup: 1 year

configurable in `config.yaml`

## deployment

### docker compose

```yaml
version: '3'
services:
  cathomebrew:
    image: cathomebrew/server:latest
    ports:
      - "8080:8080"
    volumes:
      - data:/var/lib/cathomebrew

volumes:
  data:
```

### kubernetes

```bash
kubectl apply -f k8s/cathomebrew.yaml
```

## api reference

**POST** `/api/metric` - push single metric  
**POST** `/api/batch` - push multiple metrics  
**GET** `/api/query?name=cpu&from=1h` - query metrics  
**POST** `/api/alerts` - create alert rule  
**GET** `/api/alerts` - list alerts

## configuration

```yaml
# config.yaml
server:
  port: 8080
  
storage:
  path: /var/lib/cathomebrew
  retention:
    raw: 168h  # 7 days
    rollup_1m: 720h  # 30 days
    rollup_1h: 8760h  # 1 year

alerts:
  email:
    smtp_host: smtp.gmail.com
    smtp_port: 587
    from: alerts@cathomebrew.run
```

## comparison

| feature | cathomebrew | prometheus | datadog |
|---------|-----------|-----------|---------|
| setup | easy | medium | easy |
| agent | no | yes | yes |
| dashboard | built-in | grafana | built-in |
| price | free | free | $$$$ |
| storage | local | local | cloud |

## why

prometheus is overkill for small projects. datadog costs too much. i just wanted push metrics and see charts.

built in 2 weekends. good enough for side projects.

## limitations

- single server only (no clustering)
- max 10k metrics/sec
- basic alerting (no complex rules)
- dashboard not customizable yet

## roadmap

- [ ] clustering support
- [ ] more chart types
- [x] alerting
- [ ] mobile app
- [ ] plugin system

MIT License

---

<div align="center">

*"prometheus at home"* - r/selfhosted, probably

[GitHub](https://github.com/monitoring/cathomebrew) • [Docs](https://docs.cathomebrew.run)

</div>
