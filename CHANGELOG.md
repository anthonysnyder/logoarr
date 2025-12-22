# Changelog - Logoarr Bug Fixes and Improvements

## 2025-12-22 - Logo Overwriting Bug Fix + TMDb ID Mapping System

### Problem Identified
When adding logos for movies with similar names (e.g., "Despicable Me" and "Despicable Me 2"), the application would save both logos to the same directory, causing the second logo to overwrite the first. This occurred due to overly permissive fuzzy string matching.

### Root Cause
The directory matching algorithm used a similarity threshold of 0.8 (80%), which allowed:
- "Despicable Me" (normalized: `despicableme`) to match "Despicable Me (2010)" (normalized: `despicableme2010`) at 85.7%
- "Despicable Me 2" (normalized: `despicableme2`) to ALSO match "Despicable Me (2010)" at 89.7%

Both movies would automatically save to the same directory, with the second overwriting the first.

### Solutions Implemented

#### 1. Increased Similarity Threshold (Immediate Fix)
- **Changed**: Similarity threshold from `0.8` to `0.9` (90%)
- **Impact**: Prevents false matches between similar titles
- **Trade-off**: More movies require manual directory selection
- **Location**: `app.py` line 439

#### 2. TMDb ID → Directory Mapping System (Long-term Solution)
A new intelligent mapping system that remembers user selections:

**How It Works:**
1. **First Time**: User selects a logo for a movie (may require manual directory selection if similarity < 90%)
2. **App Remembers**: System saves TMDb ID → Directory mapping to `tmdb_directory_mapping.json`
3. **Subsequent Times**: App checks mapping first, automatically uses saved directory
4. **Result**: User only needs to manually select directory ONCE per movie

**New Functions Added:**
- `load_directory_mapping()` - Loads saved mappings from JSON file
- `save_directory_mapping()` - Saves mappings to JSON file
- `get_mapped_directory(tmdb_id, media_type)` - Retrieves saved directory for a TMDb ID
- `save_mapped_directory(tmdb_id, media_type, directory_path)` - Saves new mapping

**Files Modified:**
- `app.py`:
  - Added JSON import
  - Added mapping functions (lines 53-98)
  - Updated `select_movie()` to pass TMDb ID to template
  - Updated `select_tv()` to pass TMDb ID to template
  - Updated `select_logo()` to check mappings first, save mappings on match
  - Updated `confirm_directory()` to save mappings when user manually selects

- `templates/logo_selection.html`:
  - Added hidden input field for `tmdb_id`

- `templates/select_directory.html`:
  - Added hidden input field for `tmdb_id`

- `.gitignore`:
  - Added `tmdb_directory_mapping.json` (contains local paths, shouldn't be committed)
  - Added `test_media/` (test directories)

### Example Mapping File
```json
{
  "movie_20352": "/kids-movies/Despicable Me (2010)",
  "movie_93456": "/kids-movies/Despicable Me 2 (2013)",
  "tv_1396": "/tv/Breaking Bad"
}
```

### Benefits
1. ✅ **Prevents Logo Overwriting**: Different movies save to different directories
2. ✅ **Learns from User Input**: Manual selections are remembered
3. ✅ **Automatic Going Forward**: After first selection, logos save automatically
4. ✅ **Self-Cleaning**: Invalid mappings (deleted directories) are automatically removed
5. ✅ **Persistent**: Mappings survive app restarts
6. ✅ **Volume-Safe**: Mapping file can be stored in Docker volume for persistence

### Testing
Comprehensive tests created:
- `test_matching.py` - Simulates directory matching logic
- `test_comparison.py` - Compares old (0.8) vs new (0.9) threshold behavior
- `test_mapping.py` - Validates TMDb ID mapping system

All tests pass successfully.

### Docker Deployment Notes
To persist mappings across container restarts, add volume mapping in `docker-compose.yml`:
```yaml
volumes:
  - ./tmdb_directory_mapping.json:/app/tmdb_directory_mapping.json
```

Or mount as part of a data directory:
```yaml
volumes:
  - ./data:/app/data
```

Then update `MAPPING_FILE` path in `app.py`:
```python
MAPPING_FILE = '/app/data/tmdb_directory_mapping.json'
```

### Migration Path
No migration needed. The mapping file is created automatically when first logo is saved. Existing installations will:
1. Start with empty mapping file
2. May require manual directory selection for movies (if similarity < 90%)
3. Build up mappings organically as users select logos
4. Automatically save to correct directories for previously-mapped movies

### Future Enhancements (Optional)
- Web UI to view/edit mappings
- Import/export mappings
- Bulk mapping from directory scan
- Confidence scores in logs
- Mapping analytics (most frequently mapped movies)
