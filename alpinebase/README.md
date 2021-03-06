# docker image: alpinebase [Layer 1]

This layer 1 docker image is built upon [alpine image](https://hub.docker.com/r/_/alpine/) `edge` tag,which itself is built from [Alpine Linux](https://alpinelinux.org/) [`Edge`](http://wiki.alpinelinux.org/wiki/Edge) branch for best support of DNS search paths and new packages, such as php7@edge/testing repository.If the intent of this image is to be a minimal base for single-service containers, tracking `edge` is not expected to cause issues (packages are still stable releases, only Alpine itself is a development build).For multi-process containers, it may generate some unexpected results, but the benefit over risk ratio seems high enough for non-critical applications (database applications using this image are required to do more testing before production).

## Description

Install [su-exec](https://github.com/ncopa/su-exec) instead of [gosu](https://github.com/tianon/gosu) to restrict user privileges. Symlink *sux->su-exec*. Usage:
`exec sux daemon cmdname` which drops root privileges to act as user daemon.


## Additions
Packages: `sed tar unzip libcap su-exec`

Files and Folders: 
```
/:
	entrypoint

/root/:
	.profile
	.profiles.d

/etc/apk/:
	repositories
    
/usr/bin/:
	apk-install
    apk-remove
    apk-cleanup
    set-timezone

/var/:
	www         [Owner:www-data:www-data]
```

Users and Groups:
```
www-data:www-data
```

## libcap Usage and Caveats
Use [setcap@libcap](http://www.friedhoff.org/posixfilecaps.html) to grant PORT<=1024 access to non-root system users(UID<=999) such as 80, 443 etc. Usage:
`setcap CAP_NET_BIND_SERVICE=+eip /usr/bin/go-dnsmasq`
However: 
`setcap` will fail during docker image building if the host kernel hadn't been compiled with a proper config line `CONFIG_AUFS_XATTR=y` for the `aufs` filesystem driver.
Check if the kernel has it: `grep AUFS /boot/config*` or `grep AUFS_X /boot/config*`

Workarounds(Don't choose `aufs`):
In default config `/etc/default/docker` or systemd service unit `/etc/systemd/system/docker.service` add a line or edit an existing DOCKER_OPTS line:
`DOCKER_OPTS="--storage-driver=devicemapper` to use DeviceMapper, OR
`DOCKER_OPTS="--storage-driver=overlay"` to use [OverlayFS/2](https://docs.docker.com/engine/userguide/storagedriver/selectadriver/) if possible. 
`sudo service docker restart` and build docker image with `setcap` agian.


## Tags

* `latest` tracks the `edge` tag from [upstream](https://hub.docker.com/r/_/alpine/)
* `e350` indicates the os version of the edge, i.e, `e350=edge@3.5.0`

_This includes the `main`, `testing`, and `community` repositories, but testing packages are masked. To install them, please use `apk-install pkgname@testing`._

# License
[Apache 2.0](https://www.tldrlegal.com/l/apache2)
