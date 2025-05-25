# Fixing Sigil: Allow Docker Containers to Access Host Services via UFW

### 1. Find your Docker bridge subnet

```bash
docker network inspect monitoring --format '{{(index .IPAM.Config 0).Subnet}}'
```

Example output: 172.19.0.0/16

### 2. Configure UFW

a. Allow traffic from that subnet to the host port
```bash
sudo ufw allow from 172.19.0.0/16 to any port 9091
```

b. (Optional) Allow general Docker bridge traffic
```bash
sudo ufw allow in on docker0
```

c. Reload UFW
```bash
sudo ufw reload
```

### 3. Verify from inside the container

```bash
docker exec -it prometheus wget -qO- http://host.docker.internal:9091/metrics
```