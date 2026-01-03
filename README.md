# Mediabunny Caption Burner

This is a browser-based tool that allows you to "burn" captions directly onto video files or HLS streams. All video processing happens locally within your browser using the `mediabunny` library, ensuring privacy and speed without needing server-side processing.

## Features

- **Local Video Processing:** No file uploads to servers; everything is encoded right in your browser.
- **Multiple Sources:**
  - **File Upload:** Support for MP4 and WebM video files.
  - **HLS Stream:** Support for `.m3u8` streaming URLs (with HLS.js integration).
- **Caption Customization:** Add custom text to be burned into the video frames.
- **Live Preview:** View the original video and the burned result side-by-side.
- **Adaptive Text:** Smart text positioning and sizing based on video orientation (vertical vs. horizontal).
- **Export & Share:** Download the processed video as an MP4 file or share it via WhatsApp (using Web Share API where supported).

## How to Use

1. **Open the Application:**
   Simply open the `index.html` file in a modern web browser.

2. **Select Video Source:**
   - **Upload Video:** Click the "Upload Video" tab and select a video file from your device.
   - **HLS Stream:** Click the "HLS Stream" tab and paste a valid `.m3u8` URL.

3. **Add Caption:**
   Enter your desired text in the "Enter caption text" field.

4. **Process:**
   Click the **⚡ Process & Burn Captions** button. The application will process the video frame-by-frame.

5. **Save or Share:**
   Once processing is complete, you can watch the result in the "Burned Result" player. Use the buttons below it to download the video or share it.

## Technologies Used

- **HTML5 & CSS3:** For structure and styling (Dark mode UI).
- **JavaScript (ES Modules):** Core logic.
- **[Mediabunny](https://esm.sh/mediabunny):** Library for client-side video processing and encoding.
- **[HLS.js](https://github.com/video-dev/hls.js):** For playing and processing HLS streams in browsers that don't support it natively.
