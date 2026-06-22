# Exercise 5: Helm Upgrade Failure

This repository contains the files for Exercise 5, which focuses on troubleshooting and resolving a common Helm upgrade failure caused by immutable fields in Kubernetes resources.

## What is this exercise about?
In Kubernetes, certain fields on resources are immutable after they are created. A common example is the `spec.selector.matchLabels` field of a `Deployment`. This exercise demonstrates what happens when you attempt to change these immutable fields during a `helm upgrade` and how to properly handle or avoid this scenario. 

In this specific lab, the `payment-service` Helm chart has its selector labels modified (in this case, hardcoded to `app: payment-v2` in the `_helpers.tpl` file). Because the selector labels define how the Deployment manages its ReplicaSets, Kubernetes prevents you from modifying them on the fly.

## How it works
1. **Initial Deployment**: The chart is initially installed with a set of selector labels (for instance, `app: payment-v1`). Kubernetes creates the Deployment and ReplicaSet based on these labels.
2. **The Breaking Change**: The chart templates are updated to use a different selector label (`app: payment-v2`).
3. **The Upgrade Failure**: Running `helm upgrade` attempts to apply this change. The Kubernetes API server rejects the patch request, returning an error stating that the `spec.selector` field is immutable.

## How to demonstrate it
Follow these steps to reproduce the Helm upgrade failure locally:

1. **Install the initial release**:
   First, ensure your `helm-lab/payment-service/templates/_helpers.tpl` sets the selector label to `v1`:
   ```yaml
   {{- define "payment-service.selectorLabels" -}}
   app: payment-v1
   {{- end }}
   ```
   Install the chart:
   ```bash
   helm install my-payment-service ./helm-lab/payment-service
   ```

2. **Introduce the breaking change**:
   Modify the same `_helpers.tpl` file to change the label to `v2` (as it is currently in this repository):
   ```yaml
   {{- define "payment-service.selectorLabels" -}}
   app: payment-v2
   {{- end }}
   ```

3. **Attempt an upgrade**:
   Run the Helm upgrade command:
   ```bash
   helm upgrade my-payment-service ./helm-lab/payment-service
   ```
   **Expected Result**: You will receive an error from the Kubernetes API similar to:
   > `Error: UPGRADE FAILED: cannot patch "my-payment-service" with kind Deployment: Deployment.apps "my-payment-service" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"payment-v2"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable`

4. **Resolution**:
   To resolve this, you must either:
   - Revert the selector labels back to what they were during the initial installation.
   - Delete the deployment and recreate it (which involves downtime) by running `helm uninstall my-payment-service` and installing it again.

## Demo Video
You can watch the demonstration and explanation of the exercise [here](https://drive.google.com/file/d/1yZj6UYECLPL1meP1-xgz5tj6okGX1Mn0/view?usp=sharing).

## Contents
- `helm-lab/`: Contains the Helm chart and related files for the exercise.
