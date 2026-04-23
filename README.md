# DSpace 9 — Azure Marketplace Documentation

> **⚠️ IMPORTANT:** This offer is currently undergoing the Azure Marketplace certification process and will be available soon. This documentation is provided in advance to help you prepare your deployment.

Post-deployment guide for the **DSpace 9 — Institutional Repository** offer on Azure Marketplace, published by Cotechnoe.

> **Official DSpace documentation:** [wiki.lyrasis.org/display/DSDOC9x](https://wiki.lyrasis.org/display/DSDOC9x)

---

## What this offer includes

- DSpace 9.x fully installed and configured on **Ubuntu 22.04 LTS** — ready to use immediately
- **HTTPS enabled out of the box** (self-signed certificate, replaceable with your own)
- DSpace **REST API backend** + **Angular-based frontend** pre-configured and operational
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

### 2. Set the DSpace administrator password

Once connected, create your DSpace admin account:

```bash
sudo /opt/dspace/bin/dspace create-administrator
```

You will be prompted for:
- Email address (this becomes your admin login)
- First and last name
- Password

> The admin web interface is then accessible at **https://\<your-ip\>/login**.

---

### 3. Verify the services are running

```bash
sudo systemctl status dspace-backend
sudo systemctl status dspace-frontend
sudo systemctl status nginx
sudo systemctl status postgresql
sudo systemctl status solr
```

All five services should be `active (running)`.

---

### 4. Access the web interface

| Interface | URL |
|-----------|-----|
| Frontend (public) | `https://<your-ip>/` |
| Admin UI | `https://<your-ip>/login` |
| REST API | `https://<your-ip>/server` |
| Solr Admin (local only) | `http://localhost:8983/solr` |

> **Note:** The default SSL certificate is self-signed. Your browser will display a security warning until you replace it with a valid certificate (see step 6 below).

---

### 5. Configure your domain name (optional)

If you have a domain name pointing to your VM's public IP:

1. Update the DSpace backend configuration:
   ```bash
   sudo nano /opt/dspace/config/local.cfg
   ```
   Edit the following lines:
   ```
   dspace.ui.url = https://your-domain.example.org
   dspace.server.url = https://your-domain.example.org/server
   ```

2. Update the Nginx server name:
   ```bash
   sudo nano /etc/nginx/sites-available/dspace
   ```
   Replace `server_name _;` with `server_name your-domain.example.org;`

3. Restart the services:
   ```bash
   sudo systemctl restart dspace-backend dspace-frontend nginx
   ```

---

### 6. Replace the self-signed SSL certificate

To install a certificate from Let's Encrypt (free, recommended):

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.example.org
```

Certbot will automatically update your Nginx configuration and set up auto-renewal.

For a manually issued certificate, place your files at:
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
sudo journalctl -u dspace-backend -f

# DSpace frontend (Angular SSR)
sudo journalctl -u dspace-frontend -f

# Nginx access and error logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

---

## Troubleshooting

### Browser shows "502 Bad Gateway"

The DSpace backend takes 3–5 minutes to fully start after the VM boots. Wait a few minutes, then refresh. Check status with:
```bash
sudo systemctl status dspace-backend
sudo journalctl -u dspace-backend --no-pager -n 50
```

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
| DSpace GitHub Repository | [github.com/DSpace/DSpace](https://github.com/DSpace/DSpace) |
| Release Notes | [github.com/DSpace/DSpace/releases](https://github.com/DSpace/DSpace/releases/tag/dspace-9.2) |
| Community Forum | [groups.google.com/g/dspace-tech](https://groups.google.com/g/dspace-tech) |
| LYRASIS Community | [lyrasis.org](https://lyrasis.org) |

---

## About this offer

This Azure Marketplace offer is published and maintained by **Cotechnoe**.  
Source and documentation: [github.com/Cotechnoe/dspace9-azure-marketplace-docs](https://github.com/Cotechnoe/dspace9-azure-marketplace-docs)

DSpace is free open source software distributed under the **BSD 3-Clause License** and governed by the LYRASIS community.  
Azure VM infrastructure costs apply based on your subscription and selected VM size.

