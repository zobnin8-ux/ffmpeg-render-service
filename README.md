# FFmpeg Render Service

Small Node.js/Express service used by the `Сыр&Вкуснятина` media pipeline.

## Purpose

The service receives a Kling-generated video URL and an ElevenLabs/Cloudinary audio URL, downloads both files, combines them with FFmpeg, converts the result to a vertical 1080x1920 H.264/AAC MP4, and returns the final video file.

## Endpoints

### `GET /`
Health check. Returns:

`FFmpeg service is running`

### `POST /render`
Request body:

```json
{
  "video_url": "https://.../video.mp4",
  "audio_url": "https://.../audio.mp3"
}
```

The current n8n WF3 also sends `task_id`; the service does not need it for rendering.

The endpoint returns the rendered `final.mp4` file.

## FFmpeg command

```bash
ffmpeg -y -i video.mp4 -i audio.mp3 \
-filter_complex "scale=1080:-1,pad=1080:1920:(ow-iw)/2:(oh-ih)/2" \
-c:v libx264 -c:a aac -shortest final.mp4
```

## Pipeline

`n8n WF2 -> Kling + ElevenLabs/Cloudinary -> n8n WF3 callback -> this service /render -> Telegram`

## Deployment

Designed for Railway using the included `Dockerfile`. Railway supplies the `PORT` environment variable automatically.

After deployment, update the FFmpeg HTTP Request URL in n8n WF3 to:

`https://<railway-domain>/render`
