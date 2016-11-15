# easy-vuls

This is a single script to provide a quick and easy way to use
[vuls](https://hub.docker.com/r/vuls/vuls/),
[go-cve-dictionary](https://hub.docker.com/r/vuls/go-cve-dictionary/), and
[VulsRepo](https://github.com/usiusi360/vulsrepo).

This requires Python3 and Docker.

This script is likely to change, and the documentation is not yet complete.

The projects this script depends on are still under development, so things may
not work as expected. If you find any issues, please report them to he relevant
project!

At the moment you have to manually run `docker pull` to get in the latest
images whenever they have been updated:
```bash
docker pull vuls/vuls
docker pull vuls/go-cve-dictionary
docker pull vuls/vulsrepo
```


## Getting started
To begin, you will need Python3 and Docker. There are no additional Python
module dependencies.

You can use `./easy-vuls --help` to get information about commands.


### Authentication
You can connect to the hosts through `SSH_AUTH_SOCK` provided by `ssh-agent`,
which is passed into the container.

You can use private keys by putting them in `easy-vuls.data/keys/` and then
referring to it in your configuration file.

See [`vuls prepare`](https://github.com/future-architect/vuls#usage-prepare)'s
documentation for more information about preparing hosts.


### Update the database
Once you have this script and Docker, you will have to update the database.
If you do not, you will get an error message telling you to do it before
you retry.
```bash
./easy-vuls database update
```

This uses
[fetchnvd](https://github.com/kotakanbe/go-cve-dictionary#usage-fetch-nvd-data).


### Create a vuls config file
You will have to provide a config file for vuls to both prepare and scan hosts.
A minimal config file to scan a single host would be something like:
```toml
[server.my-host]
host = "my-host.example.com"
user = "root"
```

The [vuls documentation](https://github.com/future-architect/vuls#configuration)
goes into more detail on everything that there is to configure, including
parameters for docker containers on the hosts.


### Prepare hosts
The `vuls prepare` command will go though the hosts in the provided config file
and make sure that any necissary preparations have been made. You can parepare
hosts through easy-vuls using:
```bash
./easy-vuls scan --config=my-config.toml prepare
```

Some host OSes don't need any preparation if you are don't mind providing vuls
with root access on them.

See [`vuls prepare`](https://github.com/future-architect/vuls#usage-prepare)'s
documentation for more information about preparing hosts.


### Run a scan
To run `vuls scan` with the database retrieved earlier, you can use:
```bash
./easy-vuls scan --config=my-config.toml
```


### Show the results
The results of scans are printed to the terminal, and JSON files for each
scan/host is stored in `./easy-vuls.results/`.

To start serving results on [127.0.0.1:8080/vulsrepo/](127.0.0.1:8080/vulsrepo/) through [VulsRepo](https://github.com/usiusi360/vulsrepo)
you would use:
```bash
./easy-vuls results serve
```

You can also browse results in your terminal through `vuls tui` by using:
```bash
./easy-vuls results browse
```


## Security
### SSH access
The script uses `SSH_AUTH_SOCK` from the environment when it gets run. This
will grant the containers access to any of the machines which allow any key on
your keyring while the container is alive.

You may also pass in private keys for the container to use, of course this adds
to how much you would be trusting the container and its source.

vuls may require a lot of permissions on a target OS to do its job, for example
on Debian it requires root/sudo access to `apt-get`, which can be used to
install packages.

### File ownership
The files created inside the docker images are owned by `root:root`, which is
a bit inconvenient if you want to have them owned by your current user on the
host.

To get around this, the script does a `chown -R` on the volumes mounted in so
that the ownership matches that of `./easy-vuls`. This will include the
contents of the `./easy-vuls.*/` directories, along with any config file you
provide.
