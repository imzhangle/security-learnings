When **Gravitee.io API Gateway** (typically version 2.x or 3.x) is configured with **Elasticsearch**, it uses Elasticsearch as a centralized data store for various types of **observability and analytics data** generated by the gateway. This integration allows you to monitor, analyze, and troubleshoot your APIs effectively.

Here’s a breakdown of the **data stored in Elasticsearch** when integrated with Gravitee:

---

### ✅ 1. **API Request Logs / Access Logs**
Each API request flowing through the gateway can be logged into Elasticsearch.

**Fields typically include:**
- Timestamp (`timestamp`)
- API name and ID
- Application name and ID (the client app making the call)
- User information (if authenticated)
- HTTP method (`GET`, `POST`, etc.)
- Path/resource accessed
- Response time / latency
- Status code (`200`, `404`, `500`, etc.)
- Request size and response size
- IP addresses (client, server)
- Correlation ID (for tracing)
- Plan used (rate limiting plan)
- Gateway node ID (in cluster setups)

This data enables powerful analytics like:
- Traffic volume over time
- Error rates
- Latency trends
- Usage per API, application, or user

---

### ✅ 2. **Metrics / Analytics**
Aggregated metrics are sent periodically to Elasticsearch (depending on configuration).

These may include:
- Total requests
- Success/failure counts
- Average/median/max response time
- Bandwidth usage (in/out)
- Rate of requests per second

Used for dashboards in tools like Kibana or Grafana.

---

### ✅ 3. **Healthchecks**
If healthchecks are enabled, Elasticsearch may also contain:
- Health status of the gateway nodes
- Uptime statistics
- Resource usage (CPU, memory)

---

### ✅ 4. **Logs from Policies (Optional)**
Depending on your configuration and policies (like Logging Policy), additional debug or trace logs might be written to Elasticsearch.

For example:
- Request/response body logging (with caution due to sensitivity)
- Custom headers or metadata

---

### ✅ 5. **Audit Logs (if enabled)**
Gravitee can log internal management events (such as configuration changes via the admin UI or REST API). These audit logs can optionally be stored in Elasticsearch if configured.

Examples:
- Who created/deleted an API
- Configuration updates
- User login/logout

---

### 🧠 Where Does This Data Come From?
The gateway generates this data and sends it to Elasticsearch using:
- **Gravitee's Reporting Engine**
- A plugin/middleware component in the gateway
- Periodic batched writes (configurable interval)

---

### 🔍 How to View the Data?
You’ll often pair Elasticsearch with:
- **Kibana**: For visualization, dashboards, and querying
- **Grafana**: With Elasticsearch datasource for metrics dashboards
- **Custom tools**: Querying Elasticsearch directly

---

### 🛠️ Example Index Pattern in Elasticsearch
Gravitee usually creates daily indices like:
```
gravitee-YYYY.MM.DD
```

Each document corresponds to one request or metric.

Example mapping:
```json
{
  "@timestamp": "2025-04-05T10:00:00Z",
  "api": {
    "id": "api-12345",
    "name": "Weather API"
  },
  "application": {
    "id": "app-67890",
    "name": "MobileApp"
  },
  "http": {
    "method": "GET",
    "status": 200,
    "request": {
      "size": 256
    },
    "response": {
      "size": 1024
    }
  },
  "response_time": 45,
  "plan": "default-plan"
}
```

---

### 📌 Summary Table

| Data Type         | Description |
|------------------|-------------|
| Access Logs       | Full record of each API request |
| Metrics           | Aggregates for dashboards |
| Healthchecks      | Node-level system stats |
| Policy Logs       | Optional detailed policy logs |
| Audit Logs        | Admin/user actions (if enabled) |

---

### 📚 References
- [Gravitee.io Documentation - Monitoring](https://docs.gravitee.io/)
- [Gravitee + Elasticsearch Configuration Guide](https://docs.gravitee.io/gateway/gateway-configuration-logging-elasticsearch)

Let me know if you want help setting up Kibana dashboards or querying specific fields!
