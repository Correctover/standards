# CCS Execution Semantics for LightAgent

## Overview

This document defines the precise execution semantics of CCS when integrated with LightAgent, covering all failure modes and their outcomes.

## Core Principle: Fail-Closed by Default

When CCS encounters any failure condition, the default action is to **block execution**. This is a structural guarantee, not a configuration option.

```python
# CCS guarantee
on_any_failure → BLOCK_EXECUTION
```

## Failure Modes and Execution Semantics

### 1. Governance Timeout

**Condition**: Governance check exceeds configured timeout
**Default Action**: Block execution
**Configurable**: Yes (timeout value only, not the outcome)

```python
@govern(timeout=5.0, on_failure="block")
def tool_execution():
    pass

# If governance takes >5.0s → BLOCKED
```

**Rationale**: Timeout indicates governance service is unhealthy or under attack. Allowing execution would violate security guarantees.

### 2. Governance Exception

**Condition**: Governance check raises any exception
**Default Action**: Block execution
**Configurable**: No (structural guarantee)

```python
# Any exception during governance → BLOCKED
try:
    decision = governance.check()
except Exception:
    return ExecutionResult.BLOCKED
```

**Rationale**: Exceptions indicate undefined behavior. Security-critical systems must fail-safe, not fail-open.

### 3. Governance Unreachable

**Condition**: Governance service is down or network is unavailable
**Default Action**: Block execution
**Configurable**: No (structural guarantee)

```python
# Connection failure → BLOCKED
try:
    decision = governance.check()
except ConnectionError:
    return ExecutionResult.BLOCKED
```

**Rationale**: Unreachable governance means no authorization can be verified. Proceeding would bypass security controls.

### 4. Governance Denied

**Condition**: Governance explicitly denies execution
**Default Action**: Block execution
**Configurable**: No (policy-driven)

```python
# Explicit denial → BLOCKED
if not decision.allowed:
    return ExecutionResult.BLOCKED
```

### 5. Governance Allowed

**Condition**: Governance explicitly allows execution
**Default Action**: Execute
**Configurable**: No (policy-driven)

```python
# Explicit approval → EXECUTED
if decision.allowed:
    return execute_tool()
```

### 6. Interceptor Exception

**Condition**: CCS interceptor itself raises exception
**Default Action**: Block execution
**Configurable**: No (structural guarantee)

```python
# CCS internal error → BLOCKED
try:
    decision = ccs_intercept()
except Exception:
    return ExecutionResult.BLOCKED
```

**Rationale**: CCS must never be the source of security bypass. Any internal failure must block execution.

## Execution State Machine

```
                    ┌─────────────┐
                    │   Start     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Invoke CCS │
                    │  Interceptor│
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
       ┌──────▼─────┐ ┌───▼────┐ ┌─────▼──────┐
       │  Timeout   │ │Denied  │ │ Exception  │
       └──────┬─────┘ └───┬────┘ └─────┬──────┘
              │            │            │
              └────────────┼────────────┘
                           │
                    ┌──────▼──────┐
                    │   BLOCKED   │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │    End      │
                    └─────────────┘

              ┌────────────────────┐
              │  Governance OK     │
              │  + Allowed         │
              └──────────┬─────────┘
                         │
                  ┌──────▼──────┐
                  │  EXECUTED   │
                  └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │    End      │
                  └─────────────┘
```

## Configuration Options

### Timeout

```python
@govern(timeout=5.0)  # 5 seconds, default 10.0s
```

### Policy Selection

```python
@govern(policy="strict")      # Most restrictive
@govern(policy="default")     # Standard policy
@govern(policy="permissive")  # Least restrictive (still fail-closed)
```

### Custom Failure Handler (Advanced)

```python
def on_failure(context):
    # Custom logging, alerting, etc.
    logger.error(f"Governance failure: {context.error}")
    # Must still return BLOCK
    return ExecutionResult.BLOCKED

@govern(on_failure=on_failure)
```

**Note**: Custom handlers can perform side effects (logging, alerting) but cannot override the BLOCK decision.

## Performance Characteristics

| Metric | Value | Notes |
|--------|-------|-------|
| P50 overhead | 0.13µs | Negligible |
| P99 overhead | 0.22µs | Negligible |
| Memory footprint | <1MB | Per interceptor instance |
| Cold start | <50ms | First invocation only |

## Thread Safety

CCS interceptors are thread-safe and can be used concurrently:

```python
# Safe for multi-threaded LightAgent
@govern(policy="default")
def shared_tool():
    pass

# Multiple threads can call simultaneously
thread1.start(shared_tool)
thread2.start(shared_tool)
thread3.start(shared_tool)
```

## Async/Sync Compatibility

CCS supports both synchronous and asynchronous execution:

### Synchronous
```python
@govern(policy="default")
def sync_tool():
    return "result"
```

### Asynchronous
```python
@govern(policy="default")
async def async_tool():
    return "result"
```

### Generator/Streaming
```python
@govern(policy="default")
def stream_tool():
    for chunk in data:
        yield chunk
```

## Integration Checklist

- [ ] Install correctover-ccs: `pip install correctover-ccs`
- [ ] Import governance decorator: `from ccs import govern`
- [ ] Apply decorator to tool execution hooks
- [ ] Configure timeout based on SLA (default 10.0s)
- [ ] Test failure modes with mock governance service
- [ ] Monitor performance metrics (expected <0.3µs overhead)
- [ ] Configure logging for failure events

## Conclusion

CCS provides deterministic, fail-closed execution semantics for LightAgent. Every failure mode results in BLOCKED execution, ensuring security guarantees are never compromised.
