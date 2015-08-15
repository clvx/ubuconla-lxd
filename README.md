# ubuconla-lxd
Ejemplos de LXD de la charla LXD Internals del UbuCon LA 2015

---

## Namespaces

### Mount

```
/usr/bin/lxc start [container]
/usr/bin/lxc exec [container] /bin/bash 
  cd /tmp/
  /bin/mount -n -o size=1m -t tmpfs tmpfs `mktemp -d --tmpdir=/tmp`
```

En el container:
`/bin/cat /proc/self/mounts`

En el host:
`/bin/cat /proc/self/mounts`

Los puntos de montaje son diferentes para cada namespace. En el namespace del contenedor se encuentra montado el temp filesystem y en el host no.

### PID

En el host:
```
/usr/bin/lxc start [container]
/usr/bin/lxc info [container]
pid=`lxc info [container]  | grep Init | cut -d' ' -f2`
/bin/cat /proc/${pid}/status
/bin/ps afux | grep ${pid}
```

En el container:
```
/usr/bin/lxc exec [container] /bin/bash 
  /bin/ps afux 
  /bin/echo 'Notar el PID 1'
  /bin/cat /proc/1/status
```

Los PID son diferentes para cada espacio de nombre. Tener PID diferentes permite mover los contenedores entre hosts sin afectar a otros procesos en ejecución.

### UTS


