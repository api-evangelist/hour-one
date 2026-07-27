---
name: Create a video from a blueprint and await completion
description: Generate an AI presenter video from an existing Hour One project blueprint, register a webhook, and retrieve the finished video and its metadata.
api: openapi/hour-one-openapi.json
operations: [create_video_videos_post, get_video_by_id_videos__video_id__get, get_video_metadata_videos__video_id__metadata_get, create_webhook_webhooks_post]
generated: '2026-07-19'
method: generated
---

# Create a video from a blueprint

Use the Hour One (MakeReals) API to render an AI presenter video from a template ("blueprint") and get the result.

## Auth
- Base URL: `https://api.makereals.com/api/v1`
- Send the `api-key: <your key>` header on video operations. Only one API key is active per account; generating a new one revokes the old.
- Webhook management endpoints use an HTTP Bearer token instead of the `api-key` header.

## Steps
1. (Optional, recommended) Register a webhook with `create_webhook_webhooks_post` (`POST /webhooks`) subscribing to `video.ready` and `video.failed` so you are notified instead of polling. Store the returned `signing_secret`.
2. Create the video with `create_video_videos_post` (`POST /videos`). Provide the blueprint `template_id`, a `name`, and any per-scene overrides in `scenes`; attach a `correlation_id` if you want to group it for analytics. The response returns the video `id`, `status`, and `progress`.
3. Wait for completion. Either receive the `video.ready` webhook, or poll `get_video_by_id_videos__video_id__get` (`GET /videos/{video_id}`) until `status` is `ready` (terminal failure states: `failed`, `stopped`, `cancelled`).
4. Read `video_player_url` / `download_url` from the video response. For per-scene timing, call `get_video_metadata_videos__video_id__metadata_get` (`GET /videos/{video_id}/metadata`).

## Rules
- Verify every webhook: recompute `HMAC-SHA256(raw_body, signing_secret)` and constant-time compare against the `x-hourone-signature` header.
- No idempotency key exists — do not blindly retry `POST /videos`; on a network error, reconcile by listing videos or matching your `correlation_id` before re-creating.
- Validation errors return HTTP 422 with a FastAPI `{"detail":[{loc,msg,type}]}` envelope (not RFC 9457).
