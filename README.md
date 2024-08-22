# Tor-Lighttpd-Onion
This project provides a setup using Docker to create a service running `Tor` and a `lighttpd` web server. The `.onion` address remains persistent between container restarts. 

## Project Structure

The project directory should look like this:

```
tor-lighttpd-onion/
│
├── docker-compose.yml
├── tor/
│   ├── torrc
│   └── hidden_service/
│       └── hostname (automatically generated)
└── html/
    └── index.html
    └── page2.html
    └── page3.html
    └── eccecc.html
```


## 1. Create the `docker-compose.yml` File

Here is an example of a `docker-compose.yml` file:

```yaml
services:
  tor:
    image: alpine:latest
    container_name: tor
    command: >
      sh -c "
      apk add --no-cache tor && 
      mkdir -p /var/lib/tor/hidden_service && 
      chmod 700 /var/lib/tor/hidden_service && 
      if [ ! -f /var/lib/tor/hidden_service/hostname ]; then
        echo 'HiddenServiceDir /var/lib/tor/hidden_service' >> /etc/tor/torrc &&
        echo 'HiddenServicePort 80 lighttpd:80' >> /etc/tor/torrc &&
        tor --quiet;
      else
        tor --quiet;
      fi"
    volumes:
      - ./tor/torrc:/etc/tor/torrc
      - ./tor/hidden_service:/var/lib/tor/hidden_service
    networks:
      - tor-network

  lighttpd:
    image: lighttpd:alpine
    container_name: lighttpd
    volumes:
      - ./html:/var/www/html    ports:
      # - "80:80" # Expose the web server on port 80 # Additionally, the site is also published on the regular web.
    networks:
      - tor-network

networks:
  tor-network:
    driver: bridge
```

## 2. Configure the torrc File

In this setup, the torrc file will be very very very simple. The main configuration is managed via the command section in the Docker Compose file. The torrc file will have only basic settings, with other settings dynamically added by the container:


```bash
# Torrc base config (other settings added by container)
Log notice stdout
```

## 3. Create the hidden_service Directory

The `hidden_service` directory will contain the `hostname` file, which will be automatically generated by Tor during the first execution. 
This file will be reused on subsequent container starts, ensuring the .onion address remains the same. It's important to map this directory as a volume in Docker Compose to persist the .onion address across container restarts.

## 4. Configure lighttpd

The html directory will contain the website you want to serve. Simply add an index.html file with the desired content for example.

## 5. Start the Service

Now you can start the service with:

```bash
docker compose up -d
```

**On the first run, the Tor container will generate a new .onion address and save it in the hostname file. On subsequent runs, the same address will be reused.**

## 6. Access the .onion Address

After starting, you can view the .onion address in the `tor/hidden_service/hostname` file. This is the address you will use to access your site via the Tor browser.


## If you don't have Docker and Docker Compose installed, follow these links:

[Install Docker](https://docs.docker.com/engine/install/)

[Install Docker Compose](https://docs.docker.com/compose/install/)

## Troubleshooting

Ensure your Docker and Docker Compose versions are up to date. If the .onion address is not persistent, check the hidden_service directory permissions and ensure it's correctly mapped in the Docker Compose file.
