# News Aggregator — High-Level Design

This repository contains a short high-level design for a news aggregator that collects articles from multiple publishers and shows related coverage as a single story.

## Requirements

- Ingest articles from RSS feeds, publisher APIs, and websites.
- Normalize articles into a common format.
- Detect duplicate or near-duplicate articles.
- Group articles about the same event into one story.
- Rank stories using recency, number of sources, source reliability, and relevance.
- Serve a duplicate-free news feed through APIs.

## High-Level Architecture

```mermaid
flowchart LR
    A[News Sources] --> B[Ingestion Service]
    B --> C[Message Queue]
    C --> D[Processing Workers]
    D --> E[Duplicate Detection]
    E --> F[Story Clustering]
    F --> G[(Article & Story Store)]
    G --> H[Ranking Service]
    H --> I[(Feed Cache)]
    I --> J[Feed API]
    J --> K[Web / Mobile Clients]
```

## Main Components

1. **Ingestion service:** Fetches articles and places them on a durable queue.
2. **Processing workers:** Clean content, standardize fields, and extract useful metadata.
3. **Duplicate detection:** Uses URL/content hashes first, followed by text similarity.
4. **Story clustering:** Groups different articles describing the same recent event.
5. **Ranking service:** Scores stories using source count, freshness, reliability, and relevance.
6. **Feed API and cache:** Returns fast, paginated, duplicate-free feeds to clients.

## Basic Data Model

- **Source:** `source_id`, `name`, `reliability_score`
- **Article:** `article_id`, `source_id`, `title`, `body`, `url`, `published_at`, `story_id`
- **Story:** `story_id`, `headline`, `summary`, `source_count`, `rank_score`, `updated_at`

## Processing Flow

1. Receive and normalize an article.
2. Reject an exact duplicate using a stable ID, canonical URL, or content hash.
3. Compare it with recent articles and stories.
4. Add it to a matching story or create a new story.
5. Recalculate the story score and refresh the cached feed.

## Reliability and Scale

- Partition queues and add workers to handle traffic spikes.
- Make article writes idempotent so retries do not create duplicates.
- Retry temporary failures and move repeatedly failing messages to a dead-letter queue.
- Keep one canonical story ID and deduplicate story IDs again before returning a feed.

## Possible Future Work

- Personalized ranking
- Multilingual clustering
- Better source-independence detection
- Cluster merge and split workflows
