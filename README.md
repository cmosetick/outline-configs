# Outline Wiki / Documentation configs and notes

Outline is a open-source clone of Notion.so and even gives an option to import data from Notion.

Outline's current outline documentation lists out the requirements that need to be met pretty well

https://docs.getoutline.com/s/hosting/doc/docker-7pfeLP5a8t

The actual authentication for users is a large topic by itself.  
The Google Auth instructions noted on the official outline documenatation are working well at time of writing.  
- It's straightforward to restrict login to users that are part of one or more Google Workspaces.  
https://docs.getoutline.com/s/hosting/doc/google-hOuvtCmTqQ

## 00 Create working directory for your docker related files
mkdir outline-deployment
cd outline-deployment

## 01 Create a `docker.env` file  

using the sample from the outline repo  

- https://github.com/outline/outline/blob/main/.env.sample

make sure to set the minimum required parameters

## 02 Prep the database
using the `docker-compose.yml` file that I have left here inside this repository.

```
# Database prep taken exactly from the official documentation

# Create the database with the following command:
docker compose run --rm outline yarn db:create --env=production-ssl-disabled

# Migrate the new database to add needed tables, indexes, etc:
docker compose run --rm outline yarn db:migrate --env=production-ssl-disabled
```

## 03 Bring up actual containers now
and check status

```
docker compose up -d
docker compose ps
```

## 04 Cloudflare Tunnel
(optional for public facing deployment)

- Functional configuration included inside the `docker-compose.yml` file here.
- I strongly recommend to use what I have included inside the compose file.

- Full documentation on the tunnel configuration is also a large topic.
Most tutorials on cloudflare tunnel I have seen thus far focus on very simple demos, even if using containers.

- You need to make sure that the cloudflare tunnel container can authenticate properly by configuring 'tunnel token' inside a .env file, or inside the docker-compose.yml file

### Important Notes on the tunnel deployment with regard to Outline

- Running the `cloudflared` tunnel contanier _inside the same network_ as Outline deployment is __critically important__, otherwise the tunnel is 'online' but not functional for serving web pages.

- If not created _inside_ the same network as Outline, then the plumbing is broken.  
e.g. do not use 'network=host' or other methods for cloudflared.
e.g. do not just copy / paste the commands that Cloudflare WebUI gives you.

- The Cloudflare WebUI will give you an option to plumb the tunnel to an IP address or hostname port combo. My highest recomendation is to paste in the the exact name that `docker compose ps` reports to you. e.g. `outline1` etc.  
- I noticed that docker compose will assign different IP addresses to the same service containers on succesive restarts, so it's very important to use a DNS name with the tunnel plumbing.

- Also important (at time of writing) there is no need to configure A records or AAAA records to make the tunnel functional. Cloudflare will create these for you, so ignore any tutorials that are instructing you to create A records. (I see this one of the main benefits of using the tunnel for public deployment.)
