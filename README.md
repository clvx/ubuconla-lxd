# ubuconla-lxd
Ejemplos de LXD de la charla LXD Internals del UbuCon LA 2015.


## LXD
Para ejecutar estos ejemplos, desde Ubuntu Vivid(15.10):

#### Instalación
```
add-apt-repository ppa:ubuntu-lxc/lxd-git-master
apt update; apt install lxd
```

#### Creación
```
lxc remote add lxc-img images.linuxcontainers.org
lxc image copy lxc-img:/ubuntu/trusty/amd64 local: --alias=trusty-amd64
lxc launch trusty-amd64 base
```

#### Ejecución
```
lxc start base
```

#### Detención
```
lxc stop base
```

---

## Namespaces

#### Mount

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

#### PID

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

#### UTS

En el host:
```
/bin/cat /etc/hostname
```

En el contenedor:
```
/usr/bin/lxc start [container]
/usr/bin/lxc exec [container] /bin/bash 
   /bin/cat /etc/hostname
```

Ambos contenedores tienen nombres diferentes.

#### Network

En el host:
```
/sbin/ip addr show              
```
- Muestra los dispositivos de red activos. lxcbr0 es el bridge en el network namespace del host donde se conectan todos los contenedores.
- vethXXX se puede apreciar cuando hay un contenedor encendido. Es el punto conectado a lxcbr0, el otro punto se encuentra conectado en el contenedor.

```
/sbin/ip route show
```
- La tabla de enrutamiento del host.

```
/sbin/iptables -t nat -L POSTROUTING
```
- Enmascara toda la red interna del contenedor con la dirección de salida.

```
/bin/ss -tulnp
```
- Los sockets de tcp/udp activos en el host.


En el container:
```
/usr/bin/lxc start [container]
/usr/bin/lxc exec [container] /bin/bash
    /sbin/ip addr show  
    /sbin/ip route show
    /bin/ss -tulnp
```
- ip addr muestra solo eth0. eth0 es la interface virtual del contenedor que se conecta a lxcbr0.
- La tabla de enrutamiento difiere del host.
- Los sockets tcp/udp activos del contenedor, ya que es un network namespace diferente se pueden crear sockets en puertos privilegiados.

En general, cada network namespace provee toda una pila de red aislada.

#### USER
