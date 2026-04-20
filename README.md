# matrix-synapse-docker-compse-setup-with-Proxy/NPM
Working Setup for Matrix-Synapse with Postgress for Docker Compose with NPM

### WIP

```bash
mkdir -p /opt/matrix
cd /opt/matrix
```
## clone yml
git clone https://github.com/ChrissWalters/matrix-synapse-docker-compse-setup-with-Nginx-Proxy-Manager.git

```bash
docker compose up -d
```

```bash
docker compose down
```
# create homeserver.yml in "synapse_data":
```bash:
docker run -it --rm \
  -v /opt/matrix/synapse_data:/data \
  -e SYNAPSE_SERVER_NAME=YourDomain.tld \
  -e SYNAPSE_REPORT_STATS=no \
  matrixdotorg/synapse:latest generatehttps://element-hq.github.io/synapse/latest/reverse_proxy.html
```

# edit /opt/matrix/synapse_data/homeserver.yaml

(TODO)

change "database" for postgress to:

```bash
[...]
database:
  name: psycopg2
  args:
    user: synapse
    password: "Your_Password_From_Compose_File"
    dbname: synapse
    host: db
    cp_min: 5
    cp_max: 10
    # seconds of inactivity after which TCP should send a keepalive message to the server
    keepalives_idle: 10
    # the number of seconds after which a TCP keepalive message that is not
    # acknowledged by the server should be retransmitted
    keepalives_interval: 10
    # the number of TCP keepalives that can be lost before the client's connection
    # to the server is considered dead
    keepalives_count: 3
[...]
```


```base
docker compopse up -d
```

# Setup Nginx Proxy Manager (see: https://element-hq.github.io/synapse/latest/reverse_proxy.html) or any other Proxy

```bash
Add New Proxy-Host
    - Tab Details
        - Domain Names: matrix.your-domain.tld
        - Scheme: http
        - Forward Hostname / IP: localhost # IP address or hostname where Synapse is hosted. Bare-metal or Container.
        - Forward Port: 8008

    - Tab Custom locations
        - Add Location
        - Define Location: /_matrix
        - Scheme: http
        - Forward Hostname / IP: localhost # IP address or hostname where Synapse is hosted. Bare-metal or Container.
        - Forward Port: 8008
        - Click on the gear icon to display a custom configuration field. Increase client_max_body_size to match max_upload_size defined in homeserver.yaml
            - Enter this in the Custom Field: client_max_body_size 50M;

    - Tab SSL/TLS
        - Choose your SSL/TLS certificate and preferred settings.

    - Tab Advanced (the gear icon on the far right of "SSL")
        - Enter this in the Custom Field. This means that port 8448 no longer needs to be opened in your Firewall.
          The Federation communication use now Port 443.

            location /.well-known/matrix/server {
              return 200 '{"m.server": "matrix.example.com:443"}';
      		  add_header Content-Type application/json;
            }

 			location /.well-known/matrix/client {
      		  return 200 '{"m.homeserver": {"base_url": "https://matrix.example.com"}}';
      		  add_header Content-Type application/json;
      		  add_header "Access-Control-Allow-Origin" *;
            }


```



### Add user:

## Edit homeserver.yaml to:
```bash
enable_registration: false
registration_shared_secret: "RANDOM_SECRET"
```

# matrix-synapse = container name
```bash
docker exec -it matrix-synapse register_new_matrix_user \
  -c /data/homeserver.yaml \
  http://localhost:8008
```
