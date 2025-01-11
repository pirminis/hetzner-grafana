# Grafana + Graphite

This is a way to setup a Grafana + Graphite in your local machine or VPS like the one from Hetzner.

# Installation

## On your local machine

- Copy `docker-compose.yml` to your desired folder
- Open `docker-compose.yml` and modify `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` to your desired values. These are your grafana login credentials.
- Run `docker compose -p my-grafana-project up -d`
- IMPORTANT: run `docker compose -p my-grafana-project down` to turn it off immediately
- Open `./graphite_data/graphite_conf/storage-aggregation.conf` with your favorite text editor and change all `xFilesFactor` values to `0`
- Save file and quit the text editor
- Run `docker compose -p my-grafana-project up -d`

Open the browser and visit `http://localhost:3001`. Login with the username and password from docker-compose.yml file.

When grafana website is loaded, go to Menu -> Connections -> Add new connection -> Search for "graphite" -> Click on it -> Click on "Add new data source" -> HTTP -> URL -> Enter `host.docker.internal:2002` -> Scroll to the bottom -> Click "Save & test"

Don't forget to put new grafana and graphite folders to `.gitignore`:
```
grafana_data
graphite_data
```

And that's it!

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
