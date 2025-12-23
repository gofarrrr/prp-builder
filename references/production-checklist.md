# Production Checklist

Deployment, monitoring, security, and cost control for production agent systems.

---

## Pre-Deployment Checklist

### Code Quality
- [ ] All tests pass (`pytest -v`)
- [ ] Linting clean (`ruff check`)
- [ ] Type checking passes (`mypy --strict`)
- [ ] No hardcoded secrets or API keys
- [ ] Dependencies pinned to specific versions

### Security Review
- [ ] Input validation on all external data
- [ ] Output sanitization where needed
- [ ] Rate limiting configured
- [ ] Authentication/authorization in place
- [ ] Audit logging enabled

### Configuration
- [ ] Environment variables documented
- [ ] Secrets in secure storage (not env files)
- [ ] Config validated at startup
- [ ] Fallback values for optional configs

### Documentation
- [ ] API documentation current
- [ ] Deployment runbook exists
- [ ] Rollback procedure documented
- [ ] Incident response contacts listed

---

## Monitoring & Observability

### Key Metrics to Track

```yaml
agent_metrics:
  performance:
    - response_latency_p50
    - response_latency_p95
    - response_latency_p99
    - tokens_per_request
    - requests_per_minute

  reliability:
    - success_rate
    - error_rate_by_type
    - timeout_rate
    - retry_rate

  cost:
    - cost_per_request
    - cost_per_user
    - daily_token_usage
    - monthly_spend_vs_budget

  quality:
    - user_satisfaction_score
    - task_completion_rate
    - escalation_rate
```

### Logging Strategy

```yaml
logging_levels:
  DEBUG:
    - Full prompt content (dev only)
    - Tool call parameters
    - Intermediate reasoning

  INFO:
    - Request start/end
    - Tool calls made
    - Task completion status

  WARNING:
    - Retry attempts
    - Rate limit approaches
    - Degraded mode activation

  ERROR:
    - Failed requests
    - API errors
    - Timeout events

structured_log_format:
  timestamp: ISO8601
  level: INFO
  request_id: uuid
  user_id: string
  action: string
  duration_ms: number
  token_count: number
  status: success|failure
  error_code: string (if failure)
```

### Alerting Rules

```yaml
alerts:
  critical:
    - name: "Error rate spike"
      condition: "error_rate > 5% for 5 minutes"
      action: "Page on-call"

    - name: "API key exhausted"
      condition: "remaining_quota < 1000"
      action: "Page on-call + auto-pause non-critical"

  warning:
    - name: "Latency degradation"
      condition: "p95_latency > 10s for 10 minutes"
      action: "Notify team"

    - name: "Cost anomaly"
      condition: "hourly_cost > 2x baseline"
      action: "Notify team + investigate"
```

---

## Security Considerations

### Input Security

```yaml
input_validation:
  # Sanitize user input
  - strip_html_tags: true
  - max_input_length: 10000
  - reject_known_injection_patterns: true

  # Validate file paths
  - no_path_traversal: true  # Reject ../
  - whitelist_extensions: [".py", ".js", ".md"]
  - require_within_project: true

  # Rate limiting
  - requests_per_minute: 60
  - requests_per_hour: 500
  - concurrent_requests: 5
```

### Output Security

```yaml
output_handling:
  # Prevent data leakage
  - filter_secrets: true  # Scan for API keys, passwords
  - redact_pii: true  # Personal identifiable info
  - sanitize_paths: true  # Remove absolute paths

  # Prevent code injection
  - escape_shell_commands: true
  - validate_generated_code: true
  - sandbox_code_execution: true
```

### API Key Management

```yaml
api_key_practices:
  storage:
    - use_secret_manager: true  # AWS Secrets, HashiCorp Vault
    - never_in_code: true
    - never_in_env_files_in_repo: true

  rotation:
    - rotate_every_days: 90
    - support_zero_downtime_rotation: true

  scope:
    - principle_of_least_privilege: true
    - separate_keys_per_environment: true
    - separate_keys_per_service: true
```

### Audit Logging

```yaml
audit_log:
  always_log:
    - user_id
    - timestamp
    - action_type
    - resource_accessed
    - outcome

  sensitive_actions:
    - file_modifications
    - external_api_calls
    - data_exports
    - configuration_changes

  retention:
    - duration_days: 90
    - compliance_requirements: [GDPR, SOC2]
```

---

## Cost Control

### Token Budget Management

```yaml
cost_controls:
  per_request:
    max_input_tokens: 50000
    max_output_tokens: 4000
    max_total_tokens: 100000

  per_user:
    daily_token_limit: 1000000
    monthly_token_limit: 10000000

  per_operation:
    simple_query: 5000
    complex_task: 50000
    research_task: 100000
```

### Cost Optimization Strategies

```markdown
## Cost Reduction Tactics

### 1. Model Selection
- Use cheaper models for simple tasks
- Reserve expensive models for complex reasoning
- Example: Use Haiku for classification, Opus for generation

### 2. Caching
- Cache identical queries
- Cache tool outputs
- Cache intermediate results
- TTL: Balance freshness vs cost

### 3. Prompt Optimization
- Compress verbose instructions
- Use references instead of full content
- Remove redundant context
- Measure tokens per section

### 4. Request Batching
- Batch similar requests
- Use parallel processing efficiently
- Avoid redundant API calls

### 5. Early Termination
- Stop when answer is clear
- Don't over-generate
- Set appropriate max_tokens
```

### Cost Monitoring Dashboard

```yaml
cost_dashboard:
  real_time:
    - current_hour_spend
    - current_day_spend
    - budget_remaining

  trends:
    - cost_per_day_7d
    - cost_per_user_7d
    - tokens_per_request_7d

  breakdowns:
    - by_model
    - by_operation_type
    - by_user_segment
    - by_feature
```

---

## Rate Limiting

### Implementation Patterns

```yaml
rate_limits:
  global:
    requests_per_second: 100
    requests_per_minute: 1000

  per_user:
    requests_per_minute: 30
    requests_per_hour: 200
    concurrent_requests: 3

  per_operation:
    expensive_operations:
      requests_per_minute: 5
    standard_operations:
      requests_per_minute: 30
```

### Graceful Degradation

```yaml
degradation_tiers:
  normal:
    all_features: enabled
    model: claude-3-opus

  degraded_1:
    trigger: "API latency > 5s"
    action: "Switch to sonnet model"

  degraded_2:
    trigger: "Error rate > 10%"
    action: "Disable non-critical features"

  degraded_3:
    trigger: "Rate limited by provider"
    action: "Queue requests, show waiting message"

  maintenance:
    trigger: "Manual activation"
    action: "Read-only mode, cached responses only"
```

---

## Rollback Strategies

### Quick Rollback

```bash
# Immediate rollback commands
git revert HEAD --no-edit
kubectl rollout undo deployment/agent-service
docker-compose down && docker-compose up -d --build
```

### Staged Rollback

```yaml
rollback_procedure:
  1_detect:
    - Monitor error rate
    - Check user reports
    - Verify metrics anomalies

  2_decide:
    - Severity assessment
    - Rollback vs hotfix decision
    - Communication plan

  3_execute:
    - Notify stakeholders
    - Execute rollback command
    - Verify old version running

  4_verify:
    - Check error rates normalized
    - Verify functionality
    - Monitor for 30 minutes

  5_post_mortem:
    - Document incident
    - Identify root cause
    - Plan prevention
```

---

## Health Checks

### Endpoint Design

```yaml
health_endpoints:
  /health:
    purpose: "Basic liveness"
    checks:
      - process_running
    response_time: "<100ms"

  /health/ready:
    purpose: "Readiness for traffic"
    checks:
      - database_connected
      - cache_available
      - api_key_valid
    response_time: "<500ms"

  /health/deep:
    purpose: "Full system check"
    checks:
      - all_dependencies_healthy
      - recent_request_success_rate
      - token_quota_remaining
    response_time: "<2s"
    frequency: "every 5 minutes"
```

### Dependency Health

```yaml
dependency_checks:
  llm_api:
    check: "test request with minimal tokens"
    timeout: 5s
    critical: true

  database:
    check: "SELECT 1"
    timeout: 1s
    critical: true

  cache:
    check: "PING"
    timeout: 500ms
    critical: false

  external_apis:
    check: "HEAD request"
    timeout: 2s
    critical: false
```

---

## Production Checklist for PRPs

When PRP involves production deployment:

```markdown
## Deployment Section

### Pre-Deploy
- [ ] All validation levels pass
- [ ] Security review completed
- [ ] Rollback plan documented
- [ ] Monitoring dashboards ready

### Deploy
- [ ] Deploy to staging first
- [ ] Run smoke tests
- [ ] Gradual rollout (canary)
- [ ] Monitor error rates

### Post-Deploy
- [ ] Verify health checks
- [ ] Check key metrics
- [ ] Confirm no alerts
- [ ] Document deployment

### If Issues
- [ ] Execute rollback
- [ ] Notify stakeholders
- [ ] Investigate root cause
- [ ] Schedule post-mortem
```

---

## Quick Reference

### Critical Limits

| Metric | Warning | Critical |
|--------|---------|----------|
| Error rate | >2% | >5% |
| P95 latency | >5s | >10s |
| Budget usage | >80% | >95% |
| Token quota | <20% | <5% |

### Emergency Contacts

```yaml
escalation:
  level_1: "On-call engineer via PagerDuty"
  level_2: "Engineering manager"
  level_3: "VP Engineering"

  api_issues: "LLM provider support"
  security: "Security team + legal"
```
