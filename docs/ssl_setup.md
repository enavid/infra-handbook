# Free SSL Certificate with Let's Encrypt (Wildcard + Manual DNS)

## Requirements

* Ubuntu 24 server
* A registered domain with DNS managed via a CDN/DNS panel (e.g. Abr Arvan)
* Certbot installed on the server

## Install Certbot

```bash
sudo apt update
sudo apt install certbot -y
```

---

## Option 1 - Manual (One-Time Setup)

Use this method if you want to obtain the certificate manually without configuring automatic renewal.

```bash
sudo certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d yourdomain.com \
  -d *.yourdomain.com
```

After running the command, Certbot will prompt you to create a DNS TXT record:

```
_acme-challenge.yourdomain.com  ->  <value provided by certbot>
```

1. Log in to your DNS panel.
2. Add the TXT record with the name `_acme-challenge.yourdomain.com` and the given value.
3. Wait a few minutes for DNS propagation.
4. Press Enter in the terminal to continue.

### Certificate file locations

```
/etc/letsencrypt/live/yourdomain.com/fullchain.pem
/etc/letsencrypt/live/yourdomain.com/privkey.pem
```

---

## Option 2 - Automatic Renewal with DNS Plugin

For wildcard certificates, automatic renewal requires a DNS plugin that can update TXT records via your DNS provider's API.

### Step 1 - Install the DNS plugin

If your DNS provider is supported (e.g. Cloudflare, DigitalOcean), install the corresponding plugin:

```bash
# Example for Cloudflare
sudo apt install python3-certbot-dns-cloudflare -y
```

### Step 2 - Create an API credentials file

```bash
sudo mkdir -p /etc/letsencrypt/secrets
sudo nano /etc/letsencrypt/secrets/dns-credentials.ini
```

Add your API credentials (format depends on the plugin):

```ini
# Cloudflare example
dns_cloudflare_api_token = YOUR_API_TOKEN_HERE
```

Secure the file:

```bash
sudo chmod 600 /etc/letsencrypt/secrets/dns-credentials.ini
```

### Step 3 - Obtain the certificate

```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/secrets/dns-credentials.ini \
  -d yourdomain.com \
  -d *.yourdomain.com
```

### Step 4 - Test automatic renewal

```bash
sudo certbot renew --dry-run
```

Certbot installs a systemd timer or cron job automatically. You can verify it:

```bash
systemctl status certbot.timer
```

---

## Notes

* Certificates issued by Let's Encrypt are valid for 90 days.
* Automatic renewal triggers when fewer than 30 days remain.
* If your DNS provider does not have a Certbot plugin, you must use the manual method and renew manually every 90 days.
* After renewal, restart any services (e.g. Nginx, Apache) that use the certificate.

```bash
sudo systemctl reload nginx
```
