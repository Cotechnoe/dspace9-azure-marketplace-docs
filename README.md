# DSpace 9 — Azure Marketplace Documentation

Post-deployment guide for the **DSpace 9 — Institutional Repository** offer on Azure Marketplace, published by Cotechnoe.

> **Official DSpace documentation:** [wiki.lyrasis.org/display/DSDOC9x](https://wiki.lyrasis.org/display/DSDOC9x)

---

## What this offer includes

- DSpace 9.x fully installed and configured on **Ubuntu 22.04 LTS** — ready to use immediately
- **HTTPS enabled out of the box** (self-signed certificate, automatically replaced by Let's Encrypt when a domain name is available)
- DSpace **REST API backend** (Tomcat) + **Angular-based frontend** pre-configured and operational
- **PostgreSQL** database included on the same VM — no external database required
- **Apache Solr** search engine pre-configured for full-text and metadata indexing
- Administrator web interface accessible immediately after deployment

---

## First steps after deployment

### 1. Connect via SSH

```bash
ssh <admin-username>@<your-vm-ip>
```

Replace `<admin-username>` with the username you provided at deployment time, and `<your-vm-ip>` with the public IP address assigned to your VM.

---

### 2. Create the DSpace administrator account

Once connected, create your DSpace admin account:

```bash
sudo /opt/dspace/bin/dspace create-administrator
```

You will be prompted for:
- Email address (this becomes your admin login)
- First and last name
- Password

> The admin web interface is then accessible at **https://\<your-ip\>/login**.

> For more information on this command, see the official DSpace documentation: [Command Line Operations](https://wiki.lyrasis.org/spaces/DSDOC9x/pages/379126827/Command+Line+Operations)

---

### 3. Verify the services are running

```bash
sudo systemctl status tomcat
sudo systemctl status dspace-frontend
sudo systemctl status nginx
sudo systemctl status postgresql
sudo systemctl status solr
```

All five services should be `active (running)`.

> **Note:** The Tomcat service (`tomcat`) hosts the DSpace REST API backend. Allow 3–5 minutes after VM boot for it to fully initialize before accessing the web interface.

---

### 4. Access the web interface

| Interface | URL |
|-----------|-----|
| Frontend (public) | `https://<your-ip>/` |
| Admin UI | `https://<your-ip>/login` |
| REST API | `https://<your-ip>/server` |
| Solr Admin (local only) | `http://localhost:8983/solr` |

> **Note:** The default SSL certificate is self-signed. Your browser will display a security warning until it is replaced by a Let's Encrypt certificate (done automatically when a domain name is available — see step 5 below).

---

### 5. Domain name and SSL certificate

Domain name configuration and SSL certificate issuance are performed **automatically** during the first boot of the VM.

**What happens automatically at first boot:**

- DSpace `local.cfg` is updated with the correct public URL (`dspace.server.url`, `dspace.ui.url`)
- Nginx is reconfigured with the correct server name
- If a valid FQDN is available and DNS resolves, a **Let's Encrypt certificate** is requested automatically via `certbot certonly --webroot` and nginx is reloaded to use it
- Automatic certificate renewal is enabled via `certbot.timer`

**If DNS was not yet assigned at first boot** (e.g., the Azure DNS label was configured after the VM started), the `dspace-dns-watch.timer` systemd timer monitors for FQDN availability and applies domain configuration and Let's Encrypt automatically once DNS resolves.

**To check the SSL configuration status:**

```bash
sudo cat /root/.dspace-credentials
```

This file records whether Let's Encrypt was successfully obtained (`SSL_TYPE=letsencrypt`) or whether a self-signed certificate is still in use.

**To view the first-boot configuration log:**

```bash
sudo cat /var/log/dspace-firstboot.log
```

#### Providing your own certificate (alternative to Let's Encrypt)

If you prefer to use a certificate from your own CA or a commercial provider, place your files at:

- Certificate: `/etc/nginx/ssl/dspace.crt`
- Private key: `/etc/nginx/ssl/dspace.key`

Then reload Nginx:

```bash
sudo systemctl reload nginx
```

---

## Administration

### Rebuild the Solr search index

If search results are incomplete or missing after importing content:

```bash
sudo /opt/dspace/bin/dspace index-discovery -b
```

See [Discovery](https://wiki.lyrasis.org/spaces/DSDOC9x/pages/379126068/Discovery) in the official DSpace documentation for details.

### Create an additional administrator

```bash
sudo /opt/dspace/bin/dspace create-administrator
```

### Backup the PostgreSQL database

```bash
sudo -u postgres pg_dump dspace > dspace-backup-$(date +%Y%m%d).sql
```

### View service logs

```bash
# DSpace backend (Tomcat)
sudo journalctl -u tomcat -f

# DSpace frontend (Angular SSR)
sudo journalctl -u dspace-frontend -f

# Nginx access and error logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# First-boot configuration log
sudo tail -f /var/log/dspace-firstboot.log
```

---

## Troubleshooting

### Browser shows "502 Bad Gateway"

The DSpace backend (Tomcat) takes 3–5 minutes to fully start after the VM boots. Wait a few minutes, then refresh. Check status with:

```bash
sudo systemctl status tomcat
sudo journalctl -u tomcat --no-pager -n 50
```

### SSL certificate is still self-signed after assigning a domain name

The `dspace-dns-watch.timer` handles this automatically. Check its status and logs:

```bash
sudo systemctl status dspace-dns-watch.timer
sudo cat /var/log/dspace-dns-watch.log
```

If Let's Encrypt was not yet obtained, the log will explain why (DNS not yet propagated, certificate request failed, etc.).

### SSH connection refused after deployment

Azure may take a few minutes to complete provisioning. Wait 2–3 minutes after the VM shows "Running" in the portal before attempting to connect.

If the issue persists, use the **Azure Serial Console** in the portal to access the VM directly.

### Solr not indexing new items

Trigger a manual index rebuild:

```bash
sudo /opt/dspace/bin/dspace index-discovery -b
```

---

## DSpace documentation

| Resource | URL |
|----------|-----|
| DSpace 9 Documentation | [wiki.lyrasis.org/display/DSDOC9x](https://wiki.lyrasis.org/display/DSDOC9x) |
| Command Line Operations | [wiki.lyrasis.org/…/Command+Line+Operations](https://wiki.lyrasis.org/spaces/DSDOC9x/pages/379126827/Command+Line+Operations) |
| System Administration | [wiki.lyrasis.org/…/System+Administration](https://wiki.lyrasis.org/spaces/DSDOC9x/pages/379126819/System+Administration) |
| Troubleshooting | [wiki.lyrasis.org/…/Troubleshooting+Information](https://wiki.lyrasis.org/spaces/DSDOC9x/pages/379126839/Troubleshooting+Information) |
| DSpace GitHub Repository | [github.com/DSpace/DSpace](https://github.com/DSpace/DSpace) |
| Release Notes | [github.com/DSpace/DSpace/releases/tag/dspace-9.2](https://github.com/DSpace/DSpace/releases/tag/dspace-9.2) |
| Community Forum | [groups.google.com/g/dspace-tech](https://groups.google.com/g/dspace-tech) |
| LYRASIS Community | [lyrasis.org](https://lyrasis.org) |

---

## About this offer

This Azure Marketplace offer is published and maintained by **Cotechnoe**.  
Source and documentation: [github.com/Cotechnoe/dspace9-azure-marketplace-docs](https://github.com/Cotechnoe/dspace9-azure-marketplace-docs)

DSpace is free open source software distributed under the **BSD 3-Clause License** and governed by the LYRASIS community.  
Azure VM infrastructure costs apply based on your subscription and selected VM size.

