---
title: Forester Project
---

{{< blocks/cover title="Forester Project" image_anchor="top" height="full" >}}
<a class="btn btn-lg btn-primary me-3 mb-4" href="/about/">
  Learn More <i class="fas fa-lightbulb ms-2"></i>
</a>
<a class="btn btn-lg btn-secondary me-3 mb-4" href="/docs/">
  Documentation <i class="fas fa-arrow-alt-circle-right ms-2"></i>
</a>
<a class="btn btn-lg btn-secondary me-3 mb-4" href="https://github.com/foresterorg/forester">
  Github<i class="fab fa-github ms-2 "></i>
</a>
<p class="lead mt-5">Unattended bare-metal image-based Anaconda installation service.</p>
{{< blocks/link-down color="info" >}}
{{< /blocks/cover >}}


{{% blocks/lead color="primary" %}}
Forester provides unattended bare-metal installation network boot workflow for Fedora, Red Hat or compatible OS images created by Image Builder. It utilizes modern technologies like UEFI HTTP Boot, Redfish, Anaconda, SecureBoot and X509 for fast and secure image-based OS installations. Forester is a simple service with REST/RPC API and CLI.

The project is currently in early development and we are looking for feedback.
{{% /blocks/lead %}}


{{% blocks/section color="dark" type="row" %}}
{{% blocks/feature icon="fa-upload" title="Upload" %}}
Upload OS images created by [Image Builder](https://www.osbuild.org/) or [Red Hat Hybrid Cloud Console](https://console.redhat.com/insights/image-builder) into Forester service.
{{% /blocks/feature %}}


{{% blocks/feature icon="fa-magnifying-glass" title="Configure" %}}
Setup your servers for HTTP UEFI Boot and configure Redfish credentials in Forester so it can discover your infrastructure.
{{% /blocks/feature %}}


{{% blocks/feature icon="fa-gas-pump" title="Deploy" %}}
Start deploying images onto your bare-metal servers en-masse.
{{% /blocks/feature %}}
{{% /blocks/section %}}


{{% blocks/section %}}

To evaluate Forester, you need a Linux machine capable of running Podman containers (e.g. RHEL VM with 4 GB RAM). Servers for deployment need to have UEFI 2.1 or higher (with UEFI HTTP Boot support) and out-of-band management capability with Redfish protocol enabled (Dell IDRAC, HP iLO and others). The only communication channel is HTTP (HTTPS) and there is no need to configure anything on the DHCP for proof-of-concept deployments.
{.h3 .text-center}

{{% /blocks/section %}}


{{% blocks/section type="row" %}}

{{% blocks/feature icon="fa-pen-to-square" title="Give us feedback" url="https://github.com/foresterorg/forester/discussions/new?category=general"%}}
We are gathering feedback from Fedora users and Red Hat customers.
{{% /blocks/feature %}}

{{% blocks/feature icon="fab fa-github" title="Contribute!" %}}
Send us a [Pull Request](https://github.com/foresterorg/forester) on **GitHub**.
{{% /blocks/feature %}}

{{% blocks/feature icon="fab fa-readme" title="About Forester" url="/about/" %}}
More information about Forester.
{{% /blocks/feature %}}

{{% /blocks/section %}}
