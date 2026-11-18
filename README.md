# Gitea Docker Installation (Homelab)

Gitea on a Proxmox Docker VM, with all data on a Synology NAS:

- Repositories, config (`app.ini`), avatars and attachments → Synology over NFS
- Database → PostgreSQL running on the Synology
- Portainer Agent (port 9001) so a Portainer server can manage this host

## Prerequisites
1. On the VM:
   ```bash
   sudo apt install nfs-common
   ```
2. On the Synology:
   - Create the folder `data` under `NFS_PATH` (default `/volume1/docker/gitea`).
   - NFS permission rule on the share for the VM's IP: Read/Write, Security `sys`,
     Squash "No mapping", "Allow users to access mounted subfolders" ticked.
3. In PostgreSQL (pgAdmin Query Tool):
   ```sql
   CREATE USER gitea WITH PASSWORD '...';
   CREATE DATABASE gitea OWNER gitea;
   ```

## Install
```bash
git clone <this repo> gitea-install
cd gitea-install
cp .env.example .env
chmod 600 .env
nano .env        # NAS_IP, DB_PASSWORD, DB_PORT, GITEA_DOMAIN, GITEA_ROOT_URL
sudo docker compose up -d
sudo docker logs gitea --tail 30
```

Open `GITEA_ROOT_URL` (e.g. `http://<vm-ip>:3000`). The database fields on the
setup page are pre-filled from `.env`; create the admin account there and click
**Install Gitea**.

## Notes
- `DB_PASSWORD` is a plain password here (not URL-encoded, unlike Bambuddy's `DATABASE_URL`).
- Git over SSH uses port `2222`: `git clone ssh://git@<vm-ip>:2222/<user>/<repo>.git`
- Only one Portainer Agent per Docker host: if this VM already runs one,
  remove the `portainer_agent` service from `docker-compose.yml`.
- Deploy from the shell, not as a Portainer stack (the agent would restart itself).
