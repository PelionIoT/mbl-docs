# Troubleshooting

## Repository access problems

If during a build you have a problem accessing a source code repository by SSH, you may see an output like:

```
Cloning into bare repository '/home/user/build/ssh.dev.azure.com.v3.myorg.myproject.myproject.git'...
ssh: Could not resolve hostname ssh.dev.azure.com:v3: Name or service not known
fatal: Could not read from remote repository.

Please make sure you have the correct access rights and the repository exists.

ERROR: myrecipe do_fetch: Fetcher failure for URL: 'git://git@ssh.dev.azure.com:v3/myorg/myproject/myproject.git;protocol=ssh;nobranch=1'. Unable to fetch URL from any source.
```

or:

```
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
error: Cannot fetch Freescale/meta-freescale-3rdparty from ssh://git@github.com/Freescale/meta-freescale-3rdparty
```

To fix this:

1. Make sure you have set up an SSH agent. For more information, see [the GitHub SSH documentation](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).
2. If the repository is not on GitHub, use the build option `--repo-host HOSTNAME` to allow access to the source code repository host in the build environment. If you have multiple hosts, you can specify this option multiple times.

For example:

```
mbl-tools/build/run-me.sh --builddir /path/to/build --outputdir /path/to/output -- --branch mbl-os-0.9 --machine imx7d-pico-mbl --repo-host ssh.dev.azure.com --repo-host gitlab.com
```
