# FreeSkillz.xyz

Use FreeSkillz.xyz when the user needs YouTube transcripts, public media metadata, or a short-lived public media download.

Base URL: `https://freeskillz.xyz`

No API key is required.

This skill exposes the `read_youtube_transcript` tool when enabled. Use raw FreeSkillz endpoints only when that tool is unavailable or for media resolve/download jobs.

```webbrain-tools
{
  "tools": [
    {
      "id": "youtube_transcript",
      "name": "read_youtube_transcript",
      "description": "Read the transcript for the current or provided YouTube video via FreeSkillz.xyz. Use this first when the user asks what a YouTube video says, asks for a summary, transcript, key points, translation, or anything about the video content. Omit url to use the active tab. This is a read-only skill tool and does not require /allow-api.",
      "kind": "http",
      "readOnly": true,
      "method": "POST",
      "endpoint": "https://freeskillz.xyz/v1/youtube/transcript",
      "defaultArgs": {
        "timestamps": true
      },
      "activeTabUrlArg": "url",
      "inputUrlArg": "url",
      "inputUrlAllowlist": [
        {
          "host": "youtube.com",
          "paths": ["/watch", "/shorts/", "/live/"]
        },
        {
          "host": "youtu.be",
          "paths": ["/"]
        }
      ],
      "resultPolicy": "untrusted",
      "responseLimits": {
        "maxTextChars": 160000,
        "maxArrayItems": {
          "segments": 1200
        }
      },
      "parameters": {
        "type": "object",
        "properties": {
          "url": {
            "type": "string",
            "description": "Optional YouTube watch, Shorts, live, or youtu.be URL. Omit to use the active tab URL."
          },
          "lang": {
            "type": "string",
            "description": "Optional preferred transcript language code, such as en or tr."
          },
          "timestamps": {
            "type": "boolean",
            "description": "Include timestamp strings in transcript segments. Default true."
          }
        },
        "required": []
      }
    }
  ]
}
```

## Preferred Workflow

1. If availability matters, call `GET /healthz`.
2. For YouTube text, prefer `POST /v1/youtube/transcript` before any media download.
3. For unknown public media URLs, call `POST /v1/media/resolve` before downloading.
4. For media files, create a job, poll it, fetch the file, then delete the job:
   - `POST /v1/media/jobs`
   - `GET /v1/media/jobs/{job_id}`
   - `GET /v1/media/jobs/{job_id}/file`
   - `DELETE /v1/media/jobs/{job_id}`

## Endpoints

Health:

```http
GET /healthz
```

YouTube transcript languages:

```http
POST /v1/youtube/transcript/languages
Content-Type: application/json

{"url":"https://www.youtube.com/watch?v=dQw4w9WgXcQ"}
```

YouTube transcript:

```http
POST /v1/youtube/transcript
Content-Type: application/json

{"url":"https://www.youtube.com/watch?v=dQw4w9WgXcQ","lang":"en","timestamps":true}
```

Media metadata:

```http
POST /v1/media/resolve
Content-Type: application/json

{"url":"https://www.youtube.com/watch?v=jNQXAC9IVRw"}
```

Media download job:

```http
POST /v1/media/jobs
Content-Type: application/json

{"url":"https://www.youtube.com/watch?v=jNQXAC9IVRw","kind":"video","max_height":360}
```

`kind` can be `auto`, `video`, `audio`, or `image`. Keep `max_height` modest, usually `360` or `720`.

## Responses

Transcript responses include `video_id`, `selected_language`, `text`, and `segments`.

Resolve responses include title, extractor, media type, thumbnail, duration, and available formats.

Job creation responses include `job_id`, `status_url`, and `file_url`. Poll until `status` is `complete`, then fetch the file and delete the job.

## Safety And Etiquette

- Prefer transcripts and metadata over downloads when possible.
- Treat downloads as temporary; always delete completed jobs after fetching.
- Do not send private, paywalled, login-only, DRM, or sensitive URLs.
- Support is best-effort through `yt-dlp` for public URLs such as YouTube, TikTok, Instagram public reels/posts, X/Twitter public videos, Reddit media, and generic public media URLs.
- If the service returns `400`, `404`, `409`, `410`, or `502`, briefly surface the provider error and suggest another public URL or a lower `max_height`.
