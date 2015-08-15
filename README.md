# ubuconla-lxd
Ejemplos de LXD de la charla LXD Internals del UbuCon LA 2015

Namespaces
---

Mount
--

```
/usr/bin/lxc start [container]
/usr/bin/lxc exec [container] /bin/bash 
  cd /tmp/
  /bin/mount -n -o size=1m -t tmpfs tmpfs `mktemp -d --tmpdir=/tmp`
```

en el container:
`/bin/cat /proc/self/mounts`

en el host:
`/bin/cat /proc/self/mounts`


