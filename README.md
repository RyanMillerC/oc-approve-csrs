# kubectl-approve-csrs

Kubectl plugin to approve all pending Kubernetes CertificateSigningRequests
(CSRs) in a single command.

**⚠️ This plugin blindly approves all pending certificates. Because of that,
this plugin should only be used in non-production environments! ⚠️**

## Why does this exist?

OpenShift 4 manages CSR approvals automatically. The certificates are short
lived. If an OpenShift cluster is powered off when the certificates expire, new
CSRs will hang in *Pending* status and must be manually approved.

When an OpenShift cluster is bootstrapped, the initial certificates expire in
24 hours. From that point on, certificates will expire every 30 days.

## When do you need to do this?

If you boot up a previous powered off OpenShift cluster and can't hit the web
console, you may need to manually approve CSRs. To see if you have pending
CSRs, log in to the cluster with `kubectl` or `oc` and run:

```bash
kubectl get csrs
```

If the output contains CSRs with the status *Pending*, you will need to
manually approve them.

## Installation

Install with Krew:

```bash
kubectl krew install approve-csrs
```

## Usage

To approve all pending CSRs:

```bash
kubectl approve-csrs
```

**NOTE:** You might need to run the command multiple times. Sometimes you need
to wait a moment, then approve a second round of pending CSRs.
