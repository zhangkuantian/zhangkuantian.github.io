# Could not resolve host: mirrorlist.centos.org; Unknown error
最近在基于CentOS 7 打 Docker 包, 发现无论怎么替换 `CentOS-Base.repo` 都不生效，老是报错误:
```shell
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
Could not retrieve mirrorlist http://mirrorlist.centos.org?arch=x86_64&release=7&repo=sclo-rh error was
14: curl#6 - "Could not resolve host: mirrorlist.centos.org; 未知的错误"


 One of the configured repositories failed (Unknown),
 and yum doesn't have enough cached data to continue. At this point the only
 safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Run the command with the repository temporarily disabled
            yum --disablerepo=<repoid> ...

     4. Disable the repository permanently, so yum won't use it by default. Yum
        will then just ignore the repository until you permanently enable it
        again or use --enablerepo for temporary usage:

            yum-config-manager --disable <repoid>
        or
            subscription-manager repos --disable=<repoid>

     5. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=<repoid>.skip_if_unavailable=true

Cannot find a valid baseurl for repo: centos-sclo-rh/x86_64
```

后来在 https://serverfault.com/questions/1161816/mirrorlist-centos-org-no-longer-resolve 的帮助下，最后解决了，发出来记录以下，避免下次找不到解决办法了

## 解决办法
```shell
sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo
sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo
sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo
```
