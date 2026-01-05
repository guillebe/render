# Client Side Caption Burner

This is a browser-based tool that allows you to "burn" captions directly onto video files or HLS streams. All video processing happens locally within your browser using the `mediabunny` library, ensuring privacy and speed without needing server-side processing.

## Features

- **Local Video Processing:** No file uploads to servers; everything is encoded right in your browser.
- **Multiple Sources:**
  - **File Upload:** Support for MP4 and WebM video files.
  - **HLS Stream:** Support for `.m3u8` streaming URLs (with HLS.js integration and explicit Load button).
- **Audio Support:** Preserves audio from both files and HLS streams in the output video.
- **Caption Customization:** Add custom text to be burned into the video frames (default: "CAPTION").
- **Adaptive Text:** Smart text positioning and sizing based on video orientation (vertical vs. horizontal).
- **Dynamic Frame Rate:** Automatically detects and uses the actual frame rate from your video.
- **Output Duration Limit:** Automatically caps output to 30 seconds maximum for performance.
- **Export & Share:** Download the processed video as an MP4 file or share it via WhatsApp (using Web Share API where supported).

## Technical Architecture

The application uses **two different processing strategies** based on the source type to optimize performance and ensure audio synchronization:

### Strategy A: Seek-and-Snapshot (Files)
Used for local file uploads (MP4, WebM):
1. **Ingestion:** File loaded into hidden video element via Blob URL
2. **Metadata:** Extracts actual frame rate and duration from video track
3. **Audio Extraction:** Uses Mediabunny's `input.getPrimaryAudioTrack()` to access audio directly
4. **Frame Capture:** Seeks to precise timestamps and captures frames via canvas drawing
5. **Encoding:**
   - Video: H.264 (AVC) at detected frame rate
   - Audio: Re-encoded from extracted track
   - Muxing: Combined into MP4 container

### Strategy B: Play-Through (HLS)
Used for HLS streaming:
1. **Ingestion:** HLS stream loaded with native support (Safari) or `hls.js` (other browsers)
2. **Metadata:** Gets dimensions and duration from video element
3. **Audio Extraction:** Uses `video.captureStream()` with `MediaStreamSource` for real-time audio capture
4. **Frame Capture:** Uses `requestVideoFrameCallback()` to capture frames during playback
5. **Encoding:**
   - Video: H.264 (AVC) at 30 FPS (default for HLS)
   - Audio: Piped from captured media stream
   - Muxing: Combined into MP4 container

### Key Features
- **Frame-accurate:** Eliminates skipping and maintains smooth playback
- **Audio sync:** Real-time capture ensures audio/video alignment
- **Duration limiting:** Output automatically capped at 30 seconds for browser performance
- **Adaptive captions:** Text sizing and positioning responds to video aspect ratio

## How to Use

1. **Open the Application:**
   Simply open the `index.html` file in a modern web browser.

2. **Select Video Source:**
   - **Upload Video:** Click the "Upload Video" tab and select a video file from your device.
   - **HLS Stream:** Click the "HLS Stream" tab, paste a valid `.m3u8` URL, and click the "Load" button to load the stream.

3. **Add Caption:**
   Enter your desired text in the "Enter caption text" field (default: "CAPTION").

4. **Process:**
   Click the **⚡ Process & Burn Captions** button. The application will play the video in the background at high speed (limited by your device's rendering power) to capture it.

5. **Save or Share:**
   Once processing is complete, you can watch the result in the "Burned Result" player. Use the buttons below it to download the video or share it.

## Technologies Used

- **HTML5 & CSS3:** For structure and styling (Dark mode UI).
- **JavaScript (ES Modules):** Core logic.
- **[Mediabunny](https://esm.sh/mediabunny):** Library for client-side video processing and encoding.
- **[HLS.js](https://github.com/video-dev/hls.js):** For playing and processing HLS streams in browsers that don't support it natively.
