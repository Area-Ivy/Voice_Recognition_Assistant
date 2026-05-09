# Voice Recognition Assistant Application

This is a voice recognition and control application developed with Vue 3 and Element Plus, supporting speech-to-text and voice command control functions.

## Features

- **Voice Recognition Transcription**: Real-time conversion of speech to text, supports transcription mode
- **Voice Command Control**: Control music player and application features through voice commands
- **Music Player**: Integrated music playback functionality, supports basic controls (play, pause, skip tracks, etc.)
- **Text Operations**: Save, clear, export recognition results
- **History Records**: Save and restore historical session content
- **Theme Switching**: Supports light and dark theme modes

## Voice Command List

The application supports the following voice commands:

| Command Category | Voice Command | Function Description |
|---------|---------|---------|
| Music Control | "Play music" | Start playing music |
|         | "Pause music" | Pause current playback |
|         | "Continue playing" | Resume music playback |
|         | "Stop playing" | Stop and close the player |
|         | "Previous song" | Play previous song |
|         | "Next song" | Play next song |
| Volume Control | "Increase volume"/"Louder" | Increase volume |
|         | "Decrease volume"/"Softer" | Lower volume |
| Text Operations | "Clear text" | Clear and archive current text |
|         | "Save text" | Export current text as a file |
| Transcription Mode | "Transcribe" | Enter transcription mode |
|         | "End"/"Exit" | Exit transcription mode |
| Other Features | "Switch theme" | Toggle light/dark theme |

## Technology Stack

- Vue 3
- Element Plus UI Framework
- Web Speech API (Voice Recognition)

## Getting Started

### Install Dependencies

```bash
npm install
```

### Start Development Mode

```bash
npm run dev
```

## Project Structure

```
my-vue-voice-app/
├── public/         # Static resources
├── src/
│   ├── assets/     # Image and style resources
│   ├── components/ # Vue components
│   ├── App.vue     # Main application component
│   └── main.js     # Application entry
├── index.html      # HTML template
└── vite.config.js  # Vite configuration
```

## Browser Compatibility

This application uses the Web Speech API for voice recognition, please ensure you use a modern browser that supports this API (such as Chrome, Edge, etc.). 