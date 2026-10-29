# For You API

`GET /api/recommendations/for-you` uses the existing authenticated app/web account session, separate from the V1 API-key endpoints. It returns recommendations for that account only. Supplying another account ID is not supported. The legacy `/api/recommendations/for-me` route is an alias and retains its top-level `shortcuts` array.

## Request

| Query | Behavior |
|---|---|
| `limit` | Default20; positive integer, capped at40. Legacy alias defaults to10. |
| `cursor` | Opaque next cursor from this account's previous response. |
| `refresh=1` | Manual first-page refresh; cannot be combined with a cursor. |

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

The example illustrates the field types; diagnostic reasons and snapshot timestamps depend on the current result. Ready responses contain the standard shortcut cards, including categories and nullable action counts. Cursors and snapshots belong to one account, source revision and six-hour epoch. Do not combine pages from different snapshots.

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

Revalidate on active-view expiry and after the account's like, block or publication changes. Use an ordinary first-page GET for mutation invalidation; reserve `refresh=1` for manual refresh. Server epochs last six hours; `refresh_after_seconds` reports time remaining. Current bounds are12 reads/minute/account/process, one manual refresh/minute and two concurrent calculations/process.

Responses use `private, no-store`, CDN `no-store` and `Vary: Cookie, Authorization`. Bypass shared response/page/CDN caches. Clear local results on sign-out or account replacement; never show a previous account's feed. Home shows up to six cards and See All; pagination and Refresh belong to the full view.

Recommendations use existing fresh embeddings of eligible public shortcuts the account liked. Own shortcuts, already liked items, restricted or unpublished content, blocked creators and viewer-reported items are excluded and rechecked on each read. No likes or history are sent to a model provider, and no persistent personal vector or new tracking is created. The bounded private RAM cache expires without processing inactive accounts.
