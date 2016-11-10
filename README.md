# vuls-scan

This provides a single script to provide a quick and easy way to use
[vuls](https://hub.docker.com/r/vuls/vuls/) and
[go-cve-dictionary](https://hub.docker.com/r/vuls/go-cve-dictionary/).

This requires Python3 and Docker.

This is currently the work of a couple of evenings, so it is probably good to
wait until this has matured a bit more before using taking anything as final.

## Minimal steps
Below are the minimal steps to running and reviewing a scan.

### Prepare hosts/authenication
You can connect to the hosts using `ssh-agent` which is what I would currently recommend unless you have a vuls user set up on each host, in which case I it
wouldn't be quite as crazy to put a private key into the container.

You can use private keys by putting them in `vuls-scan.data/keys/` and then referring to it in your config file. ***Think extra carefully before doing
this.***

A fair few types of host don't need any preparation if you are okay with running
the scans as root on target machines.  see [`vuls prepare`](https://github.com/future-architect/vuls#usage-prepare)

I need to write more about this, and also update the code to at least try to
run `vuls prepare`.

### Minimal usage
Once you have this script and Docker available, you will have to update the
database. If you do not, you will get an error message telling you to do it
before you retry.
```bash
$ ./vuls-scan database update
```

### Create a vuls scan config file
You will have to provide a config file for vuls, but all you need to scan a single host would be:
```toml
[server.name-of-results]
host = "myhost.domain.tld"
user = "root"
```

The [documentation](https://github.com/future-architect/vuls#configuration)
goes into more details on everything that there is to congiure. Here is an
example with all options that you can configure for all/individual hosts if
you want to:
```toml
[default]
port = "22"
user = root

[server]
[server.mail]
host = "mail.my-domain.net"
# put the keys in ./vuls-scan.data/keys/mail_rsa to mount them in
keyPath = "/vuls/keys/mail_rsa"

[server.logs]
host = "logs.my-domain.net"
ignoreCves = [
  "CVE-2011-1155"
]

[server.port]
host = "port.my-domain.net"
port = "180"
# Add optional data to the resulting JSON resulting
optional = [
  ["key", "value"],
  ["opinion", "non-standard ports are bad"]
]
# see the vuls documentation
cpeNames = [
  "cpe:/a:rubyonrails:ruby_on_rails:4.2.1",
]

[server.host]
host = "host"
# scan Docker containers running on the host
containers = [
  "nginx",
  "my-service",
]
```
The config will be copied to `vuls-scan.data/config.toml` for the run, so that
it ends inside the vuls container at `/vuls/config.toml`.


## Caveats

### vuls prepare
It looks like [`vuls prepare`](https://github.com/future-architect/vuls#usage-prepare)
doesn't always work. During testing with access to root on a Debian 8 (Jessie)
it was unable to install `aptitude` as `apt-get` raised a `[yN]` prompt without
an interactive session, so failed.

This should be easy enough to fix in `vuls`
by providing `-y` or setting an environment variable to tell `apt` that the
session isn't interactive (I can't remember what right now).

## vuls scan
At the moment the results of vuls' scan are owned by root. I'm not sure of a
workaround, some ideas are:
 * see about updating [`vuls scan`](https://github.com/future-architect/vuls#usage-scan)
  to work around this, using either the `Dockerfile` and/or program. This is probably the best thing to do if reasonable to put a pull request in for.
 * add a flag to `scan` which chmods the results to the same user:group as the
  `vuls-scan` script. This doesn't reduce the permissions needed but may make
  life easier
 * mention a --userns-remap option for users who don't mind getting their hands
  dirty. It sounds like this involves system wide changes that require root to
  set up, so doesn't help maintain a minimal barrier to use.
