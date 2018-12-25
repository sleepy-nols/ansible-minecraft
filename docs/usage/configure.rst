Configure the Role
=============================

.. code-block:: bash

    [root@centos7 minecraft]# tree -L 3
    .
    |-- plugins
    |   |-- current -> /opt/minecraft/plugins/releases/minimal
    |   |-- releases
    |   |   `-- minimal
    |   `-- shared
    |       |-- PermissionsEx
    |       |-- PermissionsEx-1.23.4.jar -> /opt/minecraft/plugins/releases/minimal/PermissionsEx-1.23.4.jar
    |       |-- PluginMetrics
    |       |-- TNE.jar -> /opt/minecraft/plugins/releases/minimal/TNE.jar
    |       |-- TheNewEconomy
    |       |-- Updater
    |       |-- Vault
    |       |-- Vault.jar -> /opt/minecraft/plugins/releases/minimal/Vault.jar
    |       `-- bStats
    `-- server
        |-- current -> /opt/minecraft/server/releases/1.9
        |-- releases
        |   `-- 1.9
        `-- shared
            |-- ...
            |-- plugins -> /opt/minecraft/plugins/shared
            |-- spigot.jar -> /opt/minecraft/server/current/spigot-1.9.jar
            `-- ...


Configure the Server
-----------------------------

Role variables
`````````````````````````````````````````````````````````````````

The following variable defaults are defined in ``defaults/main.yml``.

``minecraft_version``
   Minecraft version to install (default: ``latest``)

   Examples:

   .. code:: yaml

       minecraft_version: latest
       minecraft_version: 1.10
       minecraft_version: 1.9.1
       minecraft_version: 16w21a

``minecraft_eula_accept``
   accept the Minecraft eula License, must accepted by the Role User (default: ``false``)

``minecraft_url``
   Minecraft download URL (default:
   ``https://s3.amazonaws.com/Minecraft.Download/versions``)

``minecraft_user``
   system user Minecraft runs as (default: ``{{ minecraft_server }}``)

``minecraft_group``
   system group Minecraft runs as (default: ``{{ minecraft_server }}``)

``minecraft_home``
   directory to install Minecraft to (default: ``/opt/minecraft``)

``minecraft_max_memory``
   Java max memory (``-Xmx``) to allocate (default: ``1024M``)

``minecraft_initial_memory``
   Java initial memory (``-Xms``) to allocate (default: ``1024M``)

``minecraft_service_name``
   systemd service name or Supervisor program name (default: ``minecraft``)

``minecraft_supervisor_name``
   **DEPRECATED:** Supervisor program name (default: ``{{ minecraft_service_name }}``)

``minecraft_process_control``
   Choose between ``systemd`` and ``supervisor`` (default: ``systemd``).

``minecraft_whitelist``
   list of Minecraft usernames to whitelist (default: ``[]``)

``minecraft_ops``
   list of Minecraft usernames to make server ops (default: ``[]``)

``minecraft_banned_players``
   list of Minecraft usernames to ban (default: ``[]``)

``minecraft_banned_ips``
   list of IP addresses to ban (default: ``[]``)

``minecraft_server_properties``
   dictionary of server.properties entries (e.g. ``server-port: 25565``) to set (default: ``{}``)

``minecraft_server``
  choose between ``minecraft`` or ``spigot`` (default: ``minecraft``)

``minecraft_server_java_ops``
   additional java ops like remote debug ``-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005`` (default: *none*)

``minecraft_java_external_managed``
  used for skipping the java installation tasks from this role, for handle Java by external scripts/roles (default: ``false``)



Hooks and run stages
`````````````````````````````````````````````````````````````````

**ansible-minecraft** organizes execution into a number of run stages:

``setup``
   -  install prerequisites (e.g., Java)
   -  create Minecraft user and group

``download``
   -  fetch the latest version of from the launcher API
   -  download Minecraft

``install``
   -  symlink version to ``minecraft_server.jar``
   -  agree to EULA

``acl``
   -  configure server ACLs (whitelist, banned players, etc.)

``configure``
   -  set ``server.properties``

``start``
   -  (re)start server

You can execute custom tasks before or after specific stages. Simply specify a `task include file <https://docs.ansible.com/ansible/playbooks_roles.html#task-include-files-and-encouraging-reuse>`__ using the relevant role variable:

.. code:: yaml

    - hosts: minecraft
      roles:
        - role: devops-coop.minecraft
          minecraft_hook_before_start: "{{ playbook_dir }}/download-world-from-s3.yml"

The available hooks are:

``minecraft_hook_before_setup``
   run before ``setup`` tasks

``minecraft_hook_after_setup``
   run after ``setup`` tasks

``minecraft_hook_before_download``
   run before ``download`` tasks

``minecraft_hook_after_download``
   run after ``download`` tasks

``minecraft_hook_before_install``
   run before ``install`` tasks

``minecraft_hook_after_install``
   run after ``install`` tasks

``minecraft_hook_before_start``
   run before ``start`` tasks

``minecraft_hook_after_start``
   run after ``start`` tasks

Example
`````````````````````````````````````````````````````````````````

.. code:: yaml

    - hosts: minecraft
      roles:
         - { role: nolte.ansible_minecraft, minecraft_whitelist: ["jeb_", "dinnerbone"]}




Install Plugins
-------------------------------

| The Problems by many plugins is the copatibility.
| The most plugins have a ``*.jar``` and a configuration Folder.
| Plugin updates shoud only change the used ``*.jar``.
| The config of a plugin will be placed under ``plugins/shared``.
| the jars placed under ``plugins/releases/{pluginsets}/*.jar`` and will finaly link to ``plugins/shared``

| finaly the ``plugins/shared`` will be linked to ``server/shared/plugins``
| all pluginruntime data of your server will be stored under ``plugins/shared``

**example config:**

.. code-block:: yaml

    minecraft_plugins_set_version: "minimal"
    minecraft_plugin_sets:
      minimal:
        vault:
          src: https://media.forgecdn.net/files/2615/750/Vault.jar
        permissionsEx:
          src: https://media.forgecdn.net/files/909/154/PermissionsEx-1.23.4.jar
          config:
            - src: config_permissionex.yml.j2
              dest: PermissionsEx/config.yml
        tne:
          src: https://github.com/TheNewEconomy/TNE-Bukkit/releases/download/Beta-1.1.3/Beta.1.1.3.zip
          type: "archive"

Configure Plugin
`````````````````````````````````````````````````````````````````

| To automatical configure a plugin create a ``Jinja`` templatefile at your Playbook ``templates`` folder, and add a ``config:`` entry.
| The ``dest:`` path is relative to the ``plugins/shared`` folder.