# Certstream Server with NGINX Proxy

After CaliDog's Certstream server went offline for a while a few weeks ago I created this docker-compose file for self-hosting a certstream instance. It comes with an Nginx proxy to handle certificate management etc separately.

It uses a Docker image I've created using the [original CaliDog certstream server repo](https://github.com/CaliDog/certstream-server). 

# Installation

Assuming that you've set up a server and DNS records for your (sub)domain already. You'll need Docker and certbot to be installed.

Clone this repo onto your server:

`git clone https://github.com/nixintel/certstream-proxy`

Modify `docker-compose.yml` and all the `.conf` files in `nginx/conf.d` and replace the string `sub.domain.com` with domain/subdomain name that you want to use.

Ensure ports 80/443 are open on your host server.

Use certbot to create an SSL certificate with Let's Encrypt. Run `certbot certonly` and follow the prompts.

Ensure your cert files are in the correct Docker volume shared with the nginx docker container e.g.:

``` 
- /etc/letsencrypt/archive/sub.domain.com:/etc/letsencrypt/archive/sub.domain.com
- /etc/letsencrypt/live/sub.domain.com:/etc/letsencrypt/live/sub.domain.com

```
Run the `docker-compose` file :

`docker-compose up -d`

Go to your website at `https://sub.domain.com` and you should see the default Certstream splash page.

# Usage

To ingest data from Certstream, install the Python client:

`pip install certstream`

Then run:

`certstream --url wss://sub.domain.com/`

to start ingesting certs from the server you just created.

# Notes and Issues

Certstream ingests a *lot* of data. CaliDog estimate approx 250TB worth of data a month passing through the pipeline. In my experience of self-hosting so far, this is pretty close to reality.

I had some initial issue setting this up with a domain that was behind Cloudflare and kept throwing 520 errors. The fix was to add  `large_client_header_buffers 4 16k;` to `nginx/conf.d/default.conf`
