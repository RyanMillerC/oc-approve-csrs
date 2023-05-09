# oc-approve-csrs

How many times has this happened to you?

* You have an OpenShift cluster in a lab (or something like that) 👍
* You power it off (because why run a lab OpenShift cluster 24/7) 😅
* You power it back on a few days later 🔌
* The web console doesn't come up 😬
* You start digging around 🕵️
* You notice some pending CertificateSigningRequests (CSRs) 🤦
* You dig through your old notes or links to find the support article with the
  command approve all pending CSRs 🔎
* You run the command to approve them 💻
* You wait 10 minutes and the web console comes up! 😃

This OpenShift CLI (oc) plugin will approve all pending CSRs in a single
command, without the need to dig through your notes, search the web, or
memorize **this monster command** 😈:

```bash
oc --insecure-skip-tls-verify=true get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc --insecure-skip-tls-verify=true adm certificate approve
```

## Why does this happen?

OpenShift 4 manages CSR approvals automatically. The certificates are short
lived. If an OpenShift cluster is powered off when the certificates expire, new
CSRs will hang in *Pending* status and must be manually approved.

When an OpenShift cluster is bootstrapped, the initial certificates expire in
24 hours. From that point on, certificates will expire every 30 days. It's hard
to judge how long you are able to leave a cluster powered off before you
encounter expired certs. You may have certs that expire in 2 days or certs that
just renewed and don't expire for another 28 days. 🤷

It shouldn't matter if your certs expire though! You have this awesome plugin
to save your cluster. 🎉

## When do you need to do this?

If you boot up a previous powered off OpenShift cluster and can't hit the web
console, you may need to manually approve CSRs. To see if you have pending
CSRs, log in to the cluster with `oc` (or `kubectl`) and run:

```bash
oc get csrs
```

If the output contains CSRs with the status *Pending*, you will need to
manually approve them. 😅

## Installation

**NOTE:** This plugin requires `oc` because it relies on the `oc adm
certificates` subcommand that `kubectl` doesn't have.

Install approve-csrs with Krew:

```bash
oc krew install approve-csrs
```

## Usage

**⚠️ This plugin blindly approves all pending certificates. Because of that,
this plugin should only be used in non-production environments! You've been
warned... ⚠️**

To approve all pending CSRs:

```bash
kubectl approve-csrs
```

The command will intentionally hang for 10 seconds after approving pending CSRs
to approve any subsequent CSRs that pop up as pending. Saves you from typing
the command twice. 😎
