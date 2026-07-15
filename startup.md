IMPORTANT

Before booting Traefik for the first time, run these two commands in your /home/tim/traefik/ directory to create the network and secure the ACME key permissions:

bash


docker network create traefik_proxy
touch acme.json && chmod 600 acme.json
