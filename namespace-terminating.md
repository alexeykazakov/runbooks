# namespace-terminating

## Overview

The `namespace-terminating` alert is triggered when a namespace is stuck in the `terminating` phase for an excessive duration.

## AI-Friendly Troubleshooting Steps

> This runbook is designed for both human administrators and AI systems (LLMs) to follow step-by-step. Each step can be executed by an LLM using appropriate tools or APIs. Please ensure all actions are logged and errors are reported.

### 1. Retrieve Namespace Status
- **Action:** Get the full YAML status of the namespace.
- **Command:**
  ```shell
  oc get ns/<name> -o yaml
  ```
- **AI Note:** Replace `<name>` with the namespace name from the alert. Parse the output for `status.conditions` and `finalizers` fields.

### 2. Identify Blocking Finalizers
- **Action:** Check if any resources in the namespace have custom finalizers that prevent deletion.
- **AI Note:** For each resource type in the namespace, list resources and check for non-empty `finalizers` in metadata. Example resource types: pods, services, configmaps, etc.

### 3. Operator Finalizer Handling
- **Action:** If a finalizer belongs to an operator, escalate or notify the operator team.
- **AI Note:** Use resource annotations or finalizer naming conventions to identify operator-owned finalizers. If detected, log and escalate.

### 4. Remove Finalizers (if safe)
- **Action:** Remove blocking finalizers to allow resource deletion.
- **Command:**
  ```shell
  kubectl patch <resource-type> <resource-name> -n <namespace> --type='merge' -p '{"metadata":{"finalizers":null}}'
  # or
  oc edit <resource-type>/<resource-name> -n <namespace>
  ```
- **AI Note:** Replace placeholders with actual values. Only remove finalizers if not operator-owned or after escalation. Confirm patch success.

### 5. Confirm Deletion
- **Action:** Verify that the resources and the namespace are deleted.
- **Command:**
  ```shell
  oc get ns/<name>
  ```
- **AI Note:** If the namespace still exists, repeat steps or escalate.

---

> **For AI:** Log all actions, commands, and outputs. If any step fails, capture error details and escalate as needed. For humans, follow the same steps and consult platform documentation if unsure.

---

