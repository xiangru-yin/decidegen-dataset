# DecideGen — Traceable Bilingual Social-Media Corpus

[![status](https://img.shields.io/badge/status-prepared%20·%20release%20deferred-orange)]()
[![license](https://img.shields.io/badge/license-%5Bto%20be%20set%5D-lightgrey)]()

Companion dataset for **[Paper title]** — *[Journal]*, [Year].
The corpus supports the study of how *individual* user behaviour aggregates into *collective* information propagation on social media: it links each observed action to the actor's profile, historical expressions, and the upstream content it derives from, and provides the supervisory labels used to train the **DecideGen** agents and drive the **Context-Constrained Social Simulator (CCSS)**.

Code released alongside this dataset: **[decidegen-code]** (`[[code-repository URL]](https://github.com/xiangru-yin/decidegen-code)`).

---

## ⚠️ Release status (please read first)

This repository is **not yet publicly downloadable**.

- The dataset has been **prepared and assembled**, and de-identification is being finalised.
- **During peer review**, private access links are provided to the editors and reviewers **on request** (see [Access](#access)).
- **Upon publication**, the released artefacts will be made available through a public repository at **[data-repository URL] ([DOI])**.

**What can and cannot be redistributed.** This corpus is built from *publicly available* Weibo and Twitter records collected under the respective platform terms of service. **Raw platform content (verbatim posts, comments, reposts, profile pages) cannot be redistributed**, because that would violate the platforms' terms and expose personal data. The released artefacts are therefore **derived, de-identified records** (see [Contents](#contents)).

---

## Contents

The released dataset consists of derived records only. It contains **no raw scraped text**; long free-text fields are released as de-identified placeholders or length/statistical summaries where platform terms forbid redistribution.

| Artefact | Description |
|---|---|
| `events/` | Event catalogue: 174 public events with language tag, platform, event window, and the initial post identifier. |
| `records/` | Observed **behavioural records** (see [schema](#1-behavioural-records)). |
| `users/` | **User profiles** — 22 fields (see [schema](#2-user-profiles)). |
| `history/` | **Historical post** metadata — 28 fields (see [schema](#3-historical-post-records)). |
| `follow_edges/` | Anonymised directed follow edges (22,954,692 edges). |
| `supervision/` | Decision-task and content-generation supervision built by underlying action path inference (see [Supervision](#supervision-labels)). |
| `schema/` | Field dictionaries and enumerations (JSON). |

All user identifiers are **anonymised** (stable hashes); the mapping back to platform identifiers is not released.

---

## Dataset at a glance

| Quantity | Value |
|---|---:|
| Public events | **174** (149 Chinese · 25 English) |
| Users with profiles | **130,914** |
| Observed behavioural records | **209,460** |
| Historical posts | **4,469,501** (≈ 4.47 M) |
| Directed follow edges | **22,954,692** |
| Historical posts per user | up to **50** |
| Profile fields | **22** |
| Historical-post fields | **28** |
| Behavioural-record fields | **7** |
| Evaluation events (held-out) | **15** (10 Chinese · 5 English) |

Events span the Chinese microblogging platform **Weibo** and **Twitter**; content sampling was applied to large Weibo events.

---

## Data schema

### 1. Behavioural records

Each event's propagation trace is decomposed into a chronological sequence of behavioural records. Each record is traceable to its upstream content, interaction target, and propagation branch.

| Field | Description |
|---|---|
| `actor` | User responsible for the observed action. |
| `action_type` | Observable platform operation performed by the actor. |
| `interaction_target` | User or post targeted by a directed action. |
| `content` | Text produced by the actor / associated with the interaction. |
| `timestamp` | Time at which the action was recorded. |
| `cascade_position` | Link from the action to its upstream content and propagation branch. |
| `user_context` | Link to the actor's public profile, historical posts, and following relations. |

**Observed action taxonomy (normalised, cross-platform, 4 classes):** original posting · reposting (of an original post or of an existing repost) · commenting on a post · commenting on a comment.

Only outcomes directly identifiable from public platform records are stored as observations. Intermediate actions that platforms do not record — **browsing, page navigation, reading** — are **not** present here; their feasible paths are reconstructed downstream during supervision construction.

The simulator-level action space (used by CCSS and by the supervision labels) is finer-grained:
`browse`, `read_message`, `read_repost`, `read_detail`, `read_source_post`, `post`, `comment_post`, `repost_post`, `comment_comment`.

### 2. User profiles

22 fields per user, e.g.:

`user_id` · `user_nickname` · `user_gender` · `user_location` · `user_description` · `user_verified_reason` · `user_verified_type` · `user_education` · `user_weibo_num` · `user_following` · `user_followers` · `user_status_total_counter` · `user_domain` · `user_create_time` · `user_avatar_url` · `user_cover_url` · `user_ability` · `user_label_desc` · `user_mcn_desc` · `user_companyVerified` · `user_character_description` · `follow_network`

Notes:

- `user_id` is the **anonymous** primary key used to aggregate actions across posts.
- `user_character_description` is an **LLM-generated** summary of personality, occupation, stance and style; it is a *derived* field (the simulation-layer persona), not raw profile text.
- `follow_network` is a **derived** attribute constructed from follow lists, not a directly observed field.
- Image/cover URLs are retained only as anonymised placeholders in the public release.

### 3. Historical post records

28 fields of metadata per historical post (e.g. `id`, `text`, `full_text`, `created_at`, `reposts_count`, `comments_count`, `attitudes_count`, `hashtags`, `mentions`, `post_location`, `image_urls`, `video_urls`, …).

**Temporal restriction.** For each user–event pair, only posts published **before the user's first recorded participation in that event** are retained. This prevents expressions produced *after* participation from leaking into the historical-post input.

---

## Supervision labels

Supervision is built by reconstructing the *latent action path* from information exposure to the observed outcome (see the paper's **Algorithm 1 — Underlying action path inference**). Public data record only final outcomes, so candidate paths are constructed in the reconstructed dynamic environment and filtered by **environmental reachability** and **outcome consistency**; only feasible paths are kept. Each path is serialised into a multi-step sequence of **action · target · content · emotion**.

| Split | Instances | Chinese / English |
|---|---:|---|
| Decision-task set | **528,594** | 79.5% / 20.5% |
| Content-generation set | **181,187** | 69.7% / 30.3% |

- **Decision-task set** — action, interaction target, and sentiment requirement. Sentiment: `neutral` / `positive` / `negative`; non-content actions carry a neutral default.
- **Content-generation set** — the four content-producing actions only (`post`, `comment_post`, `repost_post`, `comment_comment`) with free-text output. Mean content length: 87 characters (Chinese) / 196 characters (English).

Sentiment labels: **3 classes**. English text is scored with `cardiffnlp/twitter-roberta-base-sentiment-latest`; Chinese text uses a dedicated classifier (UER-PY based), trained by unifying six Weibo sentiment datasets into the same three-class scheme.

---

## Construction pipeline (summary)

1. **Collection & sampling** — posts, reposts, comments and their nested interaction relations are collected per event; content sampling is applied to large Weibo events.
2. **Traceability** — *content relations* (post → derivative content) and *user–action relations* (actor → interaction) are retained, yielding chronological behavioural sequences traceable to their propagation branches.
3. **Profiles & history** — 22-field profiles, following relations, and up to 50 temporally filtered historical posts per user.
4. **Supervision** — candidate action paths are reconstructed under platform-specific recommendation/ranking rules and filtered for reachability and outcome consistency.

Full details are in the paper's **Materials and Methods** and the **Supplementary Methods**.

---

## Ethics, privacy, and licensing

- **Public data only.** Weibo and Twitter records that are publicly available; collected under the respective platform terms of service.
- **Anonymisation.** All user identifiers are replaced by stable hashes before analysis and before release; no raw identifiers are redistributed.
- **No human-subjects intervention.** The study involves no laboratory animals and no experimental intervention with human participants.
- **Data licence:** **[e.g. CC BY 4.0 — to be confirmed]**. Raw platform content is **excluded** from this licence because it cannot be redistributed.
- Governed by the institutional guidelines of **[Xi'an Jiaotong University]**.

---

## Access

- **During peer review:** request private access links from the corresponding author, **[corresponding author]**, at **[email]**.
- **Upon publication:** available through **[data-repository URL] ([DOI])**.

---

## Citation

If you use this dataset, please cite the paper:

```bibtex
@article{[key],
  title   = {[Paper title]},
  author  = {[Author list]},
  journal = {[Journal]},
  year    = {[Year]},
  doi     = {[DOI]}
}
```

---

## Contact

**[Xiangru Yin]** — **[xiangruyin@stu.xjtu.edu.cn]** · **[Xi'an Jiaotong University College of Artificial Intelligence]**

*(Placeholders in `[brackets]` are to be filled in before public release.)*
