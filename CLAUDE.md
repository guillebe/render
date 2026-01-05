# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Mediabunny Caption Burner** is a browser-based video processing application that burns captions directly onto videos locally in the browser. It supports:
- Local MP4/WebM file uploads
- HLS stream URLs (.m3u8)
- Audio preservation from both sources
- Output capped at 30 seconds for performance

All video encoding happens client-side using the [Mediabunny](https://esm.sh/mediabunny) library, with no server-side processing.

## Architecture

### Single-Page Application
The entire application is contained in `index.html` (812+ lines). It's a monolithic ES module application with:
- **UI Layer**: Dark-themed interface with tab switching (file upload vs. HLS stream)
- **Processing Pipeline**: Unified video encoding pipeline using requestVideoFrameCallback
- **Output**: MP4 files with burned captions and preserved audio

### Video Processing Strategy: Play-Through

Both file and HLS sources use the same **Play-Through** strategy:
1. Load video metadata (dimensions, duration, audio)
2. Setup canvas with adaptive scaling for mobile
3. Use `requestVideoFrameCallback()` to capture frames during playback
4. Draw caption text with adaptive sizing based on aspect ratio
5. Encode frames into MP4 using Mediabunny's `CanvasSource` and audio sources

### Key Design Points

**Mobile Canvas Limits**: Videos are scaled down if they exceed:
- Max width/height: 4096px
- Max area: 16MP (to support mobile devices)

**Audio Capture - Dual Strategy**:
- **File sources**: Use Web Audio API's `createMediaElementSource` → `createMediaStreamDestination`
  - Works reliably for MP4/WebM files
  - Provides decoded audio access
- **HLS sources**: Use `video.captureStream()` for direct audio stream capture
  - Avoids audio playback through speakers (iOS issue)
  - More reliable on iOS for capturing streaming audio
  - Falls back to `mozCaptureStream()` for Firefox

**Dimension Handling**: For HLS streams (especially iOS), dimensions may not be available immediately after manifest parsing. The code waits for `loadedmetadata` event to ensure valid dimensions before processing.

**Multiple Event Listeners**: When loading HLS for preview vs. processing, separate handlers are used to avoid listener conflicts.

## Common Development Tasks

### Testing the Application
1. Open `index.html` directly in a modern browser (no build step required)
2. Test file upload with an MP4 or WebM video
3. Test HLS with the default URL or another `.m3u8` stream
4. Monitor the browser console for detailed processing logs

### Debugging

**Console Logging**: The application logs comprehensive debug information with markers:
- `=== PIPELINE START ===` / `=== PIPELINE SUCCESS ===` / `=== PIPELINE ERROR ===`
- Frame processing logs every 30 frames
- Buffer type detection and blob size warnings
- Separate log messages for file vs HLS audio capture methods

**Key Debugging Areas**:
- **HLS Dimensions**: Check console logs for "Got dimensions from..." to verify mobile dimension delays are handled
- **HLS Audio Capture**: Look for "✓ Captured HLS audio track via captureStream" or warnings about missing audio
- **File Audio Capture**: Look for "✓ Captured file audio track via Web Audio API"
- **Blob Creation**: Buffer type detection logs help identify MP4 encoding issues

### Fixing iOS Audio Issues

Recent fixes addressed iOS-specific HLS audio problems:

1. **Audio playback during processing**: Fixed by using `captureStream()` instead of Web Audio API for HLS
   - Web Audio API routes audio through speaker route unnecessarily
   - `captureStream()` provides direct media stream access without playback

2. **Missing audio in output**: Fixed by proper audio track extraction from HLS streams
   - Ensures `getAudioTracks()[0]` is properly wrapped in `MediaStreamAudioTrackSource`
   - Audio is added to output via `output.addAudioTrack(audioTrack)`

3. **Preview muting**: Preview video is muted during processing to prevent any audio leakage

### Performance Considerations

- **Frame Capture**: Uses `requestVideoFrameCallback()` which is frame-rate aware and efficient
- **Canvas Scaling**: Adaptive scaling prevents mobile canvas overflow errors
- **Duration Cap**: 30-second maximum output prevents browser memory issues
- **Audio Processing**: Opus codec with default system sample rate

## Dependencies

**External Libraries** (via CDN):
- `mediabunny@1.25.8`: Client-side video encoding (ES modules from esm.sh)
- `hls.js@latest`: HLS stream support in non-Safari browsers

**Browser APIs**:
- `requestVideoFrameCallback()`: Frame-accurate video capture
- `Canvas 2D`: Frame drawing with caption overlay
- `Web Audio API`: Audio stream creation and manipulation for files
- `MediaStream API`: Audio track extraction for both files and HLS
- `captureStream()`: Direct media stream capture from video element (HLS)
- `Web Share API`: WhatsApp sharing (with fallback to WhatsApp Web)

## Important Implementation Details

### Caption Text Rendering
Text positioning and sizing adapt based on video aspect ratio:
- Vertical videos (< 1:1 aspect): Font size = 6% of height, position lower
- Horizontal videos: Font size = 8% of height, position lower (10% of height)
- White text with black outline for visibility

### Audio Setup Pattern
```javascript
if (sourceType === 'hls') {
    // Use captureStream() for HLS
    const stream = tempVideo.captureStream ? tempVideo.captureStream() : tempVideo.mozCaptureStream();
    const nativeAudioTrack = stream.getAudioTracks()[0];
    audioTrack = new MediaStreamAudioTrackSource(nativeAudioTrack, { ... });
} else {
    // Use Web Audio API for files
    const mediaElementSource = audioContext.createMediaElementSource(tempVideo);
    const mediaStreamDestination = audioContext.createMediaStreamDestination();
    mediaElementSource.connect(mediaStreamDestination);
}
```

### Mediabunny Output Pattern
```javascript
const output = new Output({ target: new BufferTarget(), format: new Mp4OutputFormat() });
const videoSource = new CanvasSource(canvas, { codec: 'avc', bitrate: 6_000_000 });
output.addVideoTrack(videoSource, { frameRate });
if (audioTrack) output.addAudioTrack(audioTrack);
await output.start();
// ... process frames with videoSource.add(currentTime, frameDuration)
await videoSource.close();
await output.finalize();
// ... finalBlob created from output.target.buffer
```

### HLS Native vs hls.js
- Safari/iOS: Uses native HLS support (set `video.src = url`)
- Other browsers: Uses hls.js library with worker threads enabled
