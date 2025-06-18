# EKS Security Best Practice Policy Set Deployment Guide

This guide provides step-by-step instructions for deploying the EKS Security Best Practice Policy Set to your EKS cluster using the provided policy set YAML.

## Prerequisites

Before deploying the policy set, ensure you have:

1. **EKS Cluster Access**: A running EKS cluster with proper access
2. **kubectl**: Configured to communicate with your EKS cluster
3. **Nirmata Controller**: Already deployed in your cluster (see SETUP.md)
4. **Kyverno Operator**: Already deployed with PolicySet feature enabled
5. **Nirmata Account**: Access to try.nirmata.io with API key

## Step-by-Step Deployment Process

### Step 1: Verify Prerequisites

First, ensure all required components are running:

```bash
# Check if Nirmata Controller is running
kubectl get pods -n nirmata

# Check if Kyverno Operator is running
kubectl get pods -n nirmata-system

# Verify PolicySet CRD is available
kubectl get crd policysets.security.nirmata.io
```

### Step 2: Prepare the Policy Set YAML

The policy set YAML is located at `Policysets/eks-security-best-practice-policyset.yaml`. Review the configuration:

```yaml
apiVersion: security.nirmata.io/v1alpha1
kind: PolicySet
metadata:
  name: eks-security-best-practice-policyset
  namespace: nirmata-system
spec:
  source:
    helm:
      chartName: eks-security-best-practice
      chartRepo: https://github.com/nirmata/eks-best-practices-checks-with-kyverno.git
      version: 0.1.0
```

### Step 3: Deploy the Policy Set

Apply the policy set to your cluster:

```bash
# Navigate to the project directory
cd /path/to/eks-best-practices-checks-with-kyverno

# Apply the policy set
kubectl apply -f Policysets/eks-security-best-practice-policyset.yaml
```

### Step 4: Verify Policy Set Deployment

Check the status of the policy set:

```bash
# Check if the policy set was created
kubectl get policyset -n nirmata-system

# Get detailed information about the policy set
kubectl describe policyset eks-security-best-practice-policyset -n nirmata-system

# Check the status of the policy set
kubectl get policyset eks-security-best-practice-policyset -n nirmata-system -o yaml
```

### Step 5: Monitor Policy Set Processing

The Kyverno Operator will process the policy set and deploy the policies:

```bash
# Check Kyverno Operator logs for policy set processing
kubectl logs -n nirmata-system -l app.kubernetes.io/name=kyverno-operator -f

# Check if policies are being created
kubectl get policies -A

# Check if policy reports are being generated
kubectl get policyreports -A
```

### Step 6: Verify Individual Policies

The policy set includes several security policies. Verify they are deployed:

```bash
# List all policies
kubectl get policies -A

# Check specific policies that should be deployed:
# - namespace-quota-check
# - restrict-binding-system-groups
# - restrict-wildcard-action-for-clusterrole
# - restrict-wildcard-resources-for-clusterrole

# Get details of a specific policy
kubectl describe policy namespace-quota-check -n nirmata-system
```

### Step 7: Test Policy Enforcement

Test that the policies are working correctly:

```bash
# Test namespace quota policy by creating a namespace
kubectl create namespace test-namespace

# Test cluster role restrictions by attempting to create a cluster role with wildcard resources
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: test-wildcard-role
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
EOF
```

### Step 8: Monitor Policy Violations

Check for any policy violations:

```bash
# Check policy reports for violations
kubectl get policyreports -A

# Get detailed policy report
kubectl describe policyreport <policy-report-name> -n <namespace>

# Check cluster policy reports
kubectl get clusterpolicyreports
```

## Policy Set Contents

The EKS Security Best Practice Policy Set includes the following policies:

1. **namespace-quota-check.yaml**: Enforces resource quotas on namespaces
2. **restrict-binding-system-groups.yaml**: Prevents binding to system groups
3. **restrict-wildcard-action-for-clusterrole.yaml**: Restricts wildcard actions in cluster roles
4. **restrict-wildcard-resources-for-clusterrole.yaml**: Restricts wildcard resources in cluster roles

## Troubleshooting

### Common Issues and Solutions

1. **Policy Set Not Processing**:
   ```bash
   # Check Kyverno Operator logs
   kubectl logs -n nirmata-system -l app.kubernetes.io/name=kyverno-operator
   
   # Verify PolicySet CRD exists
   kubectl get crd policysets.security.nirmata.io
   ```

2. **Policies Not Deployed**:
   ```bash
   # Check if the Helm chart repository is accessible
   helm repo add eks-security https://github.com/nirmata/eks-best-practices-checks-with-kyverno.git
   
   # Verify chart exists
   helm search repo eks-security
   ```

3. **Policy Violations Not Detected**:
   ```bash
   # Check if Kyverno is running
   kubectl get pods -n kyverno
   
   # Check Kyverno logs
   kubectl logs -n kyverno -l app.kubernetes.io/name=kyverno
   ```

### Verification Commands

```bash
# Complete status check
echo "=== Checking Policy Set Status ==="
kubectl get policyset -n nirmata-system

echo "=== Checking Deployed Policies ==="
kubectl get policies -A

echo "=== Checking Policy Reports ==="
kubectl get policyreports -A

echo "=== Checking Cluster Policy Reports ==="
kubectl get clusterpolicyreports

echo "=== Checking Kyverno Status ==="
kubectl get pods -n kyverno
```

## Updating the Policy Set

To update the policy set to a newer version:

```bash
# Edit the policy set YAML to update the version
kubectl edit policyset eks-security-best-practice-policyset -n nirmata-system

# Or apply the updated YAML
kubectl apply -f Policysets/eks-security-best-practice-policyset.yaml
```

## Removing the Policy Set

To remove the policy set and all associated policies:

```bash
# Delete the policy set
kubectl delete policyset eks-security-best-practice-policyset -n nirmata-system

# Verify policies are removed
kubectl get policies -A
```

## Next Steps

After successful deployment:

1. **Monitor Policy Compliance**: Regularly check policy reports for violations
2. **Review and Tune Policies**: Adjust policy rules based on your specific requirements
3. **Set Up Alerts**: Configure monitoring and alerting for policy violations
4. **Document Exceptions**: Create policy exceptions for legitimate use cases

## Support

For additional support:
- Check the [SETUP.md](SETUP.md) file for component deployment details
- Refer to [Nirmata Documentation](https://docs.nirmata.com)
- Contact Nirmata Support: support@nirmata.com
