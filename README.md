# Client Side Caption Burner

This is a browser-based tool that allows you to "burn" captions directly onto video files or HLS streams. All video processing happens locally within your browser using the `mediabunny` library, ensuring privacy and speed without needing server-side processing.

## Features

- **Local Video Processing:** No file uploads to servers; everything is encoded right in your browser.
- **Multiple Sources:**
  - **File Upload:** Support for MP4 and WebM video files.
  - **HLS Stream:** Support for `.m3u8` streaming URLs (with HLS.js integration).
- **Audio Support:** Preserves audio from both files and HLS streams (re-encoded to Opus for maximum compatibility).
- **Caption Customization:** Add custom text to be burned into the video frames.
- **Adaptive Text:** Smart text positioning and sizing based on video orientation (vertical vs. horizontal).
- **Export & Share:** Download the processed video as an MP4 file or share it via WhatsApp (using Web Share API where supported).

## Technical Architecture

This application uses a unique **"Play-Through"** pipeline to ensure frame-accurate recording and audio synchronization:

1.  **Ingestion:**
    - **Files:** Loaded into a hidden video element via Blob URL.
    - **HLS:** Loaded into the video element using `hls.js`.
2.  **Capture:**
    - **Video:** Uses `requestVideoFrameCallback` to capture frames exactly when they are rendered by the browser's compositor. This eliminates frame skipping issues common with seeking.
    - **Audio:** Uses `video.captureStream()` to extract real-time audio, which is then fed into `MediaStreamAudioTrackSource`.
3.  **Encoding:**
    - **Video:** Encoded to H.264 (AVC) via `mediabunny.CanvasSource`.
    - **Audio:** Encoded to Opus via `mediabunny.MediaStreamAudioTrackSource`.
    - **Muxing:** Combined into an MP4 container.

## How to Use

1. **Open the Application:**
   Simply open the `index.html` file in a modern web browser.

2. **Select Video Source:**
   - **Upload Video:** Click the "Upload Video" tab and select a video file from your device.
   - **HLS Stream:** Click the "HLS Stream" tab and paste a valid `.m3u8` URL.

3. **Add Caption:**
   Enter your desired text in the "Enter caption text" field.

4. **Process:**
   Click the **⚡ Process & Burn Captions** button. The application will play the video in the background at high speed (limited by your device's rendering power) to capture it.

5. **Save or Share:**
   Once processing is complete, you can watch the result in the "Burned Result" player. Use the buttons below it to download the video or share it.

## Technologies Used

- **HTML5 & CSS3:** For structure and styling (Dark mode UI).
- **JavaScript (ES Modules):** Core logic.
- **[Mediabunny](https://esm.sh/mediabunny):** Library for client-side video processing and encoding.
- **[HLS.js](https://github.com/video-dev/hls.js):** For playing and processing HLS streams in browsers that don't support it natively.
