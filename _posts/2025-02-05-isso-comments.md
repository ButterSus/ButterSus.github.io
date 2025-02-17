---
layout: post
title: "Adding Comments to Jekyll with Isso: A Self-Hosted Alternative to Disqus"
tags: jekyll
---

Ever wanted to add comments to your Jekyll blog without relying on external services? I recently switched from Disqus to Isso, and I want to share my experience with you. Isso is a fantastic lightweight commenting system that gives your readers a clean, no-login-required way to engage with your content.

# Why Isso?

While Disqus is popular, it comes with some drawbacks - it can slow down your site and requires users to create accounts. Even GitHub Discussions, while great, forces people to log in with GitHub. Isso takes a different approach: it's simple, fast, and lets anyone jump right into the conversation.

# Quick Start: Adding Isso to Your Jekyll Blog

If you're using Jekyll like me (I'm using the Jekyll-Dash theme), switching from Disqus to Isso is pretty straightforward. Here's what I changed in my layout:

Before (with Disqus):
```html
{% raw %}{% if site.dash.disqus.shortname %}
<div class="comments">
  {% include disqus.html %}
</div>
{% endif %}{% endraw %}
```

After (with Isso):
```html
{% raw %}{% if site.dash.isso.url %}
<div class="comments">
<script data-isso="{{ site.dash.isso.url }}" src="{{ site.dash.isso.url }}/js/embed.min.js"></script>
<section id="isso-thread">
<noscript>Please enable JavaScript to view the comments.</noscript>
</section>
</div>
{% endif %}{% endraw %}
```

# Setting Up Your Own Isso Server

Here's the thing: Isso needs a server to run on. Don't worry though - you can use cheap VPS. One important note: if your blog uses HTTPS (like most GitHub Pages sites do), **you'll need a domain with SSL for your Isso server too**. Let's walk through the setup.

## Step 1: Getting Your Domain Ready

First, you'll need a domain. I went with a free option from [DuckDNS](https://www.duckdns.org/login?generateRequest=github). Once you have your domain (let's say `yourdomain.duckdns.org`), getting an SSL certificate is easy:

```bash
sudo certbot --nginx -d yourdomain.duckdns.org
```

## Step 2: Installing What You Need

Let's get your server ready with the necessary software:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv sqlite3
sudo pip3 install isso
```

## Step 3: Setting Up Isso

Create a configuration file (`isso-config.cfg`) in your home directory. Here's a simple setup to get you started:

```bash
[general]
dbpath = ~/isso-comments.db
host = https://yourdomain.github.io

[server]
listen = http://127.0.0.1:8080

[admin]
enabled = true
password = your-secure-password

[moderation]
enabled = false

[guard]
ratelimit = 2
```

Don't forget to open up HTTPS traffic on your firewall:

```bash
sudo ufw allow 443/tcp
sudo ufw reload
```

## Step 4: Configuring Nginx

We'll need to set up Nginx to handle requests to your Isso server. First, let's clean up:

```bash
sudo rm /etc/nginx/sites-available/default
```

Now, create a new Nginx configuration at `/etc/nginx/sites-available/isso`. Here's my setup (remember to replace the domains and paths with yours):

```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name yourdomain.duckdns.org;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl;
    server_name yourdomain.duckdns.org;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/yourdomain.duckdns.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.duckdns.org/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Serve Isso's JavaScript files
    location /js/ {
        alias /path/to/your/isso/js/files;
        autoindex on;
        types {
            application/javascript js;
            application/javascript min.js;
            application/json map;
        }
        add_header Access-Control-Allow-Origin "*";
        add_header Access-Control-Allow-Methods "GET, OPTIONS";
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Apply your changes:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## Step 5: Running Isso

Let's first test that everything works:

```bash
isso -c ~/isso-config.cfg
```

Visit your blog and check if the comment section appears. If it does, great! Now let's make it run automatically.

## Step 6: Making Isso Run Automatically

Create a service file at `/etc/systemd/system/isso.service`:

```ini
[Unit]
Description=Isso Comment Server
After=network.target

[Service]
Type=simple
User=your-username
ExecStart=/usr/local/bin/isso -c /home/your-username/isso-config.cfg
Restart=always
RestartSec=3

# Some extra security measures
ProtectSystem=full
PrivateTmp=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

Set it up to run automatically:

```bash
sudo systemctl daemon-reload
sudo systemctl enable isso
sudo systemctl start isso
```

Also don't forget to add your server's ip to your website config (`_config.yml`):

```yml
dash:
  # My cheap VPS running backend for isso comments
  isso:
    url: https://buttersus.duckdns.org
```

# That's It!

Now you have a fully functioning comment system on your Jekyll blog! Your readers can leave comments without creating accounts, and you have full control over the system. The comments load quickly since they're served from your own server, and you can easily moderate them through Isso's admin panel.

Have questions about setting this up? Feel free to leave a comment below (yes, using Isso!) or reach out to me directly.
