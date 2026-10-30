# Search and related shortcuts

Search can combine the original public listing embedding with embeddings of its current saved translations and a supplemental deterministic workflow signal. Missing or stale supplemental vectors retain the original metadata and lexical fallback. This feature does not generate translations or call a generative model.

Saved translations are versioned by source content, locale, translation model/pipeline and saved text. Results use the best matching metadata variant for one shortcut; extra translations never create duplicate cards or a score bonus. Existing saved display fields remain separate from retrieval scores.

The workflow signal contains only allowlisted action labels and structural/type relationships from exact accepted workflow bytes. User literals, scripts, URLs, credentials and binary payloads are excluded. Unknown or unsupported workflows remain metadata-only. Search and detail requests do not download or execute workflows, run a scan or infer new vectors per card.

Exact-title navigation and stronger metadata intent remain protected. Similarity uses public current source content, applies current account/block/publication restrictions, and limits near-duplicate or same-creator results. Missing strong matches may leave fewer cards.

[For You](FOR-YOU-API.md) uses liked public shortcut embeddings with separate interests and relevant creator diversity. Comparably relevant candidates rotate deterministically per account and server cycle. Its server-owned six-hour snapshot, like/unlike exception, stable pagination, private cache guards and hidden-unavailable behavior remain authoritative. There is no general feed fallback, click/search tracking or recommendation history.

Supplemental fill is resumable and only queues missing or outdated public documents. Existing valid vectors are reused. Coverage improves as the bounded existing model worker completes jobs; configuration activation alone does not mean every translation or workflow already has a vector.
