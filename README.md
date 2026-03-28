# cam-lens-privacy
# RTSP Player - Ultra Low-Latency Android Streaming

A production-ready React Native application for playing RTSP streams with ultra-low latency on Android devices.

## Features

- **Ultra Low-Latency Streaming**: Target latency < 500ms
- **ExoPlayer Media3 Integration**: Native Kotlin module with RTSP extension
- **TCP Transport**: Reliable streaming over TCP (configurable)
- **Hardware Decoding**: Preferred hardware acceleration for performance
- **Auto-Reconnect**: Automatic reconnection with configurable retry logic
- **Real-time Statistics**: FPS, latency, bitrate, buffer health overlay
- **Snapshot Capture**: Capture current frame as PNG
- **Lifecycle Management**: Proper pause/resume handling
- **Clean Architecture**: Separation between UI, Bridge, and Native layers

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    React Native (TypeScript)                 │
├─────────────────────────────────────────────────────────────┤
│  App.tsx ─► RTSPPlayerView ─► useRTSPPlayer Hook           │
│                  │                    │                      │
│                  ▼                    ▼                      │
│          UI Components         RTSPPlayerBridge              │
│  (Controls, Status, Debug)          │                        │
└────────────────────────────────────┬┴───────────────────────┘
                                     │ Native Bridge
┌────────────────────────────────────▼────────────────────────┐
│                   Android Native (Kotlin)                    │
├─────────────────────────────────────────────────────────────┤
│  RTSPPlayerModule ─► RTSPPlayerManager ─► ExoPlayer        │
│        │                    │                │              │
│        ▼                    ▼                ▼              │
│  RTSPPlayerPackage   Event Listeners   RTSP MediaSource    │
│        │                                     │              │
│        ▼                                     ▼              │
│  RTSPSurfaceViewManager ──────────► SurfaceView            │
└─────────────────────────────────────────────────────────────┘
```

## Requirements

- Node.js 18+
- React Native 0.73+
- Android Studio Hedgehog (2023.1.1) or later
- Android SDK 34 (compileSdk)
- Android SDK 24+ (minSdk)
- JDK 17
- Gradle 8.2+

## Setup Instructions

### 1. Clone and Install Dependencies

```bash
cd RTSPPlayer
npm install
```

### 2. Android SDK Setup

Ensure you have the following installed via Android Studio SDK Manager:

- Android SDK Platform 34
- Android SDK Build-Tools 34.0.0
- Android NDK 26.1.10909125
- CMake 3.22.1

### 3. Environment Variables

Add to your `~/.bashrc` or `~/.zshrc`:

```bash
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/tools
```

### 4. Build and Run

```bash
# Start Metro bundler
npm start

# In another terminal, build and run
npm run android
```

### 5. Release Build

```bash
cd android
./gradlew assembleRelease
```

The APK will be at: `android/app/build/outputs/apk/release/app-release.apk`

## Configuration

### Player Configuration Options

```typescript
interface PlayerConfig {
  rtspUrl: string;              // RTSP stream URL
  autoPlay?: boolean;           // Auto-start playback (default: false)
  autoReconnect?: boolean;      // Auto-reconnect on disconnect (default: true)
  reconnectDelayMs?: number;    // Delay between reconnects (default: 3000)
  maxReconnectAttempts?: number;// Max reconnect attempts (default: 5)
  bufferDurationMs?: number;    // Buffer size in ms (default: 500)
  preferTcp?: boolean;          // Use TCP transport (default: true)
  enableDebugOverlay?: boolean; // Show debug stats (default: false)
  hardwareDecodingPreferred?: boolean; // Prefer HW decoder (default: true)
}
```

### Low-Latency Optimizations

The player is configured for minimal latency by default:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| MIN_BUFFER_MS | 100ms | Absolute minimum buffer |
| MAX_BUFFER_MS | 500ms | Maximum buffer size |
| BUFFER_FOR_PLAYBACK_MS | 50ms | Start playback quickly |
| REBUFFER_MS | 100ms | Minimal rebuffer on underrun |
| preferTcp | true | TCP for reliability |
| hardwareDecodingPreferred | true | Hardware acceleration |

## Usage

### Basic Usage

```tsx
import { RTSPPlayerView } from './src/components';

function MyPlayer() {
  return (
    <RTSPPlayerView
      initialUrl="rtsp://your-server/stream"
      autoPlay={true}
      showControls={true}
    />
  );
}
```

### With Hook for Custom Control

```tsx
import { useRTSPPlayer } from './src/hooks/useRTSPPlayer';

function CustomPlayer() {
  const {
    playerState,
    stats,
    isPlaying,
    play,
    stop,
    captureSnapshot,
  } = useRTSPPlayer({
    onError: (error) => console.error(error),
    onStats: (stats) => console.log('FPS:', stats.fps),
  });

  return (
    <View>
      <RTSPSurfaceView style={{ flex: 1 }} />
      <Button onPress={() => play('rtsp://...')} title="Play" />
    </View>
  );
}
```

### Ref-based Control

```tsx
import { RTSPPlayerRef } from './src/types';

function ControlledPlayer() {
  const playerRef = useRef<RTSPPlayerRef>(null);

  const handleCapture = async () => {
    const base64 = await playerRef.current?.captureSnapshot();
    console.log('Snapshot:', base64?.substring(0, 50));
  };

  return (
    <RTSPPlayerView
      ref={playerRef}
      initialUrl="rtsp://..."
    />
  );
}
```

## Example RTSP URLs

```
# Wowza test stream
rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mp4

# IP Camera (typical format)
rtsp://username:password@camera-ip:554/stream1

# ONVIF Camera
rtsp://camera-ip:554/onvif/device_service
```

## Gradle Dependencies

The following ExoPlayer Media3 dependencies are included:

```groovy
// Core ExoPlayer
implementation("androidx.media3:media3-exoplayer:1.2.1")

// RTSP extension (critical)
implementation("androidx.media3:media3-exoplayer-rtsp:1.2.1")

// UI components
implementation("androidx.media3:media3-ui:1.2.1")

// Common utilities
implementation("androidx.media3:media3-common:1.2.1")

// Session for background playback
implementation("androidx.media3:media3-session:1.2.1")

// Datasource for network
implementation("androidx.media3:media3-datasource:1.2.1")

// Decoders
implementation("androidx.media3:media3-decoder:1.2.1")
```

## Required Permissions

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

## Troubleshooting

### Stream Won't Connect

1. Check network connectivity
2. Verify RTSP URL format
3. Ensure `usesCleartextTraffic="true"` in AndroidManifest
4. Check firewall/security group rules

### High Latency

1. Reduce `bufferDurationMs` (minimum 100ms)
2. Enable TCP transport (`preferTcp: true`)
3. Ensure hardware decoding is enabled
4. Check network bandwidth

### Decoder Errors

1. Try `hardwareDecodingPreferred: false`
2. Update device firmware
3. Check codec compatibility

### App Crashes on Background

1. Ensure lifecycle methods are properly implemented
2. Check if `release()` is called on unmount

## Performance Metrics

The debug overlay shows real-time statistics:

- **FPS**: Frames per second (target: 25-30)
- **Latency**: Estimated stream delay in ms
- **Bitrate**: Current stream bitrate
- **Buffer Health**: Buffer fullness percentage
- **Resolution**: Video dimensions
- **Codec**: Active video codec
- **Transport**: TCP or UDP
- **Packets**: Received and lost count

## Scaling Recommendations

For production deployment with multiple streams:

### 1. RTSP to WebRTC using go2rtc

```yaml
# go2rtc.yaml
streams:
  camera1: rtsp://camera1-ip:554/stream
  camera2: rtsp://camera2-ip:554/stream

webrtc:
  candidates:
    - stun:stun.l.google.com:19302
```

### 2. Load Balancing

- Use nginx-rtmp module for RTSP proxying
- Deploy go2rtc instances behind HAProxy
- Consider MediaMTX for multi-protocol support

### 3. Edge Caching

- Deploy edge servers close to viewers
- Use CDN for HLS fallback
- Implement adaptive bitrate switching

## Project Structure

```
RTSPPlayer/
├── android/
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── java/com/rtspplayer/
│   │   │   │   ├── player/
│   │   │   │   │   ├── RTSPPlayerManager.kt      # Core player logic
│   │   │   │   │   ├── RTSPPlayerModule.kt       # RN bridge module
│   │   │   │   │   ├── RTSPSurfaceViewManager.kt # View manager
│   │   │   │   │   └── RTSPPlayerPackage.kt      # Package registration
│   │   │   │   ├── MainActivity.kt
│   │   │   │   └── MainApplication.kt
│   │   │   ├── res/
│   │   │   └── AndroidManifest.xml
│   │   └── build.gradle
│   ├── build.gradle
│   └── settings.gradle
├── src/
│   ├── components/
│   │   ├── RTSPPlayerView.tsx    # Main player component
│   │   ├── PlayerControls.tsx    # Control buttons
│   │   ├── StatusIndicator.tsx   # State display
│   │   ├── DebugOverlay.tsx      # Stats overlay
│   │   └── LoadingOverlay.tsx    # Loading indicator
│   ├── hooks/
│   │   └── useRTSPPlayer.ts      # React hook
│   ├── native/
│   │   └── RTSPPlayerBridge.ts   # Native bridge
│   ├── types/
│   │   └── index.ts              # TypeScript types
│   └── App.tsx                   # Main app
├── index.js
├── package.json
└── README.md
```

## License

MIT License

## Contributing

1. Fork the repository
2. Create feature branch
3. Submit pull request

---

Built with ExoPlayer Media3 and React Native
