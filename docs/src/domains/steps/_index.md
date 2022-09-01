---
title: "Custom Domains - Step by step guide"
weight: 2
sidebarTitle: "Step by step guide"
description: |
  Once your project is ready for production, you can add a custom domain to it.
layout: single
---

{{% description %}}

{{< note >}}
If you are migrating a site from an existing provider,
configure the domain on your project before switching DNS over.
{{< /note >}}

## Before you begin

You need to:

- Have a project that’s ready to be live.
- Have a domain and have access to its settings on the registrar’s website.
- (Optional) Have the [CLI](/development/cli/_index.md) installed locally.
- Make sure that your registrar allows [CNAMEs on Apex domains or one of the alternatives](../steps/dns.md).

## 1. Change your plan to a production plan

Note that the domain used for non-production environments is always generated and can't be customized,
even if your project is on a Production plan.

If you are on a Development plan, you can't add a domain.
To upgrade your subscription to a production plan:

<!--This is in HTML to get the icon not to break the list. -->
<ol>
  <li>Access the <a href="https://console.platform.sh">project overview</a></li>
  <li>On the tile of the project you want to edit, click {{< icon more >}}</strong>More</strong>.</li>
  <li>Click <strong>Edit plan</strong>.</li>
  <li>Change the type of plan to at least <code>Standard</code> to go live. When you make changes, the monthly price you pay is updated.</li>
  <li>Click <strong>Save</strong>.</li>
</ol>

You can find more information on pricing on the [pricing page](https://platform.sh/pricing).

## 2. Configure your DNS provider

{{< note >}}
If you are serving the site through a CDN, configure your DNS provider to point at your CDN account.
The address or CNAME to set for that varies with the CDN provider.
Refer to their documentation or to the [CDN guide](/domains/cdn/_index.md).
{{< /note >}}

If you're not using a CDN, configure your domain name to point to your project:

{{< codetabs >}}

---
title=Using the CLI
file=none
highlight=false
---

1. Sign in to your registrar's management system to configure your domain.
2. Set the time to live (TTL) on your domain to the lowest possible value to minimize transition time.
3. Get the CNAME target by running the [CLI](/development/cli/_index.md) command `platform environment:info edge_hostname`.
4. Add a CNAME record from the `www` subdomain (`www.<YOUR_DOMAIN>`) to the value of the CNAME target.
  If you have multiple domains you want to be served by the same application you need to add a CNAME record for each of them.
  If you are planning to host multiple subdomains on different projects,
  see the additional information about [Subdomains](/domains/steps/subdomains.md) *before* you add your domain to Platform.sh.
5. Add a CNAME/ANAME/ALIAS from your apex domain (`<YOUR_DOMAIN>`) to the value of the CNAME target.
  Not all registrars allow these kinds of records.
  If yours doesn't, see [alternatives](../steps/dns.md).
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
3. Get the CNAME target by accessing your production environment and adapting the auto-generated URL you use to access your website.
  It's in the form `<branch>-<hash>-<project_id>.<region>.platformsh.site` minus the protocol (`https://`).
  For example if the URL to access your domain is `https://main-def456-abc123.eu-2.platformsh.site`, the CNAME target is `main-def456-abc123.eu-2.platformsh.site`.
4. Add a CNAME record from the `www` subdomain (`www.<YOUR_DOMAIN>`) to the value of the CNAME target.
  If you have multiple domains you want to be served by the same application you need to add a CNAME record for each of them.
  If you are planning to host multiple subdomains on different projects,
  see the additional information about [Subdomains](/domains/steps/subdomains.md) *before* you add your domain to Platform.sh.
5. Add a CNAME/ANAME/ALIAS from your apex domain (`<YOUR_DOMAIN>`) to the value of the CNAME target.
  Not all registrars allow these kinds of records.
  If yours doesn't, see [alternatives](../steps/dns.md).
6. Check that the domain and subdomain are working as expected.
7. Set the TTL value back to its previous value.

{{< /codetabs >}}

Note that depending on your registrar and the TTL you set on step 2,
it can take anywhere from 15 minutes to 72 hours for DNS changes to be taken into account.

Example where `<YOUR_DOMAIN>` is `mysite.com`:

- `www.mysite.com` is a CNAME to `main-def456-abc123.eu-2.platformsh.site`.
- `mysite.com` is an ALIAS/CNAME/ANAME  to `main-def456-abc123.eu-2.platformsh.site`.

Both `www.mysite.com` and `mysite.com` point to the same target. The app handles the [redirects](../../define-routes/_index.md).

## 3. Set your domain in Platform.sh

Add a single domain to your Platform.sh project for `<YOUR_DOMAIN>`:

{{< codetabs >}}

---
title=In the console
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

Adding a domain to your project tells the edge layer where to route requests for your website.
You can add multiple domains to point to your project.

Adding a custom domain disables the automatically generated URLs (`main-def456-abc123.eu-2.platformsh.site`).
When a domain is added to your project,
the `{default}` in `routes.yaml` is replaced with `<YOUR_DOMAIN>` anywhere it appears when generating routes to respond to.
Access the original internal domain by running `platform environment:info edge_hostname -e <BRANCH_NAME>`.

{{< note >}}
If you are planning on using subdomains across multiple projects, [the setup differs slightly](subdomains.md).
{{< /note >}}

## Result

With the assumption that all caches are empty, an incoming request for `mysite.com` results in the following:

1. Your browser asks the DNS root servers for `mysite.com`'s DNS A record (the IP address of this host).
   It responds with "it's an alias for `www.main-def456-abc123.eu-2.platformsh.site`" (the CNAME),
   which itself resolves to the A record with IP address `1.2.3.4` (or whatever the actual address is).
   By default DNS requests by browsers are recursive, so there is no performance penalty for using CNAMEs.
2. Your browser sends a request to `1.2.3.4` for domain `mysite.com`.
3. Your router responds with an HTTP 301 redirect to `www.mysite.com` (because that's what `routes.yaml` specified).
4. Your browser looks up `www.mysite.com` and, as above, gets an alias for `www.main-def456-abc123.eu-2.platformsh.site`, which is IP `1.2.3.4`.
5. Your browser sends a request to `1.2.3.4` for domain `www.mysite.com`.
   Your router passes the request through to your application which in turn responds with whatever it's supposed to do.

On subsequent requests, your browser knows to connect to `1.2.3.4` for domain `www.mysite.com` and skips the rest.
The entire process takes only a few milliseconds.

## Optional: Use hosts file

If you require access to the site before the domain name becomes active,
you can create a `hosts` file entry on your computer
and point it to the IP address linked to your project's production branch internal domain.

To get the IP addresses of your internal domain, run `dig +short $(platform environment:info edge_hostname`.

{{< codetabs >}}

---
title=On macOS and Linux
file=none
highlight=false
---

1. With an admin account, open the `/etc/hosts` file with `sudo vi /etc/hosts` or with your favorite text editor.
2. Add the IP and the domain you want to override to that file.
3. Save and close the file.

After adding the line the file looks like:

![Hosts File](/images/config-files/hosts-file.png "0.4")

Once the domain is live, don't forget to delete the entry you added.

<--->

---
title=On windows
file=none
highlight=false
---

1. With an admin account, open the `c:\Windows\System32\Drivers\etc\hosts` file with your favorite text editor.
2. Add the IP to that file.
3. Save and close the file.

After adding the line the file looks like:

![Hosts File](/images/config-files/hosts-file.png "0.4")

Once the domain is live, don't forget to delete the entry you added.

<--->

---
title=In the browser
file=none
highlight=false
---

You can dynamically switch DNS IP addresses without modifying your `hosts` file with the [Firefox LiveHosts add-on](https://addons.mozilla.org/en-US/firefox/addon/livehosts/).

{{< /codetabs >}}