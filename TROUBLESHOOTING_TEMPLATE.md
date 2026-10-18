# Troubleshooting Case Template

Use this template when documenting a real technical problem.

The objective is to record the investigation accurately without presenting assumptions as confirmed causes.

---

# <Problem title>

## Problem

Describe what was expected to happen and what actually happened.

Avoid assuming the cause at this stage.

## Environment

Record the relevant environment.

Example:

```text
Operating system:
Kernel:
Desktop environment:
Display server:
Application:
Application version:
Hardware:
Relevant extension/service:
```

Only include information that matters to the case.

## Symptoms

Describe the observable behaviour.

Examples:

- Application fails to start
- Window is positioned incorrectly
- Service enters a failed state
- Network connection times out
- Disk reports degraded state
- Container continuously restarts

## Initial checks

Record the commands used to establish the state of the system.

Example:

```bash
systemctl --failed
journalctl -b -p warning
ip addr
ss -tulpn
df -h
free -h
```

Use only the commands relevant to the incident.

## Investigation

Document the troubleshooting process in chronological order.

For each important step, record:

1. What was checked
2. Why it was checked
3. What was observed

Do not describe an assumption as a confirmed root cause.

## Attempts that did not solve the problem

Document relevant approaches that were tested but did not resolve the issue.

Example:

```text
Attempt:
Result:
Why it was rejected:
```

Recording failed attempts can prevent the same troubleshooting path from being repeated later.

## Root cause

Only use this section when the cause was actually confirmed.

If the exact cause was not confirmed, use:

```text
The exact root cause was not conclusively identified.
```

Then describe the evidence that narrowed the problem down.

## Solution

Document the change that resolved the problem.

Example:

```bash
<command>
```

Explain only the configuration required to reproduce the solution.

## Verification

Confirm that the system behaves correctly after the change.

Example:

```bash
<verification command>
```

Record the observed result.

## Side effects

Document any functionality lost, changed or affected by the solution.

If none were observed:

```text
No relevant side effects were observed during testing.
```

## Final tested state

Example:

```text
Service: active
Application: working
Network connectivity: working
Errors: not observed during subsequent testing
```

Avoid claiming that a solution is universal when it was tested only in one environment.

## Useful commands

Collect the most relevant diagnostic commands used during the investigation.

```bash
<command 1>
<command 2>
<command 3>
```

## Security and privacy

Before publishing:

- Replace private IP addresses when they are not essential
- Remove usernames when unnecessary
- Never publish passwords, tokens or API keys
- Never publish private keys
- Remove public domain names when they expose private infrastructure
- Replace sensitive paths with placeholders when appropriate

Example:

```text
192.168.10.20
/home/<user>/
<container-name>
<domain.example>
```

## Lesson learned

Record the technical lesson from the incident.

Focus on what the troubleshooting process demonstrated rather than only the final command.

---

## Documentation principles

A good troubleshooting note should distinguish between:

```text
Observation
Hypothesis
Test
Result
Confirmed conclusion
```

A workaround is not necessarily a permanent fix.

If new evidence changes the conclusion later, update the documentation rather than preserving an inaccurate explanation.
