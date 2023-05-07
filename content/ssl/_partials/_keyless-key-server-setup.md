---
_build:
  publishResources: false
  render: never
  list: never
inputParameters: param1
---

Finally, you need to install the key server on your infrastructure, populate it with the SSL keys of the certificates you wish to use to terminate TLS at Cloudflare’s edge, and activate the key server so it can be mutually authenticated.

{{<Aside type="note">}}

If you plan to run Keyless SSL in a [high availability setup](/ssl/keyless-ssl/reference/high-availability/), you may need to set up additional infrastructure (load balancing and health checks).

{{</Aside>}}

### Install

#### Debian/Ubuntu packages

1.  Add the Cloudflare Package Repository as per https://pkg.cloudflare.com/.
2.  Update your OS’ package listings with `apt-get update`.
3.  Install the `gokeyless` server (minimum version used should be `1.5.3`): `sudo apt-get install gokeyless`.

#### RHEL/CentOS packages
    
Use either of the following examples to install the `gokeyless` package for RHEL or CentOS.

**Option 1**

```sh
---
header: RHEL or CentOS
---
$ yum makecache
$ yum-config-manager --add-repo https://pkg.cloudflare.com/gokeyless.repo
$ echo 'gpgkey=https://pkg.cloudflare.com/cloudflare-ascii-pubkey.gpg' >> /etc/yum.repos.d/gokeyless.repo
$ yum install gokeyless
```

**Option 2**

```sh
---
header: RHEL or CentOS
---
$ yum install -y sudo dnf dnf-plugins-core && yum clean all
$ dnf config-manager --add-repo https://pkg.cloudflare.com/gokeyless.repo
$ dnf install gokeyless
```

{{<Aside type="note">}}

Amazon Linux customers may need to update their final installation command to be something similar to `sudo yum install rsyslog shadow-utils && sudo yum install gokeyless`.

{{</Aside>}}

### Configure

Add your Cloudflare account details to the configuration file located at `/etc/keyless/gokeyless.yaml`:

1.  Set the hostname of the key server, for example, `$1`. This is also the value you entered when you uploaded your keyless certificate and is the hostname of your key server that holds the key for this certificate.
2.  Set the Zone ID (found on **Overview** tab of the Cloudflare dashboard).
3.  [Set the Origin CA API key](/fundamentals/api/get-started/ca-keys).

### Populate keys

Install your private keys in `/etc/keyless/keys/` and set the user and group to keyless with 400 permissions. Keys must be in PEM or DER format and have an extension of `.key`:

```sh
$ ls -l /etc/keyless/keys
-r-------- 1 keyless keyless 1675 Nov 18 16:44 example.com.key
```

When running multiple key servers, make sure all required keys are distributed to each key server. Customers typically will either use a configuration management tool such as Salt or Puppet to distribute keys or mount `/etc/keyless/keys` to a network location accessible only by your key servers. Keys are read on boot into memory, so a network path must be accessible during the gokeyless process start/restart.

### Activate

To activate, restart your keyless instance:

- systemd: `sudo service gokeyless restart`
- upstart/sysvinit: `sudo /etc/init.d/gokeyless restart`

If this command fails, try troubleshooting by [checking the logs](/ssl/keyless-ssl/troubleshooting/).