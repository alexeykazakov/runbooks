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

### 2. Analyze Namespace Status
- **Action:** Analyze the namespace status to determine which resources are still present and preventing deletion.
- **AI Note:** Examine the `status.conditions` array for conditions like `NamespaceContentRemaining` and `NamespaceFinalizersRemaining`. Look for messages indicating specific resource types and counts (e.g., "Some resources are remaining: <resource_type>. has 1 resource instances"). If ANY remaining resources are found, Step 3 becomes MANDATORY.

### 3. List Remaining Resources **[MANDATORY IF STEP 2 FINDS REMAINING RESOURCES]**
- **Action:** List all resources still present in the namespace according to the namespace status.
- **Command:**
  ```shell
  oc get <resource> -n <namespace> -o yaml
  ```
- **AI Note:** **CRITICAL:** If Step 2 revealed remaining resources (e.g., "Some resources are remaining: <resource_type>. has 1 resource instances"), you MUST retrieve the actual resource details. Extract resource types from the `NamespaceContentRemaining` condition message and call the appropriate commands for each resource type. Do NOT proceed without getting the actual resource YAML - knowing "1 <resource_type> exists" is not enough, you need the resource's content, name, and finalizers.

### 4. Check for Stuck Finalizers **[MANDATORY IF STEP 3 FOUND RESOURCES]**
- **Action:** For each remaining resource retrieved in Step 3, check if it is stuck due to finalizers.
- **AI Note:** Inspect the `metadata.finalizers` field in each resource from Step 3. Identify which resources have non-empty finalizers and note their values. This step requires the actual resource YAML from Step 3.

---

## ✅ DIAGNOSTIC PHASE COMPLETE

**At this point, you should have:**
- Namespace status with all conditions and finalizers identified
- Complete YAML details of any remaining resources preventing deletion
- Specific finalizer names and their associated resources
- Clear understanding of what is blocking the namespace deletion

**If you don't have all the above data, continue data gathering. If you do, proceed to remediation steps.**

---

## REMEDIATION PHASE

### 5. Operator Finalizer Handling
- **Action:** If a finalizer belongs to an operator, escalate or notify the operator team.
- **AI Note:** Use resource annotations or finalizer naming conventions to identify operator-owned finalizers. If detected, log and escalate.

### 6. Remove Finalizers (if safe)
- **Action:** Remove blocking finalizers to allow resource deletion.
- **Command:**
  ```shell
  kubectl patch <resource-type> <resource-name> -n <namespace> --type='merge' -p '{"metadata":{"finalizers":null}}'
  # or
  oc edit <resource-type>/<resource-name> -n <namespace>
  ```
- **AI Note:** Replace placeholders with actual values. Only remove finalizers if not operator-owned or after escalation. Confirm patch success.

### 7. Confirm Deletion
- **Action:** Verify that the resources and the namespace are deleted.
- **Command:**
  ```shell
  oc get ns/<name>
  ```
- **AI Note:** If the namespace still exists, repeat steps or escalate.

---

> **For AI:** Log all actions, commands, and outputs. If any step fails, capture error details and escalate as needed. For humans, follow the same steps and consult platform documentation if unsure.

---

