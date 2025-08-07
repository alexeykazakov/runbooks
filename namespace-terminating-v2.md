# namespace-terminating

## Overview

The `namespace-terminating` alert is triggered when a namespace is stuck in the `terminating` phase for an excessive duration.

## AI Analysis Instructions

> **IMPORTANT FOR AI AGENTS:** This runbook is for **analysis and recommendation only**. You are in read-only investigation mode. DO NOT attempt to modify, patch, delete, or fix any resources. Your role is to diagnose the issue and provide specific remediation recommendations for human operators.

## Investigation Workflow

Follow these steps to systematically investigate the namespace termination issue:

### Phase 1: Namespace Status Investigation

#### Step 1: Retrieve Namespace Details
**Objective:** Get complete namespace information to understand current state.

**AI Investigation:**
- Use available tools to retrieve the full namespace YAML
- Focus on `status.conditions` and `spec.finalizers` fields
- Look for `deletionTimestamp` to confirm termination is in progress

**What to analyze:**
- Namespace finalizers (usually just `kubernetes`)
- Deletion timestamp and grace period
- Status conditions, especially `NamespaceFinalizersRemaining` and `NamespaceContentRemaining`

#### Step 2: Identify Blocking Resources
**Objective:** Determine what resources are preventing namespace deletion.

**AI Investigation:**
- Examine `status.conditions` array for conditions like:
  - `NamespaceContentRemaining` (indicates remaining resources)
  - `NamespaceFinalizersRemaining` (indicates stuck finalizers)
- Look for messages like "Some resources are remaining: secrets. has 1 resource instances"
- Extract specific resource types and counts

**Critical:** If ANY remaining resources are found, proceed to Phase 2.

### Phase 2: Resource-Level Investigation

#### Step 3: Catalog Remaining Resources
**Objective:** Get detailed information about each remaining resource.

**AI Investigation:**
- For each resource type identified in Step 2, list all instances in the namespace
- Retrieve full YAML details of each remaining resource
- **Do not skip this step** - resource names and details are essential for proper analysis

**Example:** If namespace status shows "configmaps. has 2 resource instances":
- List all configmaps in the namespace
- Get complete YAML for each configmap found

#### Step 4: Analyze Resource Finalizers
**Objective:** Identify specific finalizers blocking resource deletion.

**AI Analysis:**
- For each resource from Step 3, examine `metadata.finalizers` field
- Document exact finalizer names and their purposes
- Categorize finalizers as:
  - Standard Kubernetes finalizers
  - Operator-owned finalizers (e.g., `example.com/finalizer-name`)
  - Custom application finalizers

### Phase 3: Root Cause Analysis

#### Step 5: Determine Finalizer Ownership
**Objective:** Understand what owns each blocking finalizer.

**AI Analysis:**
- Check resource annotations for operator information
- Analyze finalizer naming patterns (domain-based names often indicate operators)
- Identify whether finalizers are:
  - Safe to remove manually
  - Require operator intervention
  - Need escalation to specific teams

## Expected AI Output

After completing the investigation, provide a comprehensive analysis report with:

### 1. Recommended Actions for Human Operators

**For Standard Finalizers:**
```bash
# Remove finalizer from specific resource
kubectl patch <resource-type> <resource-name> -n <namespace> \
  --type='merge' -p '{"metadata":{"finalizers":[]}}'
```

**For Operator Finalizers:**
- Document which operator owns the finalizer
- Recommend checking operator status and logs
- Suggest operator team escalation if needed

**For Unknown Finalizers:**
- Recommend investigation of finalizer purpose before removal
- Suggest checking cluster documentation or operator catalogs

### 2. Root Cause Analysis
- Specific namespace and resource names involved
- Exact finalizers blocking deletion
- Why these finalizers are stuck (if determinable)

### 3. Current System State
- Complete inventory of blocking resources
- Finalizer details and ownership
- Any error conditions or unusual states

### 4. Prevention Recommendations
- Best practices for avoiding similar issues
- Monitoring suggestions
- Process improvements

## Example Scenarios

### Scenario 1: Resource with Custom Finalizer
```yaml
# If you find a resource like this:
metadata:
  finalizers:
    - example.com/finalizer-name
```

**AI Should Recommend:**
"The resource has a custom finalizer 'example.com/finalizer-name' which appears to be owned by an operator. Recommend checking if the operator is running and healthy before manually removing the finalizer."

### Scenario 2: Multiple Resource Types
**AI Should Investigate:**
- Each resource type separately
- Interdependencies between resources
- Priority order for finalizer removal

## Success Criteria

Investigation is complete when you can answer:
- [ ] What specific resources are blocking namespace deletion?
- [ ] What finalizers are on each resource and who owns them?
- [ ] What exact commands should operators run to resolve the issue?
- [ ] Are there any risks or prerequisites for the remediation?
