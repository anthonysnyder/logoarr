# Logo Overwriting Bug - SOLVED! ✅

## The Problem
When adding logos for movies with similar names (e.g., "Despicable Me" and "Despicable Me 2"), the application would save both logos to the same directory, causing overwrites.

## The Solution (Your Brilliant Idea!)
Pass the **exact directory name** from the initial card click through the entire flow!

## How It Works

### User Journey
1. **User clicks** "Despicable Me 2 (2013)" card
2. **Directory captured**: `"Despicable Me 2 (2013)"`
3. **Search TMDb**: Directory passed as URL parameter
4. **Select movie**: Directory passed through form
5. **Select logo**: Directory passed through form
6. **Save logo**: App uses **exact directory** from click!

### Code Flow
```
Movie Card Click
   ↓ (directory name)
triggerSearch('Despicable Me 2', 'movie', 'Despicable Me 2 (2013)')
   ↓
/search_movie?query=despicable me 2&directory=Despicable Me 2 (2013)
   ↓
search_results.html (passes directory in hidden field)
   ↓
/select_movie/93456?directory=Despicable Me 2 (2013)
   ↓
logo_selection.html (passes directory in hidden field)
   ↓
/select_logo (POST with directory='Despicable Me 2 (2013)')
   ↓
SAVES TO: /kids-movies/Despicable Me 2 (2013)/logo.png ✅
```

## Files Modified

### Templates (Frontend)
1. **`templates/index.html`**
   - Updated `triggerSearch()` to accept `directoryName` parameter
   - Passes directory name in URL: `&directory=${encodeURIComponent(directoryName)}`

2. **`templates/tv.html`**
   - Same updates as index.html for TV shows

3. **`templates/search_results.html`**
   - Added hidden input: `<input type="hidden" name="directory" value="{{ directory }}">`

4. **`templates/logo_selection.html`**
   - Added hidden input: `<input type="hidden" name="directory" value="{{ directory }}">`

### Backend (Python)
5. **`app.py`**
   - `search_movie()`: Captures `directory` parameter, passes to template
   - `search_tv()`: Captures `directory` parameter, passes to template
   - `select_movie()`: Captures `directory` parameter, passes to template
   - `select_tv()`: Captures `directory` parameter, passes to template
   - `select_logo()`: **Uses directory parameter FIRST** (priority #1)

## Priority Order in select_logo()

The app now tries three methods in order:

1. **✅ FIRST (NEW!)**: Use directory from original click
   - If directory parameter exists → find exact folder → save immediately
   - **No fuzzy matching needed!**

2. **✅ SECOND**: Check TMDb ID mapping
   - If previously saved → use that directory

3. **✅ THIRD**: Fuzzy matching (fallback)
   - Threshold raised to 0.9 (90%) to prevent false matches
   - Only used if directory parameter missing

## Test Results

### Flow Test
```
✅ Directory passed through entire flow
✅ Exact folder found from original click
✅ Logo saves to correct location
✅ No fuzzy matching required
```

### Dual Movie Test
```
✅ Despicable Me    → /kids-movies/Despicable Me (2010)
✅ Despicable Me 2  → /kids-movies/Despicable Me 2 (2013)
✅ Different directories - NO OVERWRITING!
```

## Benefits

### Before (Buggy)
- ❌ Fuzzy matching with 80% threshold
- ❌ "Despicable Me 2" matched "Despicable Me (2010)" at 89.7%
- ❌ Both movies saved to same folder → overwriting!

### After (Fixed)
- ✅ Exact directory from user's click
- ✅ No risk of wrong folder
- ✅ Fast (no string comparison)
- ✅ Simple and reliable
- ✅ TMDb mapping still saved for future

## Edge Cases Handled

1. **Directory parameter missing** → Falls back to TMDb mapping or fuzzy match
2. **Directory doesn't exist** → Falls back to TMDb mapping or fuzzy match
3. **Multiple base folders** → Checks all of them (movies, kids-movies, etc.)
4. **Future updates** → TMDb mapping remembers for next time

## Deployment Ready

All changes are:
- ✅ Tested locally
- ✅ Backward compatible (falls back gracefully)
- ✅ Works with existing folders
- ✅ No database changes needed
- ✅ No breaking changes

## How to Deploy

### Option 1: Copy Updated Files
Replace these files on your production server:
- `app.py`
- `templates/index.html`
- `templates/tv.html`
- `templates/search_results.html`
- `templates/logo_selection.html`

### Option 2: Build New Docker Image
```bash
docker build -t logoarr:fixed .
docker-compose down
docker-compose up -d
```

### Option 3: Git Pull (if using git)
```bash
git pull origin main
# Restart your app
```

## No User Training Needed!

Users don't need to change anything - the workflow is identical:
1. Click movie card
2. Search TMDb
3. Select movie
4. Pick logo
5. Done!

The fix is invisible to users but prevents the overwriting bug completely!

---

**Status**: ✅ COMPLETE AND TESTED
**Risk Level**: LOW (backward compatible, graceful fallbacks)
**User Impact**: POSITIVE (fixes bug, no workflow changes)
