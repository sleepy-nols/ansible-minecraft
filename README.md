# ansible-minecraft


**This is a fork from [github.com/nolte/ansible-minecraft](https://github.com/nolte/ansible-minecraft)**

This role installs and configures different Minecraft servers on Debian.

## Features

**Currently supported servers:**
- Vanilla
- Paper
- Spigot

Other features:
-  safely stops the server using [stop](http://minecraft.gamepedia.com/Commands#stop) when running under **systemd**
-  manages user ACLs
-  manages Bukkit/Spigot Plugins
-  manages ``server.properties``
-  hooks: include arbitrary tasks at specific stages during execution

## Active development
This role is being actively developed.
**Goals:**
- Paper support
- Forge support

## Out of Role Scope

- install a *Java Runtime*, this must be done, before you use this Role, you can use [nolte/ansible-role-msopenjdk](https://github.com/nolte/ansible-role-msopenjdk) for example.
- executing backups and recovery
- healthy checks like [Minecraft-Region-Fixer](https://github.com/Fenixin/Minecraft-Region-Fixer)
- handle utility services like [filebeat](https://www.elastic.co/de/products/beats/filebeat) or [prometheus](https://github.com/prometheus/node_exporter)
- install additional Tools like [rcon-cli](https://github.com/itzg/rcon-cli).

**All of this is needet but not a part of this role!**, _you will find examples at [nolte/minecraft-infrastructure](https://github.com/nolte/minecraft-infrastructure)._

## Install

```
   ansible-galaxy install nols.minecraft
```

or add this to your ``requirements.yml``

```
- name: nolte.minecraft
```

and execute ``ansible-galaxy install -r requirements.yml``

## Use

```
  - hosts: minecraft
    roles:
       - nols.minecraft
```

## Requirements

-  Python 3.x on the Ansible control machine to generate user ACLs
-  Ansible 2.7.0+ on the control machine to fetch the Minecraft version
-  Existing Compatible Java Runtime for start and install Minecraft on target System.


## Contributing

Bug reports and contributions are welcome :) 

## License

Apache 2.0

## Disclaimer

To execute an automatic installation you must accept the [Minecraft EULA](https://account.mojang.com/documents/minecraft_eula). Be aware that by using this role, you implicitly accept the same EULA.
You can handle the acception by using a Environment Property like: ``export mc_accept_eula=true`` the default is ``false`` for disagree.
