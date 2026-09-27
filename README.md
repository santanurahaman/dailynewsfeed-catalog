# dailynewsfeed-catalog

Remote feed catalog for [DailyNewsFeed](https://github.com/santanurahaman/DailyNewsFeeds)
(SPEC §9.1). The app bundles a copy of this file at first install and checks this repo's `main`
branch for a newer `version` once per 24 hours (a plain conditional `GET` against
`raw.githubusercontent.com`, ETag-based, no authentication).

## Schema (SPEC §9.2)

```json
{
  "version": 2,
  "updatedAt": "2026-09-27T00:00:00Z",
  "categories": [
    { "id": "world", "name": "World", "order": 2 }
  ],
  "feeds": [
    {
      "url": "https://feeds.bbci.co.uk/news/world/rss.xml",
      "title": "BBC World",
      "siteUrl": "https://www.bbc.com/news",
      "categoryId": "world",
      "sourceWeight": 1.4,
      "paywalled": false,
      "language": "en",
      "region": "uk"
    }
  ]
}
```

- `version` — integer, must increase on every publish. A same-or-lower version than what the app
  already has is ignored.
- `categories[].id` — `top` is reserved (computed by the app, never assign it to a feed).
- `feeds[].sourceWeight` — `0.5`–`1.5`. Wire services and major outlets: `1.3`–`1.5`. Solid
  mainstream outlets: `1.0`–`1.2`. Aggregators/blogs: `0.5`–`0.9`.
- `feeds[].url` must be a valid `https://` (or auto-upgradable `http://`) URL.
- Every field beyond `url`/`title`/`categoryId`/`sourceWeight` is optional.

Any schema violation — a missing field, an out-of-range weight, a feed pointing at the reserved
`top` category, a duplicate feed URL — causes the app to reject the **whole file**, never a partial
update. See `CatalogParser`/`CatalogMerger` in the main app repo for the exact validation and merge
rules (remote wins on `title`/`categoryId`/`sourceWeight`/`paywalled`/`icon`; a feed dropped from
this file is disabled in the app, not deleted; user-added feeds are never touched by this catalog).

## Publishing an update

1. Edit `catalog.json`.
2. Bump `version` and `updatedAt`.
3. Commit and push to `main` (via a reviewed, merged PR — direct pushes to `main` are blocked, see
   below).

No app release is required — the next app that checks in within 24 hours of the push picks it up.

## Branch protection

`main` requires a pull request with at least one approval before merging (direct pushes and
unreviewed merges are blocked), so no external PR can land without the owner's explicit review —
see the repo's Settings → Branches.
