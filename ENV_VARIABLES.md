# Environment Variables

## Configuration

Logoarr can be configured using the following environment variables:

### Required

| Variable | Description | Example |
|----------|-------------|---------|
| `TMDB_API_KEY` | Your TMDb API key for fetching movie/TV metadata | `your_api_key_here` |

### Optional

| Variable | Description | Default | Example |
|----------|-------------|---------|---------|
| `MOVIE_FOLDERS` | Comma-separated list of movie folder paths | `/movies,/kids-movies,/anime` | `/movies,/movies2,/anime` |
| `TV_FOLDERS` | Comma-separated list of TV show folder paths | `/tv,/kids-tv` | `/tv,/tv2,/kids-tv` |
| `SLACK_WEBHOOK_URL` | Slack webhook URL for notifications | None | `https://hooks.slack.com/...` |

## Docker Compose Example

### Basic Setup

```yaml
version: '3.8'

services:
  logoarr:
    image: swguru2004/logoarr:latest
    container_name: logoarr
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      - TMDB_API_KEY=${TMDB_API_KEY}
      # Optional: Override default folders
      - MOVIE_FOLDERS=/movies,/kids-movies,/anime
      - TV_FOLDERS=/tv,/kids-tv
      # Optional: Slack notifications
      - SLACK_WEBHOOK_URL=${SLACK_WEBHOOK_URL}
    volumes:
      - /path/to/movies:/movies
      - /path/to/kids-movies:/kids-movies
      - /path/to/anime:/anime
      - /path/to/tv:/tv
      - /path/to/kids-tv:/kids-tv
      # Optional: Persist TMDb mappings
      - ./tmdb_directory_mapping.json:/app/tmdb_directory_mapping.json
```

### Advanced Setup with Multiple Folders

```yaml
version: '3.8'

services:
  logoarr:
    image: swguru2004/logoarr:latest
    container_name: logoarr
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - TMDB_API_KEY=${TMDB_API_KEY}
      # Custom folder configuration
      - MOVIE_FOLDERS=/movies,/movies2,/kids-movies,/anime
      - TV_FOLDERS=/tv,/tv2,/kids-tv,/kids-tv2
      - SLACK_WEBHOOK_URL=${SLACK_WEBHOOK_URL}
    volumes:
      - ./config:/config
      - /mnt/movies:/movies
      - /mnt/movies2:/movies2
      - /mnt/kids-movies:/kids-movies
      - /mnt/anime:/anime
      - /mnt/tv:/tv
      - /mnt/tv2:/tv2
      - /mnt/kids-tv:/kids-tv
      - /mnt/kids-tv2:/kids-tv2
      - ./tmdb_directory_mapping.json:/app/tmdb_directory_mapping.json
    networks:
      - media

networks:
  media:
    external: true
```

## How It Works

1. **Folder Discovery**: On startup, Logoarr reads the `MOVIE_FOLDERS` and `TV_FOLDERS` environment variables
2. **Path Validation**: Only folders that actually exist are used (invalid paths are skipped)
3. **Logging**: The folders being used are logged on startup for verification
4. **Backward Compatibility**: If no environment variables are set, defaults are used

## Folder Path Rules

- Paths should be **comma-separated** without spaces: `/movies,/tv,/anime`
- Paths are **automatically trimmed** of whitespace
- Non-existent paths are **automatically filtered out**
- At least one valid folder must exist for each type (movies/TV)

## Example Startup Logs

```
INFO in app: Movie folders: ['/movies', '/kids-movies', '/anime']
INFO in app: TV folders: ['/tv', '/kids-tv']
```

If a folder doesn't exist:
```
INFO in app: Movie folders: ['/movies', '/kids-movies']
INFO in app: TV folders: ['/tv']
```
(Note: `/anime` was skipped because it doesn't exist)

## Docker Run Example

```bash
docker run -d \
  --name logoarr \
  -p 5000:5000 \
  -e TMDB_API_KEY=your_api_key_here \
  -e MOVIE_FOLDERS=/movies,/kids-movies,/anime \
  -e TV_FOLDERS=/tv,/kids-tv \
  -e SLACK_WEBHOOK_URL=https://hooks.slack.com/your/webhook \
  -v /path/to/movies:/movies \
  -v /path/to/kids-movies:/kids-movies \
  -v /path/to/anime:/anime \
  -v /path/to/tv:/tv \
  -v /path/to/kids-tv:/kids-tv \
  swguru2004/logoarr:latest
```

## Migration from Hardcoded Paths

If you're upgrading from a version with hardcoded paths:

1. **No action required** - The defaults match the old hardcoded values
2. **Optional**: Add `MOVIE_FOLDERS` and `TV_FOLDERS` to your docker-compose.yml for clarity
3. **Recommended**: Remove unused volume mounts to clean up your configuration

## Troubleshooting

### "No folders found" error

Make sure:
1. Environment variables are set correctly (comma-separated, no extra spaces)
2. Volume mounts exist and match the paths in environment variables
3. Paths inside the container match the environment variable paths

Example problem:
```yaml
environment:
  - MOVIE_FOLDERS=/movies,/anime  # Expects /movies and /anime
volumes:
  - /host/movies:/films  # ❌ Container path doesn't match!
```

Example fix:
```yaml
environment:
  - MOVIE_FOLDERS=/movies,/anime  # Expects /movies and /anime
volumes:
  - /host/movies:/movies  # ✅ Container path matches
  - /host/anime:/anime    # ✅ Container path matches
```

### Check Current Configuration

View logs to see which folders are being used:
```bash
docker logs logoarr 2>&1 | grep "folders:"
```
