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

To fix this, use the build option `--repo-host HOSTNAME` to allow access to the source code repository host in the build environment. You can specify this option multiple times if you have multiple hosts.

For example:

```
mbl-tools/build/run-me.sh --builddir /path/to/build --outputdir /path/to/output -- --branch master --machine imx7s-warp-mbl --repo-host ssh.dev.azure.com --repo-host gitlab.com
```


***

Copyright © 2020 Arm Limited (or its affiliates)
