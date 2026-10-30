# For You API

`GET /api/recommendations/for-you` uses the existing authenticated app/web account session, separate from the V1 API-key endpoints. It returns recommendations for that account only. Supplying another account ID is not supported. The legacy `/api/recommendations/for-me` route is an alias and retains its top-level `shortcuts` array.

## Request

| Query | Behavior |
|---|---|
| `limit` | Default20; positive integer, capped at40. Legacy alias defaults to10. |
| `cursor` | Opaque next cursor from this account's previous response. |
| `refresh=0` or `refresh=1` | Accepted for older clients and ignored; neither forces recomputation. |

Unknown or repeated parameters return400. Requests require the current account's existing authentication; account restrictions apply regardless of client version.

## Response

```json
{
  "shortcuts": [],
  "feed": {
    "kind": "for_you",
    "state": "no_matches",
    "available": false,
    "personalized": true,
    "reason": null,
    "snapshot_id": null,
    "generated_at": null,
    "expires_at": null,
    "refresh_after_seconds": 21600,
    "next_cursor": null,
    "has_more": false,
    "limit": 20
  }
}
```

The example illustrates the field types; diagnostic reasons and snapshot timestamps depend on the current result. Ready responses contain the standard shortcut cards, including categories and nullable action counts. Cursors and snapshots belong to one account and server-generated six-hour snapshot. Do not combine pages from different snapshots.

| State | HTTP | Client behavior |
|---|---|---|
| `ready` |200| Show only when `available`, `personalized` and nonempty. |
| `needs_likes`, `no_matches` |200| Hide the entry and feed. |
| `sign_in_required` |401| Clear account-scoped results and hide. |
| `unavailable` |403/503| Hide; respect account restrictions and `Retry-After` when present. |
| `snapshot_expired` |409| Discard cursor and reload the first page once. |
| `rate_limited` |429| Respect `Retry-After`. |

Every non-ready result has an empty `shortcuts` array and `available=false`. Neither endpoint falls back to a general feed or promotional empty state. Errors and empty responses must not move the user's current navigation.

## Refresh and privacy

Ranking lasts six hours from server generation. Only a saved like/unlike change permits an earlier normal rebuild. Visibility, blocks, reports, publication and source freshness are rechecked on reads and filter or suppress cached results without reranking. Revalidate with ordinary first-page GETs after expiry or confirmed account changes; clients have no force-refresh control. `refresh_after_seconds` reports time remaining. Current bounds are12 reads/minute/account/process and two concurrent calculations/process, with one active read per account.

Responses use `private, no-store`, CDN `no-store` and `Vary: Cookie, Authorization`. Bypass shared response/page/CDN caches. Clear local results on sign-out or account replacement; never show a previous account's feed. Home shows up to six cards and See All; pagination belongs to the full view. Manual recommendation refresh is unavailable.

Recommendations use existing fresh embeddings of eligible public shortcuts the account liked. Own shortcuts, already liked items, restricted or unpublished content, blocked creators and viewer-reported items are excluded and rechecked on each read. No likes or history are sent to a model provider, and no persistent personal vector or new tracking is created. The bounded private RAM cache expires without processing inactive accounts.

The existing bounded RAM cache is process-local. A server restart or capacity eviction can require a cold rebuild; this is not a client refresh facility. A cursor whose snapshot is absent or expired returns409 without triggering ranking; restart from page one. No schema change is required.
