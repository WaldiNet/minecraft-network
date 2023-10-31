<!-- omit from toc -->
# Minecraft Network
This repo contains the barebone structure of the WaldiCraft Server network.

It is intended to be re-usable and is able to grow with new instances.

<!-- omit from toc -->
## Table of contents
* [What's included?](#whats-included)
  * [Container](#container)
  * [Pre-selected Mods/Datapacks](#pre-selected-modsdatapacks)
* [How to Use](#how-to-use)


## What's included?
### Container
* 1x MC Proxy ([itzg/docker-bungeecord](https://github.com/itzg/docker-bungeecord/))
  * Container: `mc-proxy`
  * Currently only Velocity is implemented. If you want to use another Proxy you need to configure everything yourself
* 3x MC Server ([itzg/minecraft-server](https://docker-minecraft-server.readthedocs.io/))
  * Container: `mc-server-*`
* 3x MC Backup ([itzg/docker-mc-backup](https://github.com/itzg/docker-mc-backup))
  * Container: `mc-backup-*`
* **Optional Services:**
  * 1x Nginx *(for dynmap integration)*
  * 1x PHP *(for dynmap integration)*
  * 1x MySQL
  * 1x Redis
  * 1x CodeServer

### Pre-selected Mods/Datapacks
**>> IMPORTANT <<**<br>
To be sure what mods/datapacks are included by default please check the `.env` file carefully!

---

If you want to change the mods/datapacks you need to change the values inside the `.env` file for the variables ending with `_DATAPACKS`, `_MODRINTH_PROJECTS` and `_CURSEFORGE_FILES` per server.

The mods are listed according to the format descriped in the Docs: <br>
https://docker-minecraft-server.readthedocs.io/en/latest/mods-and-plugins/

## How to Use
**Prerequesits:** You need to have Docker installed on your host system.

1. Copy `.env.dist` to `.env`
2. **Read** the `.env` **carefully** (!!!) and change the values according to your needs
3. **Read** the config `./proxy/velocity/config/velocity.toml` **carefully** (!!!) and change the values according to your needs
4. **Optional:** If you have any mods that are not present on [Curseforge](https://www.curseforge.com/minecraft) or [Modrinth](https://modrinth.com/) you need to put the `.jar` files inside the correct server directory at `./server/**/mods/`
5. **Optional:** If needed put your custom configurations files into the correct config directories at `./server/**/config/`
6. **Optional:** If you want to have additional [Velocity plugins](https://hangar.papermc.io/?page=0&platform=VELOCITY&sort=-stars) put them into `./proxy/velocity/plugins/`
7. Once everything is configured you can run `docker compose up -d` to start all containers. This might take a while