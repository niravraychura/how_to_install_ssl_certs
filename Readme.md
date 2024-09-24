# Installing Let's Encrypt SSL Certificates on Linux Servers

*Author: Nirav Raychura*

---

## Introduction

Securing your website with an SSL certificate is essential for protecting user data and enhancing trust. **Let's Encrypt** is a free, automated, and open Certificate Authority that provides SSL/TLS certificates. This guide will walk you through the process of installing Let's Encrypt SSL certificates on popular Linux distributions using both **Apache** and **Nginx** web servers. We'll also cover how to set up automatic renewal of these certificates to ensure continuous security.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installing Certbot](#installing-certbot)
    - [Ubuntu/Debian](#ubuntu-debian)
    - [CentOS/RHEL/Amazon Linux](#centos-rhel-amazon-linux)
    - [Fedora](#fedora)
3. [Obtaining SSL Certificates](#obtaining-ssl-certificates)
    - [For Apache](#for-apache)
    - [For Nginx](#for-nginx)
4. [Automating Certificate Renewal](#automating-certificate-renewal)
    - [Test the Renewal Process](#test-the-renewal-process)
    - [Setting Up Automatic Renewal](#setting-up-automatic-renewal)
        - [Using Cron Jobs](#using-cron-jobs)
        - [Using Systemd Timers](#using-systemd-timers)
    - [Renewal Confirmation and Troubleshooting](#renewal-confirmation-and-troubleshooting)
5. [DNS Configuration](#dns-configuration)
6. [Conclusion](#conclusion)
7. [Additional Resources](#additional-resources)

---

## Prerequisites

Before starting, ensure you have the following:

- A **Linux server** (Virtual Machine on any cloud platform).
- A registered **domain name** pointing to your server's IP address.
- **Administrative (sudo) access** to the server.
- **Apache** or **Nginx** web server installed and running.
- Basic command-line knowledge.

---

## Installing Certbot

**Certbot** is a client that automates the process of obtaining and renewing Let's Encrypt SSL certificates. We'll install Certbot on various popular Linux distributions.

### Ubuntu/Debian

1. **Update the package list:**

   ```bash
   sudo apt update
   ```

2. **Install Certbot and the web server plugin:**

   - **For Apache:**

     ```bash
     sudo apt install certbot python3-certbot-apache
     ```

   - **For Nginx:**

     ```bash
     sudo apt install certbot python3-certbot-nginx
     ```

### CentOS/RHEL/Amazon Linux

1. **Enable the EPEL repository:**

   ```bash
   sudo yum install epel-release
   ```

2. **Install Certbot and the web server plugin:**

   - **For Apache:**

     ```bash
     sudo yum install certbot python-certbot-apache
     ```

   - **For Nginx:**

     ```bash
     sudo yum install certbot python-certbot-nginx
     ```

### Fedora

1. **Update the package list:**

   ```bash
   sudo dnf update
   ```

2. **Install Certbot and the web server plugin:**

   - **For Apache:**

     ```bash
     sudo dnf install certbot python3-certbot-apache
     ```

   - **For Nginx:**

     ```bash
     sudo dnf install certbot python3-certbot-nginx
     ```

---

## Obtaining SSL Certificates

Now that Certbot is installed, we'll obtain and install SSL certificates for your domain.

### For Apache

1. **Ensure Apache is running:**

   ```bash
   # Ubuntu/Debian
   sudo systemctl status apache2

   # CentOS/RHEL/Fedora/Amazon Linux
   sudo systemctl status httpd
   ```

2. **Run Certbot to obtain and install the certificate:**

   ```bash
   sudo certbot --apache
   ```

   - **Explanation:**
     - `--apache` tells Certbot to use the Apache plugin for authentication and installation.
     - Certbot will prompt you for:
       - **Email address** for urgent renewal and security notices.
       - **Terms of Service** agreement.
       - **Domain names** you want to secure (e.g., `example.com` and `www.example.com`).
       - **HTTP to HTTPS redirection** preference.
     - Certbot will automatically obtain and install the SSL certificate and adjust your Apache configuration.

3. **Verify the SSL certificate:**

   Open your website in a browser using `https://` and check for the secure lock icon.

### For Nginx

1. **Ensure Nginx is running:**

   ```bash
   sudo systemctl status nginx
   ```

2. **Run Certbot to obtain and install the certificate:**

   ```bash
   sudo certbot --nginx
   ```

   - **Explanation:**
     - `--nginx` tells Certbot to use the Nginx plugin for authentication and installation.
     - Certbot will prompt you for:
       - **Email address** for urgent renewal and security notices.
       - **Terms of Service** agreement.
       - **Domain names** you want to secure.
       - **HTTP to HTTPS redirection** preference.
     - Certbot will automatically obtain and install the SSL certificate and adjust your Nginx configuration.

3. **Verify the SSL certificate:**

   Open your website in a browser using `https://` and check for the secure lock icon.

---

## Automating Certificate Renewal

Let's Encrypt certificates are valid for **90 days**, but it's recommended to renew them every **60 days** to avoid any downtime. Certbot can automate the renewal process. We'll set up a **cron job** or **systemd timer** to handle automatic renewals.

### Test the Renewal Process

Before setting up automatic renewal, test the renewal process to ensure everything works correctly.

```bash
sudo certbot renew --dry-run
```

- **Explanation:**
  - The `--dry-run` flag simulates the renewal process without making any changes.
  - Check the output for any errors. If there are issues, resolve them before proceeding.

### Setting Up Automatic Renewal

Depending on your Linux distribution, you can use **cron jobs** or **systemd timers** to schedule automatic renewal.

#### Using Cron Jobs

**Cron** is a time-based job scheduler that runs tasks at specified intervals.

##### Create a Cron Job

1. **Open the root user's crontab:**

   ```bash
   sudo crontab -e
   ```

2. **Add the renewal command:**

   Append the following line to schedule the renewal to run twice daily:

   ```cron
   0 */12 * * * /usr/bin/certbot renew --quiet
   ```

   - **Explanation:**
     - `0 */12 * * *` schedules the job to run at minute 0 every 12 hours.
     - `/usr/bin/certbot renew --quiet` runs the Certbot renewal command silently.
     - Adjust the path to `certbot` if it's different on your system (use `which certbot` to find the path).

3. **Save and exit:**

   - In the editor (usually nano or vi), save the file and exit.
   - The cron job is now scheduled.

##### Verify the Cron Job

1. **List all cron jobs:**

   ```bash
   sudo crontab -l
   ```

   - Ensure your renewal command is listed.

2. **Check Cron Logs (Optional):**

   - Logs can be found in `/var/log/syslog` or `/var/log/cron`.
   - Use the following command to check for cron activities:

     ```bash
     sudo grep CRON /var/log/syslog
     ```

#### Using Systemd Timers

**Systemd** can manage scheduled tasks using timers.

##### Create a Systemd Timer and Service

1. **Create a service file:**

   Create a new file `/etc/systemd/system/certbot-renew.service` with the following content:

   ```ini
   [Unit]
   Description=Certbot Renewal

   [Service]
   Type=oneshot
   ExecStart=/usr/bin/certbot renew --quiet
   ```

   - **Explanation:**
     - `Type=oneshot` indicates the service runs a single task and then exits.
     - `ExecStart` runs the Certbot renewal command.

2. **Create a timer file:**

   Create a new file `/etc/systemd/system/certbot-renew.timer` with the following content:

   ```ini
   [Unit]
   Description=Runs Certbot Renewal Twice Daily

   [Timer]
   OnCalendar=*-*-* 00,12:00:00
   Persistent=true

   [Install]
   WantedBy=timers.target
   ```

   - **Explanation:**
     - `OnCalendar=*-*-* 00,12:00:00` schedules the timer to run at midnight and noon every day.
     - `Persistent=true` ensures the timer catches up if the system was off during the scheduled time.

3. **Enable and start the timer:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable certbot-renew.timer
   sudo systemctl start certbot-renew.timer
   ```

##### Verify the Systemd Timer

1. **Check the timer status:**

   ```bash
   sudo systemctl list-timers certbot-renew.timer
   ```

   - **Explanation:**
     - This command lists timers and shows when they last ran and when they're next scheduled.

2. **View the timer's logs:**

   ```bash
   sudo journalctl -u certbot-renew.service
   ```

   - **Explanation:**
     - This shows logs related to the Certbot renewal service.

### Renewal Confirmation and Troubleshooting

After setting up automatic renewal, it's important to confirm that it's working correctly.

#### Manually Run the Renewal Command

To test the setup, manually run the renewal command:

```bash
sudo systemctl start certbot-renew.service
```

- Check the status:

  ```bash
  sudo systemctl status certbot-renew.service
  ```

- Review the logs for any errors.

#### Check Renewal Logs

Certbot logs renewal attempts in `/var/log/letsencrypt/letsencrypt.log`. Review this file to troubleshoot any issues.

```bash
sudo less /var/log/letsencrypt/letsencrypt.log
```

- **Tip:** Press `Shift + G` to go to the end of the file, where the most recent logs are.

### Distribution-Specific Notes

While the above methods generally apply to most Linux distributions, here are some specific notes:

#### Ubuntu/Debian

- **Systemd is the default init system.**
- Using **systemd timers** is recommended.
- Certbot packages from the repositories may set up systemd timers automatically.

#### CentOS/RHEL/Amazon Linux

- **CentOS 7 and newer use systemd.**
- Older versions may require using **cron jobs**.
- Ensure you have the EPEL repository enabled for Certbot updates.

#### Fedora

- **Systemd is used by default.**
- Certbot packages may already configure systemd timers.
- Verify with:

  ```bash
  sudo systemctl list-timers | grep certbot
  ```

### Email Notifications (Optional)

To receive email notifications in case of renewal failures:

1. **Install mail utilities:**

   ```bash
   # Ubuntu/Debian
   sudo apt install mailutils

   # CentOS/RHEL/Amazon Linux
   sudo yum install mailx

   # Fedora
   sudo dnf install mailx
   ```

2. **Modify the cron job to send emails:**

   ```cron
   0 */12 * * * /usr/bin/certbot renew --quiet || echo "Certbot renewal failed" | mail -s "Certbot Renewal Failed" your-email@example.com
   ```

   - **Explanation:**
     - If the renewal fails, an email is sent to notify you.
     - Replace `your-email@example.com` with your actual email address.

---

## DNS Configuration

Ensure your domain's DNS records point to your server's IP address. This can be done through your domain registrar or DNS provider.

- **Add an A record** for your domain and `www` subdomain (if used):

  - **Host:** `@` (for the root domain) and `www` (for the subdomain)
  - **Type:** `A`
  - **Value:** Your server's public IP address

- **Propagation Time:**

  - DNS changes can take up to 24-48 hours to propagate globally.
  - Use tools like [WhatsMyDNS](https://www.whatsmydns.net/) to check propagation.

---

## Conclusion

Congratulations! You have successfully installed a Let's Encrypt SSL certificate on your Linux server using Apache or Nginx. You've also set up automatic renewal to ensure your website remains secure without manual intervention. Your website now has enhanced security and trustworthiness.

---

## Additional Resources

- **Certbot Documentation:** [https://certbot.eff.org/docs/](https://certbot.eff.org/docs/)
- **Let's Encrypt Community Support:** [https://community.letsencrypt.org/](https://community.letsencrypt.org/)
- **SSL Labs Server Test:** Use this tool to check your SSL configuration: [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)
- **Cron Job Syntax Generator:** [https://crontab-generator.org/](https://crontab-generator.org/)
- **Understanding Systemd Timers:** [https://www.freedesktop.org/software/systemd/man/systemd.timer.html](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
- **Certbot Renewal Guide:** [https://certbot.eff.org/docs/using.html#renewal](https://certbot.eff.org/docs/using.html#renewal)

---

**Note:** Always ensure your server software and Certbot are up to date to maintain security and compatibility. Keeping your server's time synchronized using NTP is also important to prevent scheduling issues.

### Install NTP (Network Time Protocol)

1. **Install NTP:**

   ```bash
   # Ubuntu/Debian
   sudo apt install ntp

   # CentOS/RHEL/Amazon Linux
   sudo yum install ntp

   # Fedora
   sudo dnf install ntp
   ```

2. **Start and enable the NTP service:**

   ```bash
   sudo systemctl enable ntp
   sudo systemctl start ntp
   ```

---

By following these steps, you've ensured that your SSL certificates will renew automatically, keeping your website secure without any manual effort. Regularly check the renewal process and logs to catch any issues early.
