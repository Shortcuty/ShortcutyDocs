# Discovery criteria and public version comparisons

These additive APIs use explicit schema, writer and read readiness checks.
Required-app results include only verified current metadata; missing evidence
stays unknown. Historical facts bind exact approved public update identities.

## Discovery

The existing opt-in `GET /api/discover` retains `app` (case-sensitive bundle ID),
`category` (canonical category, including secondary-category membership), `q`,
`sort`, `per_page` and filter-bound `cursor`. It adds `min_actions` and
`max_actions`: inclusive decimal integers in `0…2147483647`, with minimum no larger
than maximum. Unknown counts do not match a count filter. Known zero does.
Duplicate, unsupported, malformed and inverted parameters return 400.

`GET /api/discover/filters` returns contract version 1 and boolean `app`,
`category` and `action_count` capabilities. It accepts no query parameters.

Count filtering narrows with the accepted workflow state/candidate join, then
verifies the publication identity digest and exact response snapshot with the
existing accepted-publications reader. It never reads scan reports/private JSON,
chooses a latest candidate, or substitutes a pending count. Discovery capture,
page rechecks and app choices also enforce the existing ledger-derived permanent
deactivation policy for published metadata. Action-count bounds additionally
exclude temporary-banned authors, matching the released count reader. Capture remains
bounded at 1000 candidates, with existing truncation semantics. Count verification
uses batches of at most 100; it rechecks eligibility on every cursor page.
`applied_filters` includes both bounds, and cursor fingerprints bind them.
Serialization now uses the existing batch serializer, avoiding one count query
per card. Default, unfiltered capture still selects IDs only.

The existing source/approval/snapshot/app-index/coverage gates for discovery
remain mandatory. Count readiness alone does **not** activate discovery or prove
its older immutable app metadata/index storage is populated. This patch does not
migrate, attest, or backfill that pipeline. Legacy listings and semantic search
keep their existing routes and ranking.

## Comparison

`GET /api/shortcuts/<identifier>/versions/compare?from_update_id=21&to_update_id=22`
accepts exactly two distinct positive INT update IDs. Version labels are display
text, not identifiers. Non-V1 changelog entries add optional `update_id`; old
clients ignore it, and new clients tolerate omission/corrupt optional IDs.

The endpoint is default off (`VERSION_COMPARISON_ENABLED`). Both updates must
belong to the same currently approved, public shortcut and have approved status.
Owners/admins receive no broader comparison access. The parent must pass the
existing ledger-derived account-access policy both before and after evidence
reads; receipt writes/reads independently apply that policy. Ineligible, cross-shortcut,
pending, rejected and private targets return indistinguishable 404 responses.
Status, visibility and update rows are rechecked after optional evidence reads.
Responses use `Cache-Control: private, no-store`. Temporary bans preserve published release notes, while numeric counts and optional receipt evidence remain unknown; a new ban during reads suppresses already-read evidence before response.

The version-1 response contains `scope: approved_public_updates`, `before` and
`after` with update ID, version, exact stored release notes/date, nullable action
count and evidence availability. `changes` has `apps`,
`capabilities` and `url_hosts`, each with boolean `available`, and nullable
`added`/`removed` arrays. Unavailable arrays are null, never empty. Available empty
arrays mean no difference in recorded static evidence, not proof that a workflow
has no capability or network behavior.

`evidence_basis` is `static_published_workflow_evidence`, `runtime_consent` is
`unknown`, and `network_activity` is `not_observed`.
Capability labels reuse the existing rule-based permission extraction; they do
not establish actual iOS prompts, access grants or execution. URL hosts reuse
existing static URL extraction and can include URLs in workflow text. They are
not confirmed connections or an exhaustive destination inventory. Only normalized
HTTP(S) hostnames are retained: userinfo, port, path, query and fragment never
enter the ledger or response. Apps are bundle IDs; no new App Store lookups run.

## Approval-time evidence and rollout

The review-only `schema/public-version-evidence-proposal.sql` adds
`shortcut_public_version_evidence`. All three factory attestations default false:
`public_version_evidence_schema_ready`, `public_version_evidence_writer_ready`,
`public_version_evidence_reads_ready`. They require the existing accepted-workflow
writer/epoch; reads require schema and writer readiness first.

After an update is approved, the existing approval transaction may capture a
receipt only when the exact candidate has become the accepted publication, the
parent is approved/public, and the final approved update matches its artifact.
It stores the accepted count plus an optional bounded public projection of that
candidate's exact legacy catalog IDs, or the exact snapshot published on that
same approval cursor when the immutable metadata producer is enabled. That snapshot
must match the current parent pointer, accepted source hash, update ID, URL/version,
complete extraction and publication marker. Count-only candidates with no matching
evidence still create count-only receipts. It never fills facts from mutable live
links or another scan candidate.
Existing metadata snapshots do not prove historical public visibility and are
not used as fallback. Initial releases without an update ID, private approvals,
metadata refresh/recount operations, and older versions create no history receipt.

Receipts bind shortcut/update IDs, approved revision/time, accepted source and
workflow capability epoch. A matching retry is idempotent. A reused approval with
missing/different provenance permanently invalidates its earlier receipt rather
than reviving old evidence. Reads require the current epoch and an uninvalidated
receipt. Update/shortcut deletion cascades receipts. Reader/schema failure leaves
release notes available and historical facts unknown; no historical parse/scan is
authorized. The existing reader may prove the exact currently accepted update's
count independently, but cannot fill an older update by matching labels/URLs.

Before activation, the integration owner must review the new schema and approval
writer, prove MySQL rollback/locking/collation/cascade behavior in disposable
staging, upgrade all approval writers, assess coverage, and independently enable
backend reads plus website/app gates. If an older writer runs, keep reads disabled
until the capability epoch/provenance is reestablished; old epoch receipts cannot
be assumed valid. Do not rebuild historical evidence from private stages or
rescan merely to fill the UI. Disabling read/UI flags rolls back presentation;
retain the additive storage and approval-compatible writers.

## Interface

The app and website retain the existing home layout. App/category/action criteria
are disclosed inside one existing filter control. App choices/categories/capabilities
load only on opening it.
Action presets are any, 0–25, 26–100 and 101+. Clearing criteria restores the original
home feeds. Comparison opens on demand inside existing history, between adjacent
approved update IDs; it creates no permanent home section, card badge or toolbar.
Native comparisons additionally require `VERSION_COMPARISONS_READY`; discovery
continues to require `DISCOVERY_FILTERS_READY`. The website comparison flag also
defaults false. No build configuration enables either native flag in this patch.

Labels are supplied for all 13 existing supported locales, including Arabic.
Controls retain native labels, keyboard disclosure/focus and retry states; zero
and unknown counts are distinct. Native views were typechecked against the iOS
27.1 simulator SDK at deployment target iOS 26 with inert boundaries. The full app,
widget and test target also compile in an unsigned simulator build with feature
flags enabled only for verification. Live visual/VoiceOver verification remains
a release gate. See the handoff for exact source and build receipts.
