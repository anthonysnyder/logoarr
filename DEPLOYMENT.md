# Deployment Guide - logoarr with Stable SMB Mounts

This deployment guide addresses the "random blank page" issue by implementing two key fixes:

1. **Stable mount path** - Uses your Automounter SMB mount at `/Volumes/UNAS_Data`
2. **Fail loudly, not silently** - Shows error pages instead of blank pages when SMB is unavailable

## Prerequisites

- **macOS host** with Automounter configured to mount `UNAS_Data` at `/Volumes/UNAS_Data`
- **Docker Desktop** installed
- **UNAS share structure:**
  ```
  /Volumes/UNAS_Data/
  └── Media/
      ├── Movies/
      ├── Kids Movies/
      ├── TV Shows/
      ├── Kids TV/
      └── Anime/
  ```

## Setup Instructions

### 1. Create Environment File

```bash
cp .env.example .env
```

Edit `.env` with your values:
```env
TMDB_API_KEY=your_actual_api_key
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL  # Optional
```

### 2. Verify Automounter Mount

Ensure your UNAS is mounted:
```bash
ls -la /Volumes/UNAS_Data/Media/
```

You should see: Movies, Kids Movies, TV Shows, Kids TV, Anime

### 3. Deploy with Docker Compose

```bash
docker-compose up -d
```

### 4. Verify Deployment

```bash
# Check container is running
docker ps | grep logoarr

# Check logs
docker logs logoarr -f

# Test volume access
docker exec logoarr ls -la /movies
```

### 5. Access the Application

Open: http://localhost:5000

## What Was Changed

### 1. Improved Retry Logic (app.py)

**Before:**
- 8 retries with 0.05s base delay
- Returned empty list on failure (causing blank page)

**After:**
- **15 retries** with 0.1s base delay (max ~3.2 seconds total)
- **Raises IOError** on failure (shows error page)
- **Logs warnings** for each retry attempt

### 2. User-Visible Error Pages

**Before:**
- Silent failure = blank page
- No indication of what went wrong

**After:**
- Custom error page with helpful troubleshooting steps
- Shows the actual error message
- Provides "Retry" button

### 3. Simple Bind Mounts

**Before:**
- Relied on complex CIFS volume drivers
- Conflicted with macOS `/Volumes/Data` path

**After:**
- Simple bind mounts from Automounter path
- `/Volumes/UNAS_Data/Media/Movies` → `/movies` in container
- No conflicts, Docker just passes through to existing mount

## Testing the Fix

### Test 1: Normal Operation
```bash
# Access the app
open http://localhost:5000

# Should load normally with all your movies
```

### Test 2: Simulate SMB Failure
```bash
# Unmount the SMB share
diskutil unmount /Volumes/UNAS_Data

# Try to access the app
open http://localhost:5000

# Should see error page instead of blank page
```

### Test 3: Recovery
```bash
# Remount via Automounter or manually
open smb://192.168.1.29/UNAS_Data

# Refresh the app
open http://localhost:5000

# Should work again
```

## Troubleshooting

### Issue: "Error: Unable to access /movies after 15 retries"

**Cause:** SMB mount is unavailable or slow

**Solution:**
1. Check Automounter status
2. Verify mount: `ls /Volumes/UNAS_Data`
3. Remount if needed
4. Restart container: `docker restart logoarr`

### Issue: Container can't find /Volumes/UNAS_Data

**Cause:** Docker Desktop file sharing not configured

**Solution:**
1. Open **Docker Desktop** → **Settings** → **Resources** → **File Sharing**
2. Add `/Volumes/UNAS_Data`
3. Click **Apply & Restart**

### Issue: Still getting blank pages

**Cause:** Browser cached the old blank page

**Solution:**
1. Hard refresh: `Cmd+Shift+R`
2. Clear browser cache
3. Check logs: `docker logs logoarr --tail 50`

## Performance Notes

With 15 retries and exponential backoff:
- **Retry 1:** 0.1s delay
- **Retry 5:** 1.6s delay
- **Retry 10:** ~51s cumulative
- **Retry 15:** ~204s cumulative (max ~3.4 minutes)

The app will wait up to **~3.4 minutes** before giving up. This is intentional to handle temporary SMB slowness.

If this is too long, edit `app.py` and reduce `retries=15` to `retries=10` (max ~1.6 minutes).

## Logs to Monitor

**Success:**
```
INFO - Movie folders: ['/movies', '/kids-movies']
INFO - TV folders: ['/tv', '/kids-tv', '/anime']
```

**Retry Warning (normal during slow SMB):**
```
WARNING - BlockingIOError on attempt 3/15 for /movies
```

**Failure (shows error page):**
```
ERROR - Failed to list /movies after 15 retries: [Errno 11] Resource temporarily unavailable
```

## Benefits of This Approach

1. ✅ **No silent failures** - Users see helpful error messages
2. ✅ **More aggressive retries** - 15 attempts instead of 8
3. ✅ **Better logging** - Track when SMB is slow
4. ✅ **Stable mount path** - Uses Automounter, no conflicts
5. ✅ **Simple configuration** - Just bind mounts, no CIFS drivers

## Maintenance

### Update the Image

```bash
docker-compose pull
docker-compose up -d
```

### View Logs

```bash
# Follow logs in real-time
docker-compose logs -f

# Last 100 lines
docker logs logoarr --tail 100
```

### Restart Container

```bash
docker-compose restart
```

### Stop and Remove

```bash
docker-compose down
```

## For Production Deployment

If deploying to a production Mac mini server:

1. Ensure Automounter starts on boot
2. Consider increasing retries to 20+ for very slow networks
3. Set up monitoring for the error logs
4. Configure Docker to start on boot

## Next Steps

Once deployed and stable:
1. Monitor for a few days
2. Check logs for any retry warnings
3. If working well, deploy to other arr apps (postarr, backgroundarr)
4. Consider creating a multi-service stack
