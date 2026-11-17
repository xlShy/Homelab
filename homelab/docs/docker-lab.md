# Docker Lab (Docker Engine + Compose on Ubuntu Server)

## Goal
Learn Docker hands-on in a safe test VM: install it, run containers, understand
ports, logs, and volumes, then run a small multi-container app with Docker Compose.
This prepares me to run self-hosted services (for example a photo app or a password
manager) on my home server later.

## Environment
- Ubuntu Server 26.04 LTS VM in VirtualBox (4 GB RAM, 2 vCPU, 40 GB dynamically allocated disk)
- Bridged network adapter, so my PC's browser can reach the containers
- Docker Engine and the Compose plugin, installed from Docker's official apt repository
- A separate VM from my file server, so mistakes while learning can't affect it

## What I did
1. **Built the lab VM.** Installed Ubuntu Server with OpenSSH and managed it from
   PowerShell over SSH.
2. **Installed Docker** from Docker's own repository: added its signing key and
   source list, then installed `docker-ce`, the CLI, and the Compose plugin. Verified
   with `docker run hello-world`.
3. **Ran a web server in a container** (`nginx`) and used the core commands:
   `docker run -d --name web -p 8080:80`, `docker ps`, `docker logs`, and
   `docker exec` to look inside the container.
4. **Used a bind-mount volume** to serve my own page from a folder on the VM
   (`-v ~/site:/usr/share/nginx/html:ro`). Edited the file and saw the change
   immediately, with no container restart.
5. **Ran a three-container stack with Docker Compose:** Nginx (web), MariaDB (database),
   and Adminer (a web interface for the database), all defined in one `compose.yaml`.
   Adminer connected to the database by its service name over Compose's private network.
6. **Tested persistence.** Created a table with rows in MariaDB, ran
   `docker compose down` and `docker compose up -d`, and confirmed the data survived,
   because it lives in a named volume instead of the container.

### The Compose file (simplified)
```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./site:/usr/share/nginx/html:ro
    restart: unless-stopped

  db:
    image: mariadb:lts
    environment:
      MARIADB_ROOT_PASSWORD: change-me      # lab only, see security notes
      MARIADB_DATABASE: practice
    volumes:
      - dbdata:/var/lib/mysql
    restart: unless-stopped

  adminer:
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - db
    restart: unless-stopped

volumes:
  dbdata:
```

## Security notes
- The database service has **no published port**, so only the other containers can
  reach it, not the rest of the network.
- The password in the Compose file is for this lab only. For real services, secrets
  belong in a separate `.env` file that is kept out of Git.
- Membership in the `docker` group is effectively root access, so it should be kept small.
- Ports published with `-p` bypass `ufw` rules, because Docker edits the firewall
  directly. Worth remembering before running anything on a real server.
- Only pulled images from official sources (`nginx`, `mariadb`, `adminer`).

## Troubleshooting log
| Problem | Cause | Fix |
|---|---|---|
| VM looked frozen at "Loading essential drivers" | Slow first load from a hard drive, and the installer menu was waiting for input | Waited, then pressed Enter at the boot menu |
| Root filesystem was only about 19 GB on a 40 GB disk | The installer's default LVM layout leaves half the disk unallocated | `sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv` |
| `docker: command not found` while following the steps | I was still inside the container's shell (the prompt showed a container ID), where Docker isn't installed | Typed `exit` to return to the VM, and learned to check the prompt first |
| `docker compose up` seemed to hang, then failed with "failed to copy" | A large image download was interrupted partway | Re-ran the same command, which resumed from the layers already downloaded |

## What I learned
- The difference between an image and a container, and why containers are disposable
- Port mapping, reading container logs, and opening a shell inside a container
- Bind mounts vs named volumes, and why persistent data belongs in volumes
- Describing a multi-service app in one Compose file, and how services reach each
  other by name on a private network
- Recognizing which machine I'm typing on (VM, container, or my PC) from the prompt
- Retrying interrupted image pulls instead of starting over

## Possible next steps
- Run a real self-hosted application with Compose, using its official documentation
- Include container volumes in my backup routine
