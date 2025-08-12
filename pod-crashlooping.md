# Runbook: Analyzing a CrashLoopBackOff Pod in Kubernetes

## Overview
This runbook provides a step-by-step guide for an AI Agent to investigate and diagnose a pod in CrashLoopBackOff state in a Kubernetes cluster. CrashLoopBackOff occurs when a container repeatedly crashes and restarts. The agent should systematically gather information, identify root causes, and suggest resolutions. Use available tools to run `kubectl` commands or equivalent API queries for cluster interaction.

Key objectives:
- Identify the cause (e.g., OOM, misconfiguration, application error).
- Recommend fixes.

> **IMPORTANT FOR AI AGENTS:** This runbook is for **analysis and recommendation only**. You are in read-only investigation mode. DO NOT attempt to modify, patch, delete, or fix any resources. Your role is to diagnose the issue and provide specific remediation recommendations for human operators.

## Step 1: Confirm the Pod Status
- Run `kubectl get pods -n <namespace>` to list pods and confirm the status is CrashLoopBackOff.
- **Analysis**: Note the restart count. High restarts indicate a loop.
- **Expected Output**: Look for `STATUS: CrashLoopBackOff` and `RESTARTS: >0`.
- **If Issue**: If not in CrashLoopBackOff, this runbook may not apply—check for other states like Pending or Evicted.

## Step 2: Describe the Pod for Events and Details
- Execute `kubectl describe pod <pod-name> -n <namespace>`.
- **Analysis**:
  - Check the `Last State` section for termination reasons (e.g., "OOMKilled", "Error", exit code).
  - Review Events at the bottom for warnings like resource limits exceeded or image pull failures.
  - Note container specs: image, command, args, resources (requests/limits).
- **Common Causes**:
  - Exit code 1: Application error.
  - OOMKilled: Memory limit hit.
  - ImagePullBackOff: Invalid image.
- **Agent Tip**: Parse output for keywords like "OOM", "Error", "Back-off restarting failed container".

## Step 3: Inspect Pod Logs
- Run `kubectl logs <pod-name> -n <namespace> --previous` (for crashed container logs) or `kubectl logs <pod-name> -n <namespace>` for current.
- **Analysis**:
  - Look for error messages, stack traces, or exceptions in the application logs.
  - If no logs: Container might crash before logging (e.g., invalid command).
  - Multi-container pods: Specify `-c <container-name>`.
- **Agent Tip**: Correlate logs with describe output. Search for patterns like "out of memory" or "segmentation fault".

## Step 4: Check Resource Usage
- Use `kubectl top pod <pod-name> -n <namespace>` or query metrics API for CPU/memory usage.
- **Analysis**:
  - Compare usage against requests/limits in the pod spec.
  - High memory/CPU spikes suggest resource exhaustion.
- **Common Fixes**:
  - If OOM: Increase `resources.limits.memory` (but investigate app for leaks).
  - If CPU throttle: Adjust CPU limits/requests.
- **Agent Tip**: If no metrics, recommend enabling monitoring.

## Step 5: Review Deployment/Manifest Configuration
- Run `kubectl get deployment <deployment-name> -o yaml` (or equivalent for StatefulSet/DaemonSet).
- **Analysis**:
  - Inspect container: command/args (e.g., infinite loops), env vars, volumes, probes (liveness/readiness failing?).
  - Check for misconfigurations: Wrong image tag, missing secrets/configmaps.
  - Validate YAML syntax if custom.

## Step 6: Investigate External Dependencies
- Check related resources:
  - `kubectl get services,configmaps,secrets -n <namespace>`.
  - Test network: `kubectl exec` into another pod to curl services.
- **Analysis**: Crashes from unmet dependencies (e.g., DB connection fail).
- **Agent Tip**: Look for logs indicating connection refusals or timeouts.

## Step 7: Root Cause Summary and Recommendations
- **Summarize Findings**: Based on above, categorize cause (e.g., "Application bug: Infinite memory allocation").
- **Recommend Fixes**:
  - Code-level: Fix app logic (e.g., bound loops).
  - Config: Adjust resources, add probes (e.g., livenessProbe to detect hangs).
  - Rollout: `kubectl rollout restart deployment <name>`.
  - Example Patch: Use `kubectl patch` to update limits.
