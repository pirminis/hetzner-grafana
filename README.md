# Grafana + Graphite

This is a way to setup a Grafana + Graphite in your local machine or VPS like the one from Hetzner.

# Installation

## On your local machine

- Copy `docker-compose.yml` to your desired folder
- Open `docker-compose.yml` and modify `GF_SECURITY_ADMIN_USER`, `GF_SECURITY_ADMIN_PASSWORD` and `GRAPHITE_BASIC_AUTH_PASSWORD` to your desired values. These are your grafana login credentials.
- Create `./nginx/graphite.conf` with the content below:
    ```
    server {
      listen 80;
      server_name _;

      # Basic Auth for Graphite
      auth_basic "Graphite";
      auth_basic_user_file /etc/nginx/.htpasswd;

      location / {
        proxy_pass http://graphite:80;  # Graphite web inside the Docker network
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
      }
    }
    ```
- Save the file and quit text editor
- Run `htpasswd -cB nginx/.htpasswd graphite` and create password for graphite basic auth. It should be the same as GRAPHITE_BASIC_AUTH_PASSWORD value from docker-compose.yml
- Run `docker compose -p my-grafana-project up -d`
- IMPORTANT: run `docker compose -p my-grafana-project down` to turn it off immediately
- Open `./graphite_data/graphite_conf/storage-aggregation.conf` with your favorite text editor and change all `xFilesFactor` values to `0`
- Save the file and quit text editor
- Run `docker compose -p my-grafana-project up -d`

Open the browser and visit `http://localhost:3001`. Login with the username and password from docker-compose.yml file.

When grafana website is loaded, go to:
- Menu
- Connections
- Add new connection
- Search for "graphite" and click on it
- Click on "Add new data source"
- Find HTTP -> URL -> Enter `host.docker.internal:2002`
- Scroll to Auth -> Basic Auth and turn it ON -> Enter "graphite" for username and your password from docker-compose.yml (GRAPHITE_BASIC_AUTH_PASSWORD) and htpasswd for password
- Scroll to the bottom 
- Click "Save & test"

That's it!

## On Hetzner

For instructions check out this video I made: [https://drive.google.com/file/d/1YbTEHO40mP2Nmi2cA8Zkk8Vbnk9kLWlu/view?usp=sharing](https://drive.google.com/file/d/1YbTEHO40mP2Nmi2cA8Zkk8Vbnk9kLWlu/view?usp=sharing)

# Sending stats

## On your local machine

Ruby:
```
def measure(path, value, statsd_type)
  sock = UDPSocket.new
  message = "#{path}:#{value}|#{statsd_type}"

  sock.send(message, 0, "localhost", 8125)
  sock.close
end

measure("worker.my_worker.execution_time", 123, 'ms')
```

## On Hetzner

Same as on your local machine, just use your Hetzner server's IP address instead of localhost.
