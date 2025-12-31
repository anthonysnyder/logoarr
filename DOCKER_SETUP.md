# Docker Compose Setup with Automatic SMB Mounts

This setup eliminates the "blank page" issue caused by stale SMB mounts by letting Docker manage the CIFS connections directly.

## Why This Fixes the Problem

**Before:** The container relied on the host's SMB mounts, which could get into a stale state (Errno 11: Resource temporarily unavailable). Restarting the container didn't help because the mount issue was on the host.

**After:** Docker manages the CIFS volumes directly. When the container restarts, Docker automatically remounts the SMB shares, eliminating stale mount issues.

## Prerequisites

1. **Docker with CIFS support** - Your Docker host needs `cifs-utils` installed:
   ```bash
   # On Ubuntu/Debian
   sudo apt-get install cifs-utils

   # On RHEL/CentOS
   sudo yum install cifs-utils
   ```

2. **NAS credentials** - Username and password for your NAS
3. **Fixed NAS IP** - Your NAS should have a static IP address

## Setup Instructions

### Step 1: Create Environment File

Copy the example and fill in your values:

```bash
cp .env.example .env
```

Edit `.env` with your actual values:

```env
# TMDB API Configuration
TMDB_API_KEY=your_actual_tmdb_key

# NAS/SMB Configuration
NAS_IP=192.168.1.XXX
SMB_USERNAME=your_nas_username
SMB_PASSWORD=your_nas_password

# Optional: Slack Notifications
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

**IMPORTANT:** Add `.env` to your `.gitignore` to keep credentials secure!

### Step 2: Update NAS Share Paths (if needed)

If your NAS share names are different from the defaults (`movies`, `kids-movies`, `tv`, `kids-tv`), update the `device` paths in `docker-compose.yml`:

```yaml
volumes:
  movies:
    driver: local
    driver_opts:
      type: cifs
      o: "username=${SMB_USERNAME},password=${SMB_PASSWORD},vers=3.0,file_mode=0777,dir_mode=0777,iocharset=utf8,cache=loose,actimeo=60,_netdev"
      device: "//${NAS_IP}/your-actual-share-name"  # ← Change this
```

### Step 3: Deploy with Docker Compose

```bash
# Build and start the container
docker-compose up -d

# Watch the logs to verify it's working
docker-compose logs -f
```

### Step 4: Verify Mounts are Working

```bash
# Check volume status
docker-compose exec logoarr ls -la /movies /kids-movies /tv /kids-tv

# Test Python access
docker-compose exec logoarr python -c "import os; print('Movies:', len(os.listdir('/movies')))"

# Check health status
docker-compose ps
```

You should see:
- Healthy status in `docker-compose ps`
- No "Resource temporarily unavailable" errors
- Actual file counts from your NAS

## Troubleshooting

### Issue: "mount error(13): Permission denied"

**Solution:** Check your SMB username and password in `.env`

### Issue: "mount error(115): Operation now in progress"

**Solution:** Verify your NAS IP is correct and reachable:
```bash
ping ${NAS_IP}
```

### Issue: Volumes not mounting

**Solution:** Check Docker host has CIFS support:
```bash
docker run --rm -it --privileged alpine sh -c "apk add cifs-utils && mount.cifs -V"
```

### Issue: Still getting blank pages

**Solution:**
1. Check container health: `docker-compose ps`
2. Watch logs during page load: `docker-compose logs -f`
3. Test volumes: `docker-compose exec logoarr python -c "import os; print(os.listdir('/movies'))"`

## Advanced Configuration

### Adjusting SMB Version

If your NAS requires SMB 2.1 or 2.0, change `vers=3.0` to:
- `vers=2.1` for SMB 2.1
- `vers=2.0` for SMB 2.0
- `vers=1.0` for SMB 1.0 (not recommended - security risk)

### Performance Tuning

The mount options are optimized for reliability and performance:

- `file_mode=0777,dir_mode=0777` - Full read/write access
- `iocharset=utf8` - Handle special characters in filenames
- `cache=loose` - Better performance with slightly relaxed consistency
- `actimeo=60` - Cache file attributes for 60 seconds (reduces network traffic)
- `_netdev` - Wait for network before mounting

To make it more aggressive (better performance, less reliability):
```yaml
o: "username=${SMB_USERNAME},password=${SMB_PASSWORD},vers=3.0,file_mode=0777,dir_mode=0777,iocharset=utf8,cache=loose,actimeo=300,_netdev"
```

To make it more conservative (better reliability, slower):
```yaml
o: "username=${SMB_USERNAME},password=${SMB_PASSWORD},vers=3.0,file_mode=0777,dir_mode=0777,iocharset=utf8,cache=strict,actimeo=30,_netdev"
```

## Migrating from Existing Portainer Setup

### Option 1: Use Docker Compose in Portainer

1. In Portainer, go to **Stacks** → **Add Stack**
2. Name it "logoarr"
3. Copy the contents of `docker-compose.yml`
4. Add environment variables in the **Environment variables** section
5. Deploy

### Option 2: Remove Old Container, Deploy New

1. Stop and remove the old container in Portainer
2. Deploy using the command line:
   ```bash
   docker-compose up -d
   ```
3. The new container will appear in Portainer automatically

## Health Monitoring

The container includes a health check that tests volume access every 30 seconds. Monitor it with:

```bash
# Check health status
docker inspect logoarr | grep -A 10 Health

# Watch health checks in real-time
watch -n 5 'docker inspect logoarr | grep -A 10 Health'
```

## Benefits of This Approach

1. **Auto-recovery** - Docker remounts volumes when container restarts
2. **Consistent state** - No more stale mounts from host system
3. **Portable** - Easy to move to different hosts
4. **Version controlled** - Infrastructure as code
5. **Health monitoring** - Built-in checks for mount availability

## Reference

Based on: https://blog.mbirth.uk/2025/02/01/docker-compose-and-automatic-mounts.html
