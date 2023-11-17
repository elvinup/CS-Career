
To create an alert

```bash
curl -k -w '\n' -i --user 'prometheus:prometheus_pw' \
  --header 'Content-Type: application/json' \
  --data '[{"labels":{"instance":"sometesthost.xt.local", "alertname":"InstanceDown", "pagerduty":"yes", "severity":"critical"},"endsAt":"2023-11-18T00:10:53-03:00"}]' \
  [https://atl1s07prms01.xt.local:9093/api/v1/alerts](https://atl1s07prms01.xt.local:9093/api/v1/alerts)
```

To resolve the alert, just make endsAt, earlier than now. Also include startsAt parameter

```bash
curl -k -w '\n' -i --user 'prometheus:prometheus_pw' \
  --header 'Content-Type: application/json' \
  --data '[{"labels":{"instance":"sometesthost.xt.local", "alertname":"InstanceDown", "pagerduty":"yes", "severity":"critical"},"startsAt":"2023-11-16T00:10:53-03:00", "endsAt":"2023-11-17T00:10:53-03:00"}]' \
  [https://atl1s07prms01.xt.local:9093/api/v1/alerts](https://atl1s07prms01.xt.local:9093/api/v1/alerts)
```

