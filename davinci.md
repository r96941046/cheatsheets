# Davinci Resolve 4K slog2 Workfow

## Import Phase

1. Playback Memory export
2. Copy raw files into working disk

## Config Phase (in Davinci Resolve)

1. Make sure timeline settings
  - Project Settings
  - Timeline Format
  - 1920x1080 at 24fps
2. Change optimized media location to working hard disk
  - Project Settings
  - Master Settings
  - Working Folders, change both
3. Set color space transformations
  - Project Settings
  - Color Management
  - Color Space & Transforms
    - Color science: `ACEScc`
    - ACES Input Device Transform: `No Input Transform` (apply manually clip-wise)
    - ACES Output Device Transform: `Rec.709`
4. Add media into the media pool, without change timeline settings when prompted
5. Select all footage in media pool, and
  - apply ACES input transform: `Sony slog2`
  - Clip Attributes
    - Video Frame Rate: 24
    - Data Levels: `Full`
  - Generate optimized media

## Edit Phase (editing in timeline)

1. Go to Playback
  - check `Use optimized media if available`
  - [Optional] Proxy Mode: `Half resolution`
  - [Optional] Render Cache: `Smart`

## Export Phase

1. Go to Playback
  - uncheck `Use optimized media if available`
  - Proxy Mode: `Off`
2. Export with 4K format
  - check `Force sizing to higher quality`
  - check `Force debayer res to highest quality`
  - uncheck `Use optimized media`
  - uncheck `Use render cached images`
3. [Experimental] change to 4K timeline
