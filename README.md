# Evolution Media Converter

This project is a microservice in Go that processes **audio**, **video**, and **image** files, converting them to various formats and returning the converted media (as base64 or S3 URL). The service accepts media files sent as **form-data**, **base64**, or **URL**, and includes advanced features like audio transcription and S3 storage integration.

## Requirements

Before starting, you'll need to have the following installed:

- [Go](https://golang.org/doc/install) (version 1.21 or higher)
- [Docker](https://docs.docker.com/get-docker/) (to run the project in a container)
- [FFmpeg](https://ffmpeg.org/download.html) (for audio processing)

## Installation

### Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/EvolutionAPI/evolution-audio-converter.git
cd evolution-audio-converter
```

### Install Dependencies

Install the project dependencies:

```bash
go mod tidy
```

### Install FFmpeg

The service depends on **FFmpeg** to convert audio, video, and image files. Make sure FFmpeg is installed on your system.

- On Ubuntu:

  ```bash
  sudo apt update
  sudo apt install ffmpeg
  ```

- On macOS (via Homebrew):

  ```bash
  brew install ffmpeg
  ```

- On Windows, download FFmpeg [here](https://ffmpeg.org/download.html) and add it to your system `PATH`.

### Configuration

Create a `.env` file in the project's root directory. Here are the available configuration options:

#### Basic Configuration

```env
PORT=4040
API_KEY=your_secret_api_key_here
```

#### Transcription Configuration

```env
ENABLE_TRANSCRIPTION=true
TRANSCRIPTION_PROVIDER=openai  # or groq
OPENAI_API_KEY=your_openai_key_here
GROQ_API_KEY=your_groq_key_here
TRANSCRIPTION_LANGUAGE=en  # Default transcription language (optional)
```

#### Storage Configuration

```env
ENABLE_S3_STORAGE=true
S3_ENDPOINT=play.min.io
S3_ACCESS_KEY=your_access_key_here
S3_SECRET_KEY=your_secret_key_here
S3_BUCKET_NAME=audio-files
S3_REGION=us-east-1
S3_USE_SSL=true
S3_URL_EXPIRATION=24h
```

### Storage Options

The service supports two storage modes for the converted media files:

1. **Base64 (default)**: Returns the media file encoded in base64 format
2. **S3 Compatible Storage**: Uploads to S3-compatible storage (AWS S3, MinIO, etc.) and returns a presigned URL

When S3 storage is enabled, the response will include a `url` instead of the media field (`audio`, `video`, or `image`):

```json
{
  "duration": 120,
  "format": "mp4",
  "url": "https://your-s3-endpoint/bucket/file.mp4?signature...",
  "transcription": "Transcribed text here..." // if transcription was requested (audio only)
}
```

If S3 upload fails, the service automatically falls back to base64 encoding.

## Running the Project

### Locally

To run the service locally:

```bash
go run main.go -dev
```

The server will be available at `http://localhost:4040`.

### Using Docker

1. **Build the Docker image**:

   ```bash
   docker build -t audio-service .
   ```

2. **Run the container**:

   ```bash
   docker run -p 4040:4040 --env-file=.env audio-service
   ```

## API Usage

### Authentication

All requests must include the `apikey` header with your API key.

### Endpoints

#### Process Audio

`POST /process-audio`

Converts audio files to different formats with optional transcription.

**Input formats**: MP3, WAV, M4A, OGG, FLAC, AAC, and more
**Output formats**: `ogg` (default), `mp3`, `wav`, `m4a`, `mp4`

Parameters:
- `file`: Audio file (form-data)
- `base64`: Base64 encoded audio data
- `url`: URL to download audio file
- `format`: Output format (default: `ogg`)
- `transcribe`: Enable transcription (`true` or `false`)
- `language`: Transcription language code (e.g., "en", "es", "pt")

#### Process Video

`POST /process-video`

Converts video files to different formats.

**Input formats**: MP4, AVI, MOV, MKV, WebM, FLV, and more
**Output formats**: `mp4` (default), `webm`, `avi`

Parameters:
- `file`: Video file (form-data)
- `base64`: Base64 encoded video data
- `url`: URL to download video file
- `format`: Output format (default: `mp4`)

#### Process Image

`POST /process-image`

Converts image files to different formats.

**Input formats**: JPG, PNG, GIF, BMP, TIFF, WebP, and more
**Output formats**: `jpg` (default), `png`, `webp`, `gif`

Parameters:
- `file`: Image file (form-data)
- `base64`: Base64 encoded image data
- `url`: URL to download image file
- `format`: Output format (default: `jpg`)

#### Transcribe Only

`POST /transcribe`

Transcribes audio without format conversion.

Parameters:
- `file`: Audio file (form-data)
- `base64`: Base64 encoded audio data
- `url`: URL to download audio file
- `language`: Transcription language code (optional)

### Example Requests

#### Audio Conversion with Transcription

```bash
curl -X POST -F "file=@audio.mp3" \
  -F "format=ogg" \
  -F "transcribe=true" \
  -F "language=en" \
  http://localhost:4040/process-audio \
  -H "apikey: your_secret_api_key_here"
```

#### Video Conversion

```bash
curl -X POST -F "file=@video.mov" \
  -F "format=mp4" \
  http://localhost:4040/process-video \
  -H "apikey: your_secret_api_key_here"
```

#### Image Conversion

```bash
curl -X POST -F "file=@image.png" \
  -F "format=webp" \
  http://localhost:4040/process-image \
  -H "apikey: your_secret_api_key_here"
```

#### Base64 Upload (Audio)

```bash
curl -X POST \
  -d "base64=$(base64 audio.mp3)" \
  -d "format=ogg" \
  http://localhost:4040/process-audio \
  -H "apikey: your_secret_api_key_here"
```

#### URL Upload (Video)

```bash
curl -X POST \
  -d "url=https://example.com/video.mp4" \
  -d "format=webm" \
  http://localhost:4040/process-video \
  -H "apikey: your_secret_api_key_here"
```

#### Transcription Only

```bash
curl -X POST -F "file=@audio.wav" \
  -F "language=pt" \
  http://localhost:4040/transcribe \
  -H "apikey: your_secret_api_key_here"
```

### Response Formats

#### Audio Response

With S3 storage disabled (default):
```json
{
  "duration": 120,
  "format": "ogg",
  "audio": "UklGR... (base64 of the file)",
  "transcription": "Transcribed text here..." // if requested
}
```

With S3 storage enabled:
```json
{
  "duration": 120,
  "format": "ogg",
  "url": "https://your-s3-endpoint/bucket/file.ogg?signature...",
  "transcription": "Transcribed text here..." // if requested
}
```

#### Video Response

With S3 storage disabled (default):
```json
{
  "duration": 300,
  "format": "mp4",
  "video": "AAAAIGZ0eXB... (base64 of the file)"
}
```

With S3 storage enabled:
```json
{
  "duration": 300,
  "format": "mp4",
  "url": "https://your-s3-endpoint/bucket/file.mp4?signature..."
}
```

#### Image Response

With S3 storage disabled (default):
```json
{
  "format": "webp",
  "image": "/9j/4AAQSkZJRgABA... (base64 of the file)"
}
```

With S3 storage enabled:
```json
{
  "format": "webp",
  "url": "https://your-s3-endpoint/bucket/file.webp?signature..."
}
```

#### Transcription Only Response

```json
{
  "transcription": "This is the transcribed text from the audio file."
}
```

## Supported Formats

### Audio
- **Input**: MP3, WAV, M4A, OGG, FLAC, AAC, WMA, AIFF, AU, and more
- **Output**: 
  - `ogg` - Opus codec, high quality/low size (default)
  - `mp3` - Universal compatibility
  - `wav` - Uncompressed audio
  - `m4a` - AAC in MP4 container
  - `mp4` - For audio extracted from video

### Video
- **Input**: MP4, AVI, MOV, MKV, WebM, FLV, WMV, 3GP, MPEG, and more
- **Output**:
  - `mp4` - H.264 + AAC, web-optimized (default)
  - `webm` - VP9 + Opus, for modern browsers
  - `avi` - H.264 + MP3, broad compatibility

### Image
- **Input**: JPG, PNG, GIF, BMP, TIFF, WebP, SVG, PSD, and more
- **Output**:
  - `jpg` - Lossy compression, great for photos (default)
  - `png` - Lossless, supports transparency
  - `webp` - Modern format, smaller file sizes
  - `gif` - Animations and simple transparency

## Advanced Features

### Audio Transcription
- **Providers**: OpenAI Whisper, Groq
- **Languages**: Supports multiple languages (en, es, pt, fr, de, it, etc.)
- **Models**: 
  - OpenAI: `whisper-1`
  - Groq: `whisper-large-v3-turbo` (faster, cost-effective)

### S3 Storage Integration
- **Compatible with**: AWS S3, MinIO, DigitalOcean Spaces, Wasabi, and other S3-compatible services
- **Features**: Automatic presigned URL generation with configurable expiration
- **Fallback**: Automatic fallback to base64 if upload fails

### Error Handling
- Comprehensive error messages with FFmpeg details
- Automatic cleanup of temporary files
- Graceful fallback mechanisms

## Performance Optimizations

- **Temporary Files**: Uses temporary files for video processing to handle complex formats
- **Buffer Pool**: Reuses buffers to reduce memory allocation
- **FFmpeg Optimization**: Configured with optimal parameters for each media type
- **Concurrent Processing**: Handles multiple requests simultaneously

## Troubleshooting

### Common Issues

1. **FFmpeg not found**
   ```
   exec: "ffmpeg": executable file not found in $PATH
   ```
   **Solution**: Install FFmpeg and ensure it's in your system PATH.

2. **Video conversion errors**
   ```
   error during video conversion: exit status 183
   ```
   **Solution**: The service now uses temporary files to handle complex video formats. Make sure you have sufficient disk space.

3. **Large file uploads**
   - For large files, consider using the `url` parameter instead of `file` upload
   - Increase your server's upload limits if needed

4. **Transcription errors**
   ```
   transcription is not enabled
   ```
   **Solution**: Set `ENABLE_TRANSCRIPTION=true` and configure your API keys.

### Best Practices

- **Format Selection**: 
  - Use `ogg` for audio (best compression)
  - Use `mp4` for video (best compatibility)
  - Use `webp` for images (best compression)

- **Performance**:
  - Use S3 storage for production environments
  - Monitor disk space when processing large files
  - Set appropriate timeout values for large file processing

- **Security**:
  - Always use HTTPS in production
  - Rotate your API keys regularly
  - Validate file types before processing

## Development

### Building from Source

```bash
go build -o media-converter main.go
./media-converter -dev
```

### Docker Development

```bash
docker-compose up --build
```

### Testing the API

Use the provided examples or test with curl:

```bash
# Test audio conversion
curl -X POST -F "file=@test.mp3" -F "format=ogg" \
  http://localhost:4040/process-audio \
  -H "apikey: your_key"

# Test video conversion  
curl -X POST -F "file=@test.mp4" -F "format=webm" \
  http://localhost:4040/process-video \
  -H "apikey: your_key"

# Test image conversion
curl -X POST -F "file=@test.jpg" -F "format=webp" \
  http://localhost:4040/process-image \
  -H "apikey: your_key"
```

## License

This project is licensed under the [MIT](LICENSE) license.
