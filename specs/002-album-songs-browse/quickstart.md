# Quickstart: Album Songs Browse

**Feature**: 002-album-songs-browse
**Date**: 2026-02-08

## Overview

This feature adds the ability to browse songs within an album. When users tap on an album in either the Albums or Artist Albums views, they see a list of songs they can play individually or all at once.

## Key Changes

### Files Modified

| File | Changes |
|------|---------|
| `esphome/packages/browse.yaml` | Add category 7, album context globals, `browse_album_songs` script |
| `esphome/packages/navigation.yaml` | Update back navigation for category 7 |
| `esphome/ui/browse_unified.yaml` | Update tap handlers for categories 4 and 6 |

### New Category

- **Category 7 (AlbumSongs)**: Shows songs in selected album
- Title: Album name
- Items: Track number + song title
- Actions: Play individual song, Play All

## Testing Checklist

### Path 1: Albums → Album Songs

1. Navigate to Library → Albums
2. Tap any album
3. Verify album name appears in header
4. Verify songs listed with "1. Song Name" format
5. Verify Play All button visible
6. Tap a song → should play and navigate to Now Playing
7. Go back → should return to Albums at same position

### Path 2: Artists → Artist Albums → Album Songs

1. Navigate to Library → Artists
2. Tap any artist
3. Tap any album
4. Verify album songs screen appears
5. Tap Play All → should play album and navigate to Now Playing
6. Go back twice → should return to artist's albums, then artists list

### Edge Cases

- [ ] Empty album shows "No songs in this album"
- [ ] Long song titles truncate with ellipsis
- [ ] Back navigation preserves scroll position
- [ ] No player selected → error toast on tap

## Compile Commands

```bash
# Original CYD
esphome compile esphome/main.yaml

# Freenove S3
esphome compile esphome/main_freenove_s3.yaml
```

## Rollback

If issues occur, revert the tap handlers in `browse_unified.yaml` to call `play_media_item` directly instead of `browse_album_songs` for categories 4 and 6.
