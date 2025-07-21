# 📬 Mailserver Deployment Guide

This guide provides complete instructions for deploying and maintaining a secure, production-ready mailserver using Docker Compose. It includes setup, configuration, DNS records, testing, and ongoing maintenance.

---

## 📑 Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Folder Structure](#folder-structure)  
3. [Step 1: Get the Project](#step-1-get-the-project)  
4. [Step 2: Configure Environment & Files](#step-2-configure-environment--files)  
5. [Step 3: Set Up DNS Records](#step-3-set-up-dns-records)  
6. [Step 4: Start the Mailserver](#step-4-start-the-mailserver)  
7. [Step 5: Verify the Service](#step-5-verify-the-service)  
8. [Step 6: Access & Test](#step-6-access--test)  
9. [Step 7: Maintenance](#step-7-maintenance)  
10. [Troubleshooting](#troubleshooting)  
11. [Tools & References](#tools--references)  

---

## ✅ Prerequisites

Make sure you have:

- A **registered domain name** (e.g., `example.com`) with DNS access
- A **VPS or server** with a **public static IP (Not obligatory, but highly recommended)**
- **Docker** (v20.10+): [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** (v1.29+): [Install Docker Compose](https://docs.docker.com/compose/install/)
- Open firewall ports **25**, **143**, **587**, and **993**
- Basic Linux/command-line skills

---

## 📁 Folder Structure

Ensure your directory looks like this:

```
mailserver/                   # Project root
├── docker-compose.yml        # Main Docker Compose file
├── config/                   # Configuration files
│   ├── certs/                # SSL certificates (manual)
│   ├── opendkim/             # DKIM configuration
│   └── mailserver.env        # Mailserver environment settings
├── data/                     # Persistent data storage
│   └── state/                # Mail state folders (Postfix, Dovecot, etc.)
└── README.md                 # This documentation
```

---

## 🛠️ Step 1: Get the Project

Clone or upload the project to your server:

```bash
git clone https://github.com/propstgonz/fast-dockerized-mailserver
cd mailserver
```

---

## ⚙️ Step 2: Configure Environment & Files

### SSL Certificates

If using manual SSL (recommended for production):

1. Place your certificates in `config/certs/`:
   - `fullchain.pem`
   - `server.key`
2. Ensure your `docker-compose.yml` mounts them correctly:
   ```yaml
   - ./config/certs/fullchain.pem:/etc/ssl/certs/fullchain.pem:ro
   - ./config/certs/server.key:/etc/ssl/private/server.key:ro
   ```

### 🔧 Environment Variables

The environment variables for the mailserver are defined in the `config/mailserver.env` file and in the `docker-compose.yml` file under the `environment:` section.

> 🔁 **Important:**  
> Replace all instances of `{your-domain}` with your actual domain name (e.g. `example.com`) in both `docker-compose.yml` and `mailserver.env`.  
>  
> For example:
> ```yaml
> domainname: mail.{your-domain}
> ```
> should become:
> ```yaml
> domainname: mail.example.com
> ```

#### Example `mailserver.env`

```env
ENABLE_CLAMAV=1
ENABLE_SPAMASSASSIN=1
ENABLE_FAIL2BAN=1
PERMIT_DOCKER=host
POSTMASTER_ADDRESS=postmaster@example.com
```

These variables control features like antivirus scanning, spam filtering, and access control. For a complete list of supported variables, refer to the [official environment variable documentation](https://docker-mailserver.github.io/docker-mailserver/edge/config/environment/).

### Add Users

Add users via Docker:

```bash
docker exec -it mailserver setup email add user@example.com strongpassword
```

Passwords are stored encrypted.

---

## 🌐 Step 3: Set Up DNS Records

Configure the following DNS records **at your domain provider**:

### 1. MX Record

| Type | Name | Value               | Priority |
|------|------|---------------------|----------|
| MX   | @    | `mail.example.com.` | 10       |

### 2. A Record

| Type | Name  | Value                 |
|------|-------|-----------------------|
| A    | mail  | Your server public IP |

### 3. SPF Record

| Type | Name | Value                  |
|------|------|------------------------|
| TXT  | @    | `v=spf1 mx ~all`       |

### 4. DKIM Record (Optional, Recommended)

1. Generate DKIM keys:
   ```bash
   docker exec -it mailserver setup config dkim
   ```
2. Find the generated key in `config/opendkim/keys/`
3. Add a TXT record:
   | Type | Name                        | Value                         |
   |------|-----------------------------|-------------------------------|
   | TXT  | `mail._domainkey`        | `v=DKIM1; k=rsa; p=MIIBIjAN...` |

### 5. DMARC Record (Optional)

| Type | Name   | Value                                                |
|------|--------|------------------------------------------------------|
| TXT  | _dmarc | `v=DMARC1; p=none; rua=mailto:admin@example.com`     |

---

## 🚀 Step 4: Start the Mailserver

Launch the container:

```bash
docker-compose up -d
```

This will start all services defined in your `docker-compose.yml`.

---

## 📋 Step 5: Verify the Service

Check logs to verify everything is running correctly:

```bash
docker-compose logs -f
```

Look for errors in Postfix, Dovecot, DKIM, and SSL setup.

---

## 📧 Step 6: Access & Test

### Sending & Receiving Mail

- Send a test email **to and from** your mailserver address using:
  - Webmail (if configured) or
  - An email client (e.g., Thunderbird)

### Email Client Settings

| Protocol | Host              | Port | Encryption |
|----------|-------------------|------|------------|
| SMTP     | mail.example.com  | 587  | STARTTLS   |
| IMAP     | mail.example.com  | 993  | SSL/TLS    |

### External Testing Tools

- [Mail Tester](https://www.mail-tester.com)
- [MXToolbox](https://mxtoolbox.com)
- [DNS Checker](https://dnschecker.org)

---

## 🔄 Step 7: Maintenance

### Backups

Regularly back up the `data/` directory:

```bash
tar -czf backup-$(date +%F).tar.gz ./data
```

### Updates

Keep the image up to date:

```bash
docker compose pull
docker compose up -d
```

### SSL Certificate Renewal

If you're using Let's Encrypt (not covered here), automate renewal via cron + script.

---

## 🛠️ Troubleshooting

- **Ports already in use:** Check for conflicts using `netstat` or `ss`.
- **Firewall blocking:** Ensure ports 25, 587, 993, and 143 are open.
- **DNS propagation delay:** It may take hours for DNS changes to apply.
- **Mail delivery issues:** Use tools like Mail Tester to debug SPF/DKIM/DMARC issues.

---

## 🔗 Tools & References

- 📄 [Docker Mailserver Docs](https://docker-mailserver.github.io/docker-mailserver/)
- 📄 [Postfix Docs](http://www.postfix.org/documentation.html)
- 📄 [Dovecot Docs](https://doc.dovecot.org/)
- 🔒 [Let’s Encrypt](https://letsencrypt.org/)
- 🧪 [MXToolbox](https://mxtoolbox.com/)
- 🧪 [Mail Tester](https://www.mail-tester.com)

---

## ✅ Final Notes

- Always test your configuration with **external tools** to ensure good deliverability.
- Consider setting `p=quarantine` or `p=reject` in your DMARC record once you're confident.
- If you're unsure about DNS, use [dnschecker.org](https://dnschecker.org) to verify global propagation.
