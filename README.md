# raftman

![raftman](frontend/static/ui/logo-96.png)

A syslog server with integrated full text search via a JSON API and Web UI.

## getting started

### store logs

To get started quickly, just run the containerized version of raftman:

```
sudo docker run --rm --name raftman \
    -v /tmp/logs.db:/var/lib/raftman/logs.db \
    -p 514:514/udp \
    -p 5514:5514 \
    -p 8181:8181 \
    -p 8282:8282 \
    pierredavidbelanger/raftman
```

This will start raftman with all default options. It listen on port 514 (UDP) and 5514 (TCP) on the host for incomming RFC5424 syslog packets and store them into an SQLite database stored in `/tmp/logs.db` on the host. It also exposes the JSON API on http://localhost:8181/api/ and the Web UI on http://localhost:8282/.

### send logs

Time to fill our database. The easyest way is to just start [logspout](https://github.com/gliderlabs/logspout) and tell it to point to raftman's syslog port:

```
docker run --rm --name logspout \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    gliderlabs/logspout \
        syslog://10.0.1.2:514
```

**Reaplce `10.0.1.2` with your host ip.**

This last container will grab other containers output lines and send them as syslog packet to the configured ip (ie: our raftman container).

### generate logs

Now, we also need to generate some output. This will do the job for now:

```
docker run --rm --name test \
    alpine \
    echo 'Can you see me'
```

### visualise logs

Then we can visualize our logs:

with the raftman API:

```
curl http://localhost:8181/api/list \
    -d '{"Limit": 100, "Message": "see"}'
```

or pop the Web UI at http://localhost:8282/