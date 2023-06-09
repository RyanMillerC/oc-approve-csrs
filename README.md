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

This [OpenShift CLI] plugin will approve all pending CSRs in a single `oc`
command, without the need to dig through your notes, search the web, or
memorize **this monster command** 😈:

```bash
$ oc --insecure-skip-tls-verify=true get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc --insecure-skip-tls-verify=true adm certificate approve
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
$ oc get csrs
```

If the output contains CSRs with the status *Pending*, you will need to
manually approve them. 😅

## Installation

This plugin requires the [OpenShift CLI] client `oc` to be in your PATH because
it relies on the `oc adm certificates` subcommand. You can still run this
plugin as `kubectl approve-csrs` but the `oc` executable has to be available.

[Krew] is recommended to install this plugin. (See the [OpenShift documentation
on Krew] for more details.)

This plugin isn't in the [Krew Plugin Index], so you need to install by specifying
the [krew-plugin.yaml] install manifest:

```bash
$ oc krew install --manifest-url=https://raw.githubusercontent.com/RyanMillerC/oc-approve-csrs/main/krew-plugin.yaml
```

If you can't use Krew to install plugins, you can manually copy the
`oc-approve_csrs` script from this repo and place it in your PATH.

## Usage

**⚠️ This plugin blindly approves all pending certificates. Because of that,
this plugin should only be used in non-production environments! You've been
warned... ⚠️**

To approve all pending CSRs:

```bash
$ oc approve-csrs
```

The command will intentionally hang for 10 seconds after approving pending CSRs
to approve any subsequent CSRs that pop up as pending. Saves you from typing
the command twice. 😎

In some circumstances you may need to run the command twice to manually approve
a second round of CSRs that come in after our 10 second window.

[Krew Plugin Index]: https://krew.sigs.k8s.io/plugins/
[Krew]: https://krew.sigs.k8s.io/
[OpenShift CLI]: https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html
[OpenShift Documentation on Krew]: https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/managing-cli-plugins-krew.html
[Recover from Expired Certs]: https://docs.openshift.com/container-platform/latest/backup_and_restore/control_plane_backup_and_restore/disaster_recovery/scenario-3-expired-certs.html
[krew-plugin.yaml]: krew-plugin.yaml
