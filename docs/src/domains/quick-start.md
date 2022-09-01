---
title: "Quick Start"
weight: -70
description: |
  Find out how to take your application live on your own custom domain and replace the automatically generated URLs.
---
{{% description %}}

This page gives a basic configuration example.
For more details, see the [step-by-step guide](../domains/steps/_index.md).

## Before you begin

You need:

- A project that's ready to go live.
- A project plan size of **Standard** or larger.
- A domain and have access to its settings on the registrar's website.
- A registrar that allows CNAME records on Apex domains or [one of the alternatives](./steps/dns.md).
- (Optional) Have the [CLI](/development/cli/_index.md) installed locally.

## 1. Configure your DNS zone

Start by configuring your domain name to point to your project:

{{< codetabs >}}

---
title=Using the CLI
file=none
highlight=false
---

1. Sign in to your registrar's management system to configure your domain.
2. Set the time to live (TTL) on your domain to the lowest possible value to minimize transition time.
3. Get the CNAME target by running the following [CLI](/development/cli/_index.md) command: `platform environment:info edge_hostname`
4. Add a CNAME record from the `www` subdomain (`www.<YOUR_DOMAIN>`) to the value of the CNAME target.
5. Add a CNAME/ANAME/ALIAS from your apex domain (`<YOUR_DOMAIN>`) to the value of the CNAME target.
  Not all registrars allow these kinds of records.
  If yours doesn't, see [alternatives](./steps/dns.md).
6. Check that the domain and subdomain are working as expected.
7. Set the TTL value back to its previous value.

<--->

---
title=In the console
file=none
highlight=false
---

1. Sign in to your registrar's management system to configure your domain.
2. Set the time to live (TTL) on your domain to the lowest possible value to minimize transition time.
3. Get the CNAME target by accessing your production environment
  and copying the automatically generated URL you use to access your website.
  For example if the URL to access your production environment is `https://main-def456-abc123.eu-2.platformsh.site`,
  the CNAME target is `main-def456-abc123.eu-2.platformsh.site`.
4. Add a CNAME record from the `www` subdomain (`www.<YOUR_DOMAIN>`) to the value of the CNAME target.
5. Add a CNAME/ANAME/ALIAS from your apex domain (`<YOUR_DOMAIN>`) to the value of the CNAME target.
  Not all registrars allow these kinds of records.
  If yours doesn't, see [alternatives](./steps/dns.md).
6. Check that the domain and subdomain are working as expected.
7. Set the TTL value back to its previous value.

{{< /codetabs >}}

Note that depending on your registrar and the TTL you set on step 2,
it can take anywhere from 15 minutes to 72 hours for DNS changes to be taken into account.

## 2. Set your domain

Note: custom domains can only be added to the default environment on production plans (Standard or larger).
Also, adding a custom domain disables the automatically generated URLs (`main-def456-abc123.eu-2.platformsh.site`).

Add a single domain to your Platform.sh project for `<YOUR_DOMAIN>`:

{{< codetabs >}}

---
title=In the Console
file=none
highlight=false
---

<!--This is in HTML to get the icon not to break the list. -->
<ol>
  <li>Select the project where you want to add a domain.</li>
  <li>Click {{< icon settings >}} <strong>Settings</strong>.</li>
  <li>Click <strong>Domains</strong>.</li>
  <li>Enter <code>&lt;YOUR_DOMAIN&gt;</code> into the <strong>Domain</strong> field.</li>
  <li>Click <strong>+ Add</strong>.</li>
</ol>

<--->
---
title=Using the CLI
file=none
highlight=false
---

Run the following command:

```bash
platform domain:add -p <PROJECT_ID> <YOUR_DOMAIN>
```

{{< /codetabs >}}

## What's next

- Optional: [use a Content Delivery Network (CDN)](../domains/cdn/_index.md)
- Optional: [use subdomains across multiple projects](./steps/subdomains.md).
