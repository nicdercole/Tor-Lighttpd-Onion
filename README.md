# Tor-Lighttpd-Onion
This project provides a setup using Docker Compose to create a service running Tor and a `lighttpd` web server. The `.onion` address remains persistent between container restarts. Additionally, the site is also published on the regular web.

## Project Structure

The project directory should look like this:

tor-lighttpd-onion/
│
├── docker-compose.yml
├── tor/
│ ├── torrc
│ └── hidden_service/
│ └── hostname (automatically generated)
└── html/
└── index.html

yaml


## 1. Create the `docker-compose.yml` File

Here is an example of a `docker-compose.yml` file:

```yaml
version: '3.8'

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
      - ./html:/var/www/localhost/htdocs
    ports:
      - "80:80" # Expose the web server on port 80
    networks:
      - tor-network

networks:
  tor-network:
    driver: bridge
```

## 2. Configure the torrc File

In this setup, the torrc file will be very simple. The main configuration is managed via the command section in the Docker Compose file. The torrc file will have only basic settings, with other settings dynamically added by the container:


```yaml
# Torrc base config (other settings added by Docker Compose)
Log notice stdout
```

## 3. Create the hidden_service Directory

The hidden_service directory will contain the hostname file, which will be automatically generated by Tor during the first execution. This file will be reused on subsequent container starts, ensuring the .onion address remains the same. It's important to map this directory as a volume in Docker Compose to persist the .onion address across container restarts.

## 4. Configure lighttpd

The html directory will contain the website you want to serve. Simply add an index.html file with the desired content.

## 5. Start the Service

Now you can start the service with:

```bash

docker-compose up -d
```

On the first run, the Tor container will generate a new .onion address and save it in the hostname file. On subsequent runs, the same address will be reused.

## 6. Access the .onion Address

After starting, you can view the .onion address in the tor/hidden_service/hostname file. This is the address you will use to access your site via the Tor browser.
Additional Steps: Adding Hugo for Static Site Generation

You can also add Hugo to your setup for generating static pages. Here's how you can extend the docker-compose.yml file:

yaml

  hugo:
    image: klakegg/hugo:alpine
    container_name: hugo
    command: >
      sh -c "
      hugo --baseURL http://localhost --destination /var/www/localhost/htdocs && 
      hugo server --bind 0.0.0.0 --baseURL http://localhost --appendPort=false"
    volumes:
      - ./hugo:/src
      - ./html:/var/www/localhost/htdocs
    ports:
      - "1313:1313"
    networks:
      - tor-network

This configuration allows you to work on your Hugo site and serve it both as a .onion site and as a regular website.
Docker and Docker Compose Installation

If you don't have Docker and Docker Compose installed, follow these links:

    Install Docker
    Install Docker Compose

Troubleshooting

    Ensure your Docker and Docker Compose versions are up to date.
    If the .onion address is not persistent, check the hidden_service directory permissions and ensure it's correctly mapped in the Docker Compose file.

Contributions

Feel free to open issues or submit pull requests if you find bugs or want to add features.
License

This project is licensed under the MIT License.
