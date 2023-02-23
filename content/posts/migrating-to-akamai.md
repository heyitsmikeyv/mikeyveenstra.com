---
title: "Migrating to Akamai"
date: 2023-02-19T17:21:51-05:00
draft: true
categories: 
    - Blog
---

I've used DigitalOcean for hosting my various project servers ever since I started having projects to put on servers in the first place. I've had zero problems with them and they've gotten roughtly fifteen dollars per month from me for a good long time. This week, I made the decision to close my DigitalOcean account.
<!--more-->

## Background

Last Wednesday, [DigitalOcean laid off about 11 percent of its employees](https://www.theregister.com/2023/02/15/digitalocean_layoffs/).  Then, a day later, their [2022 earnings call](https://investors.digitalocean.com/news/news-details/2023/DigitalOcean-Announces-Fourth-Quarter-and-Fiscal-Year-2022-Financial-Results/default.aspx) patted themselves on the back for "strong revenue growth, with significant increases in free cash flow margin" before approving half a billion dollars in stock buybacks. This thread by Twitter user Mason Egger sums up my feelings pretty nicely:


{{< tweet 1626231150091046912 >}}


So, I spent some time packing up and moving from DigitalOcean to Linode, which is in the middle of a rebrand as [Akamai Connected Cloud](https://www.linode.com/blog/linode/a-bold-new-approach-to-the-cloud/). I'm happy to report that my site is now living on its new host and my DigitalOcean account has been closed. 

In this post I'll be walking through my process of moving my Hugo site to a fresh server, from the initial server setup and configuration, to securing my traffic with LetsEncrypt and Cloudflare.

{{<admonition type=question title="What's with the over-explaining?">}}
I figured some of my students may be interested in a holistic view of a deployment like this, so I'll be doing a lot of clarifying and explaining things to help understand the reasoning and concepts behind each step of the process.
{{</admonition>}}

## Ingredients

For this project, we'll be using the following tools, technologies, and services:
- **Akamai Connected Cloud**, formerly known as **Linode** - My new hosting provider. I pay them for a small virtual server, effectively renting an incredibly tiny chunk of a physical server in a room somewhere around Atlanta, Georgia.
{{<admonition type=note title="Akamai? Linode? Which is it?" open=false >}}
I'll be referring to the server itself as a "linode" in this post, as that's how they currently refer to individual virtual servers in their environment even after the rebrand (that is, at the time of this writing).
{{</admonition>}}

- **Ubuntu 22.04 LTS** - The operating system (OS) installed on my linode. Ubuntu is a well-supported and relatively easy to use Linux distribution, pretty well suited for general-purpose usage. 
- **Nginx** - The webserver installed on my linode. Nginx listens for HTTP(S) requests coming in from the internet, figures out what they're asking for, and tries to serve it to them.
- **GitHub** - The version control repository I use to store my project code. The files that I need to redeploy my website live there. 
- **Hugo** - The static site generator I use to build my website. It's responsible for assembling the various code and content assets stored in GitHub into the HTML pages that actually get served to your browser from this site. 
- **Cloudflare** - The edge security provider I use for several important services on my site. I use Cloudflare for my domain's DNS, it acts as a web application firewall (WAF), and also serves as a Content Delivery Network, or CDN. In short, all the traffic coming into my site needs to go through Cloudflare first, including yours if you're reading this!
- **LetsEncrypt** - The Certificate Signing Authority (CSA) I use to generate and sign certificates so my server can seamlessly handle HTTPS traffic. Without this, we'd have to make some compromises when it came to the privacy of the data sent to and from this site. 


## Setting Up A Linode

lorem ipsum

## Webserver Configuration

lorem ipsum

## Deploying the Website

lorem ipsum

## Securing the Server

lorem ipsum

## Wrapping Up

lorem ipsum

## _ NOTES _ DELETE _ THIS _ 

1. SSH in to new linode server
2. Create non-root user
    Copy keys
    Disable password auth
3. Install apt updates 
* Install dependencies - nginx, certbot, python3-certbot-nginx, hugo
4. Nginx setup 
    - Base nginx config for site in /etc/nginx/sites-available/mikeyveenstra.com.conf
        ```
server {
    listen 80; # Temporary, removed by certbot later
    root /var/www/mikeyveenstra.com/public;
    index index.html index.htm index.nginx-debian.html;

    server_name mikeyveenstra.com www.mikeyveenstra.com;
    error_page 404 /404.html;
    location / {
        try_files $uri $uri/ =404;
    }

}
        ```
    - Enable site by creating symlink in sites-enabled/ 
    - Test nginx config `nginx -t` 
    - Reload nginx `sudo service nginx reload` 
5. Git clone site, include submodules with `--recurse-submodules`, hugo build
6. Cloudflare DNS switch 
    - Switch off strict SSL first 
7. Block non-Cloudflare IPs in Linode Firewall
    - while we're at it, block all other non-essential traffic
8. Certbot certs
    - sudo certbot --nginx (autoconfigures based on nginx config)
9. Fix nginx logs for cloudflare 
        - add to /etc/nginx/conf.d/cloudflare.conf: 
```
# - IPv4
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 104.16.0.0/12;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 131.0.72.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;


real_ip_header CF-Connecting-IP;
```