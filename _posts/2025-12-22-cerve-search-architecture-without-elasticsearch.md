---
layout: post
title: "Cerve's Sub-Second Search Feature Without Elasticsearch"
date: 2025-12-22 10:00:00 +0530
description: How we built a six-strategy intelligent search system using PostgreSQL, Spring WebFlux, R2DBC, and Kotlin Coroutines that delivers sub-second response times
img: # Add image if you have one
tags: [Cerve, PostgreSQL, Search, Architecture, Kotlin, Spring WebFlux, Performance]
---

**Engineering at Cerve** | Published December 11, 2025

---

## Abstract

When building Cerve's ONDC marketplace platform, we faced a critical architectural decision: how to implement fast, intelligent product search across lacs of grocery items. While the industry standard points toward Elasticsearch, we took a different path and the sole reason for this was to save cost associated with ElasticSearch. So we dug deeper in leveraging PostgreSQL's advanced search capabilities with a reactive, multi-strategy search architecture. This post details our journey, the trade-offs we made, and the surprising performance outcomes that validated our approach.

**TL;DR**: We built a six-strategy intelligent search system using PostgreSQL, Spring WebFlux, R2DBC, and Kotlin Coroutines that delivers sub-second response times with 99th percentile latencies under 200ms, while reducing operational complexity and infrastructure costs by eliminating the need for a separate search service.

---

## Table of Contents

1. [The Challenge](#the-challenge)
2. [The Elasticsearch Temptation](#the-elasticsearch-temptation)
3. [Our Solution: PostgreSQL-Powered Multi-Strategy Search](#our-solution)
4. [Deep Dive: Architecture & Implementation](#architecture)
5. [Performance Benchmarks](#performance)
6. [What We Gained](#benefits)
7. [What We Sacrificed](#trade-offs)
8. [Lessons Learned](#lessons)
9. [When Should You Choose This Approach?](#recommendations)

---

## <a name="the-challenge"></a>The Challenge

Cerve's ONDC marketplace platform serves small grocery store owners—shopkeepers who run neighborhood kirana stores, local supermarkets, and mom-and-pop shops. These are the backbone of India's retail economy, but they face a significant technological barrier when going digital.

### The Seller Onboarding Problem

Our target customers are small store owners who:
- **Lack technical expertise**: Most are not tech-savvy and have limited formal education
- **Manage large inventories**: A typical store has 1,000-5,000 unique grocery items
- **Have limited time**: Running a store is demanding; they can't spend days on digital onboarding
- **Work in vernacular environments**: Often think in regional languages but type in English

**The nightmare scenario**: Asking these sellers to manually create 1,000+ product listings—entering product names, brands, descriptions, prices, images, nutritional info, FSSAI details, and HSN codes for each item. This is simply not viable.

### Our Solution Approach

We built a comprehensive master catalog of standardized grocery products. Instead of manual data entry, sellers can:

1. **Search** for products in our master catalog (e.g., "tata salt")
2. **Select** the matching product from search results
3. **Add their selling price** and stock quantity
4. **Go live** for buyers immediately

This reduces onboarding from days to hours. But this approach created a critical technical requirement: **the search must be extremely forgiving and fast**.

### Why Search is Mission-Critical

Because our sellers are not tech-savvy:

**Problem 1: Spelling Mistakes are Common**
- Seller types "colget" instead of "Colgate"
- Seller types "basmti rice" instead of "basmati rice"
- Seller types "amool butter" instead of "Amul butter"

If search fails on typos, sellers get stuck and abandon onboarding.

**Problem 2: Zero Tolerance for Latency**
- Sellers expect instant results (like Google)
- A slow search (3-5 seconds) feels broken to them
- They're often on slow 2G/3G mobile connections

**Problem 3: Partial Information**
- Sellers might only remember the brand ("Tata") or category ("rice")
- They need autocomplete-like behavior for partial matches

### Technical Requirements

This user experience translated into demanding technical requirements:

**Functional Requirements:**
- **Extreme typo tolerance**: Handle 1-2 character substitutions, insertions, deletions
- **Flexible matching**: Exact matches, partial matches, fuzzy matches, word-level matches
- **Multi-field search**: Search across product names, brands, manufacturer names, ingredients
- **Advanced filtering**: Filter by brands, categories, country of origin
- **Dual search modes**: Search products and variants separately
- **Smart pagination**: Efficient navigation across millions of records

**Non-Functional Requirements:**
- **Sub-second latency**: P95 response times under 200ms (feels instant on mobile)
- **High throughput**: Support 1000+ concurrent searches during onboarding campaigns
- **Strong consistency**: Newly added products must be searchable immediately
- **Cost efficiency**: Minimize infrastructure costs while scaling to thousands of sellers
- **Operational simplicity**: Small team, can't manage complex distributed systems

### The Scale

- **Products**: 300K+ unique grocery items across all categories
- **Variants**: 500K+ product variations (different sizes, flavors, packaging)
- **Search Volume**: 1K+ searches per minute during peak onboarding hours
- **User Expectation**: Google-like search experience despite technical limitations

---

## <a name="the-elasticsearch-temptation"></a>The Elasticsearch Temptation

The conventional wisdom was clear: "Use Elasticsearch for search." The arguments were compelling:

### Why Elasticsearch Made Sense

**1. Purpose-Built for Search**
- Optimized inverted indices for full-text search
- Sophisticated relevance scoring algorithms (BM25, TF-IDF)
- Built-in support for fuzzy matching and typo tolerance

**2. Proven at Scale**
- Battle-tested by companies like Uber, Netflix, GitHub
- Horizontal scaling through sharding
- Distributed architecture for high availability

**3. Rich Feature Set**
- Aggregations and analytics out of the box
- Highlighting and suggestions
- Extensive language analysis support

### Why We Didn't Choose It

Despite these advantages, we identified several concerns:

**1. Operational Complexity**
```
Existing Stack:          With Elasticsearch:
┌──────────────┐         ┌──────────────┐
│  Spring Boot │         │  Spring Boot │
│  Application │         │  Application │
└──────┬───────┘         └──────┬───────┘
       │                        │
       │                        ├─────────┐
       ▼                        ▼         ▼
┌──────────────┐         ┌──────────┐ ┌──────────────┐
│  PostgreSQL  │         │PostgreSQL│ │Elasticsearch │
└──────────────┘         └──────────┘ └──────────────┘

Simple to maintain       Additional:
                         - Sync mechanism
                         - Index management
                         - Cluster monitoring
                         - Data consistency issues
```

**2. Data Synchronization Overhead**
- Need to maintain consistency between PostgreSQL (source of truth) and Elasticsearch
- Complex change data capture (CDC) pipeline
- Handling sync failures and recovery
- Eventual consistency issues in critical business flows

**3. Infrastructure and Cost Constraints**

This was the most decisive factor for us. As a bootstrapped startup, we were running our entire stack on three minimal cloud instances:
- **Frontend**: 1GB RAM instance
- **Backend**: 1GB RAM instance
- **PostgreSQL**: 1GB RAM instance

Adding Elasticsearch would mean:
- **Another server**: Minimum 2GB RAM for a single ES node (4GB+ recommended for production)
- **Increased complexity**: Managing another service with our small team
- **Higher costs**: 33%+ increase in infrastructure spend for a feature we weren't sure we needed

The question became: **"Can we achieve acceptable search performance with what we already have?"** If yes, we'd save significant costs and operational overhead. If no, we could always migrate to Elasticsearch later.

**4. The PostgreSQL Opportunity**
PostgreSQL 12+ introduced powerful search extensions:
- `pg_trgm`: Trigram-based fuzzy matching
- `fuzzystrmatch`: Levenshtein distance calculations
- Full-text search with tsvector and configurable weighting
- Generated columns for automatic index maintenance
- GIN indices for high-performance text search

This led us to ask: **"Can we achieve our requirements with PostgreSQL alone?"**

---

## <a name="our-solution"></a>Our Solution: PostgreSQL-Powered Multi-Strategy Search

We designed a multi-layered search system that combines PostgreSQL's advanced features with reactive programming for optimal performance.

### Core Architecture Principles

**1. Multi-Strategy Search with Intelligent Fallback**

Instead of a one-size-fits-all search approach, we implemented six distinct strategies executed in order of specificity. Each strategy targets different search scenarios:

| Strategy | Use Case | Example Query | SQL Technique |
|----------|----------|---------------|---------------|
| **EXACT_FULLTEXT** | User knows exact terms | "organic rice basmati" | `plainto_tsquery` with ts_rank |
| **PREFIX_MATCH** | Autocomplete, partial words | "organ ric" | `to_tsquery` with prefix operators |
| **SIMPLE_TEXT** | Substring matching | "ric" anywhere in text | `ILIKE '%ric%'` |
| **TRIGRAM_SIMILARITY** | Typos, misspellings | "orgnic ryce" | `similarity()` with threshold |
| **LEVENSHTEIN_NAME** | Name-specific fuzzy match | "oranic rice" | `levenshtein()` with distance limit |
| **WORD_FUZZY** | Word-level fuzzy matching | "organik" | Word-by-word Levenshtein |

**2. Reactive Architecture for Performance**

We built the search service using Spring WebFlux with R2DBC for non-blocking database access:

```kotlin
@Service
class SearchService(
    private val searchRepository: SearchRepository
) {
    suspend fun searchProducts(request: SearchRequest): SearchResult =
        withTimeout(60.seconds) {
            search(request, SearchMode.PRODUCTS)
        }
}
```

Key architectural components:
- **Spring WebFlux**: Non-blocking HTTP layer
- **R2DBC**: Reactive database connectivity
- **Kotlin Coroutines**: Structured concurrency for readable async code
- **Flow API**: Backpressure-aware streaming

**3. Unified Search Interface**

A single repository interface handles both products and variants through dynamic SQL generation:

```kotlin
interface SearchRepository {
    fun search(
        strategy: SearchStrategy,
        searchTerm: String,
        searchMode: SearchMode,  // PRODUCTS or VARIANTS
        offset: Int,
        limit: Int,
        excludePids: List<Long>,
        brands: List<String>? = null,
        categories: List<String>? = null,
        countries: List<String>? = null,
        minSimilarity: Float = 0.3f,
        maxDistance: Int = 2
    ): Flow<UnifiedSearchProjection>
}
```

This abstraction allows us to search across different tables (products vs. variants with joins) using the same search strategies.

**4. Database Schema Optimization**

Our PostgreSQL schema includes auto-generated search optimization columns:

```sql
CREATE TABLE product(
    pid SERIAL NOT NULL,
    name VARCHAR(255) NOT NULL,
    brand VARCHAR(50) NOT NULL,
    description TEXT NOT NULL,
    manufacturer VARCHAR(255),

    -- Auto-generated weighted full-text search vector
    search_vector tsvector GENERATED ALWAYS AS (
        setweight(to_tsvector('english', coalesce(name, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(brand, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(description, '')), 'B') ||
        setweight(to_tsvector('english', coalesce(manufacturer, '')), 'B')
    ) STORED,

    -- Auto-generated concatenated text for trigram/fuzzy matching
    search_text TEXT GENERATED ALWAYS AS (
        lower(trim(
            coalesce(name, '') || ' ' ||
            coalesce(brand, '') || ' ' ||
            coalesce(manufacturer, '')
        ))
    ) STORED,

    PRIMARY KEY (pid)
);

-- Specialized indices for different search strategies
CREATE INDEX idx_product_search
    ON product USING GIN (search_vector);

CREATE INDEX idx_product_search_text_trgm
    ON product USING GIN (search_text gin_trgm_ops);

CREATE INDEX idx_product_search_text_btree
    ON product USING BTREE (search_text);
```

The **GENERATED ALWAYS AS** columns are automatically maintained by PostgreSQL, eliminating manual trigger logic and ensuring indices are always up-to-date.

---

## <a name="architecture"></a>Deep Dive: Architecture & Implementation

### Execution Flow

Here's how a search request flows through the system:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Request Entry Point (ProductController)             │
│     GET /product/search?keyword=organic+rice                    │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. Cache Check (Caffeine In-Memory Cache)                      │
│     Key: "search:organic rice:limit:20"                         │
│     Hit Rate: ~65% (10-minute TTL)                              │
└───────────────────────────┬─────────────────────────────────────┘
                            │ Cache Miss
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. Service Layer (SearchService)                               │
│     Decodes pagination context from pageToken                   │
│     Manages strategy execution and result aggregation           │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  4. Sequential Strategy Execution                               │
│                                                                 │
│  ┌─────────────────────────────────────────────────┐            │
│  │ Strategy 1: EXACT_FULLTEXT                      │            │
│  │ SELECT *, ts_rank(search_vector, query) as score│            │
│  │ WHERE search_vector @@ plainto_tsquery(...)     │            │
│  │ Results: 15 products                            │            │
│  └─────────────────────────────────────────────────┘            │
│                         │                                       │
│                         ▼ (need 5 more)                         │
│  ┌─────────────────────────────────────────────────┐            │
│  │ Strategy 2: PREFIX_MATCH                        │            │
│  │ WHERE search_vector @@ to_tsquery('organ:*')    │            │
│  │ Results: 5 products (deduplicated)              │            │
│  └─────────────────────────────────────────────────┘            │
│                         │                                       │
│                         ▼ (limit reached, stop)                 │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  5. Result Aggregation & Response                               │
│     - Deduplicate by PID                                        │
│     - Encode pagination token for next page                     │
│     - Cache results (first page only)                           │
│     - Track execution time                                      │
└─────────────────────────────────────────────────────────────────┘
```

### Strategy Execution Details

Let's examine each strategy's SQL implementation:

#### Strategy 1: EXACT_FULLTEXT
**Best for**: Exact term matching with relevance ranking

```sql
SELECT
    p.pid, p.name,
    ts_rank(p.search_vector, plainto_tsquery('english', :searchTerm)) as score
FROM product p
WHERE p.search_vector @@ plainto_tsquery('english', :searchTerm)
ORDER BY score DESC
LIMIT 20;
```

**Performance**: ~30-50ms for typical queries (uses GIN index)

#### Strategy 2: PREFIX_MATCH
**Best for**: Autocomplete and partial word matching

```kotlin
// Input: "organ ric" → Transform to: "organ:* & ric:*"
val processedSearchTerm = searchTerm.trim()
    .split(' ')
    .filter { it.isNotBlank() }
    .joinToString(" & ") { "$it:*" }
```

```sql
SELECT
    p.pid, p.name,
    COALESCE(ts_rank(p.search_vector, to_tsquery('english', :searchTerm)), 0.0) as score
FROM product p
WHERE p.search_vector @@ to_tsquery('english', :searchTerm)
ORDER BY score DESC
LIMIT 20;
```

**Performance**: ~40-70ms (slightly slower due to prefix operator)

#### Strategy 3: SIMPLE_TEXT
**Best for**: Substring matching when FTS fails

```sql
SELECT
    p.pid, p.name,
    1.0 as score
FROM product p
WHERE p.search_text ILIKE :searchTerm  -- '%organic rice%'
ORDER BY p.pid
LIMIT 20;
```

**Performance**: ~80-120ms (uses B-tree index for prefix, full scan for infix)

#### Strategy 4: TRIGRAM_SIMILARITY
**Best for**: Fuzzy matching with typos

```sql
SELECT
    p.pid, p.name,
    similarity(p.search_text, :searchTerm) as score
FROM product p
WHERE p.search_text % :searchTerm  -- Trigram similarity operator
  AND similarity(p.search_text, :searchTerm) >= 0.3
ORDER BY score DESC
LIMIT 20;
```

**Performance**: ~100-150ms (GIN trigram index helps significantly)

**Example**: "orgnic ryce" matches "organic rice" with ~0.7 similarity

#### Strategy 5: LEVENSHTEIN_NAME
**Best for**: Name-specific fuzzy matching

```sql
SELECT
    p.pid, p.name,
    (1.0 - (levenshtein(LOWER(:searchTerm), LOWER(p.name))::float /
            GREATEST(LENGTH(:searchTerm), LENGTH(p.name), 1))) as score
FROM product p
WHERE levenshtein(LOWER(:searchTerm), LOWER(p.name)) <= 2
  AND LENGTH(p.name) > 0
ORDER BY score DESC
LIMIT 20;
```

**Performance**: ~120-200ms (no index support, requires table scan with filter)

**Example**: "oranic rice" (distance=1) matches "organic rice"

#### Strategy 6: WORD_FUZZY
**Best for**: Maximum recall when all else fails

```sql
SELECT DISTINCT
    p.pid, p.name,
    (1.0 - (MIN(levenshtein(LOWER(:searchTerm), word))::float /
            LENGTH(:searchTerm))) as score
FROM product p,
     unnest(string_to_array(p.search_text, ' ')) as word
WHERE levenshtein(LOWER(:searchTerm), word) <= 2
  AND LENGTH(word) > 2
GROUP BY p.pid, p.name
ORDER BY score DESC
LIMIT 20;
```

**Performance**: ~150-250ms (most expensive strategy, used as last resort)

**Example**: "basmti" matches products containing the word "basmati"

### Intelligent Pagination with Stateless Tokens

We implemented cursor-based pagination that encodes the search context:

```kotlin
data class SearchContext(
    val currentStrategyIndex: Int,        // Which strategy we're on
    val currentStrategyOffset: Int,       // Offset within that strategy
    val excludedPids: MutableSet<Int>     // PIDs already returned
)

// Encode context into a compact string token
fun encodeSearchContext(context: SearchContext): String {
    return "${context.currentStrategyIndex}:${context.currentStrategyOffset}:" +
           "${context.excludedPids.joinToString(",")}"
}

// Example token: "2:40:12345,12346,12347,..."
// Means: Currently on strategy #2, offset 40, exclude these PIDs
```

This approach allows:
- **Stateless pagination**: No server-side session storage
- **Consistent results**: Same query always returns same ordering
- **Deduplication**: Prevents duplicate results across strategy boundaries
- **Strategy continuation**: Resume from where previous page left off

### Caching Strategy

We use Caffeine for in-memory caching with specific policies:

```kotlin
private val searchCache: Cache<String, SearchResult> = Caffeine.newBuilder()
    .maximumSize(500)                    // Limit memory footprint
    .expireAfterWrite(Duration.ofMinutes(10))  // Data freshness
    .removalListener { key, _, cause ->
        logger.debug("Cache entry evicted: key={}, cause={}", key, cause)
    }
    .recordStats()                       // Monitor hit rate
    .build()
```

**Caching Decisions**:
- Only cache **first page** of results (most common query)
- Include filters in cache key for correctness
- 10-minute TTL balances freshness with hit rate
- 500 entry limit (~50MB memory for typical result sizes)

**Cache Key Structure**:
```kotlin
fun cacheKey(): String {
    val brandStr = brands?.joinToString("|") ?: ""
    val categoryStr = categories?.joinToString("|") ?: ""
    val countryStr = countries?.joinToString("|") ?: ""
    return "search:$searchTerm:$brandStr:$categoryStr:$countryStr:$limit"
}

// Example: "search:organic rice:Organic India|Tata::India:20"
```

### Dual Search Modes: Products vs. Variants

Our unified interface supports two distinct search modes:

```kotlin
enum class SearchMode {
    PRODUCTS,         // Search product table only
    VARIANTS          // Search variants joined with product
}
```

**PRODUCTS mode**: Search the product table
```sql
SELECT
    p.pid, NULL as parent_id, p.name
FROM product p
WHERE p.search_vector @@ plainto_tsquery('english', :searchTerm)
```

**VARIANTS mode**: Search variants with product data
```sql
SELECT
    v.pid, v.parent_id, m.name
FROM product_variant v
INNER JOIN product m ON v.parent_id = m.pid
WHERE m.search_vector @@ plainto_tsquery('english', :searchTerm)
```

This flexibility allows:
- **Product discovery**: Users find base products first
- **Variant selection**: Users drill down to specific sizes/variants
- **Single codebase**: Same search logic, different projections

---

## <a name="benefits"></a>What We Gained

### 1. Architectural Simplicity

**Single Source of Truth**: PostgreSQL is both our transactional database and search engine. No synchronization concerns.

```
Before (with ES):              After (PostgreSQL only):
┌──────────────┐               ┌──────────────┐
│  Application │               │  Application │
└──────┬───────┘               └──────┬───────┘
       │                              │
       ├─────────┬──────────          │
       ▼         ▼         │           ▼
┌──────────┐ ┌──────────┐ │    ┌──────────────┐
│PostgreSQL│ │Elastics- │ │    │  PostgreSQL  │
│  (OLTP)  │ │  earch   │ │    │ (OLTP+Search)│
└────┬─────┘ └─────▲────┘ │    └──────────────┘
     │             │      │
     │  ┌──────────┴────┐ │    Complexity: O(1)
     └─►│  CDC/Sync     │◄┘
        │  Pipeline     │
        └───────────────┘

Complexity: O(n²)
```

### 2. Immediate Consistency

Updates to products are immediately searchable:

```kotlin
@Transactional
fun updateProduct(productId: Long, updates: ProductUpdate) {
    // Update the product
    productRepository.save(product.apply {
        productName = updates.name
        brand = updates.brand
        // search_vector automatically updated via GENERATED column
    })

    // No need to:
    // - Send message to queue
    // - Update Elasticsearch index
    // - Handle sync failures
    // - Deal with eventual consistency

    // Searches immediately reflect changes!
}
```

### 3. Cost Efficiency

**Monthly Infrastructure Costs (Production)**:

| Component | PostgreSQL Approach  | Elasticsearch Approach     |
|-----------|----------------------|----------------------------|
| Database Server | $10 (RDS PostgreSQL) | $10 (RDS PostgreSQL)       |
| Search Service | -                    | $30 (Elasticsearch 3-node) |
| Data Sync | -                    | $10 (Kafka/Debezium)       |
| **Total** | **$10**              | **$50**                    |

**Savings**: $40/month or **80% reduction** in infrastructure costs.

### 4. Simplified Operations

**Developer Experience**:
- Single connection pool to manage
- One query language (SQL) for all data access
- Familiar PostgreSQL tooling and debugging
- No index management or shard rebalancing
- Standard database backup/restore procedures

**On-call Burden**:
- Fewer services to monitor
- Simpler incident response (single system to debug)
- No distributed system complexities (split-brain, quorum loss)

### 6. Reactive Performance

Kotlin Coroutines + R2DBC provide non-blocking I/O with readable code:

```kotlin
suspend fun searchProducts(request: SearchRequest): SearchResult {
    // Non-blocking, but reads like synchronous code
    val results = searchRepository.search(...)
        .toList()  // Collect Flow

    val totalCount = if (request.includeCount) {
        searchRepository.searchCount(...)  // Parallel query possible
    } else null

    return SearchResult(results, totalCount, ...)
}
```

Traditional blocking JDBC would require thread-per-request model, limiting concurrency.

---

## <a name="trade-offs"></a>What We Sacrificed

Honesty compels us to acknowledge the limitations of our approach.

### 1. Horizontal Scalability Ceiling

**Limitation**: PostgreSQL vertical scaling has limits. Beyond a certain point, you need read replicas.

### 2. Advanced Relevance Tuning

**Limitation**: Elasticsearch's relevance algorithms (BM25, learning-to-rank) are more sophisticated.

### 3. Real-Time Analytics

**Limitation**: Elasticsearch aggregations are faster for analytics queries.

**Example**: "Top 10 brands searched in last 24 hours with trend analysis"

### 4. Geospatial Search

**Limitation**: Elasticsearch geo queries are highly optimized.

**Our Current Situation**: We don't need location-based product search (yet).

**If we need it**: PostgreSQL PostGIS extension provides geospatial capabilities, though not as performant as Elasticsearch at extreme scale.

### 5. Distributed Full-Text Features

**What Elasticsearch provides**:
- Multi-language analysis with extensive tokenizers
- Phonetic matching (sounds-like searches)
- Advanced highlighting with context
- Did-you-mean suggestions

**What we lose**:
- Phonetic search ("Colgate" → "Kolgate")
- Rich highlighting (we have basic highlighting via SQL)
- Multi-language stemming (we support English only)

---

## <a name="lessons"></a>Lessons Learned

### 1. PostgreSQL is Seriously Underrated for Search

**Industry Perception**: "PostgreSQL is a relational database, not a search engine."

**Reality**: With pg_trgm, fuzzystrmatch, and full-text search, PostgreSQL rivals dedicated search engines for many use cases.

**Key Enabler**: Generated columns and GIN indices make it production-ready without manual triggers.

### 2. Multi-Strategy Search > Single Algorithm

Our six-strategy approach outperforms any single search algorithm because:
- Different queries have different characteristics
- Users don't know which strategy will work for their input
- Graceful degradation ensures we always return something useful

**Data Point**: 12% of successful searches required fallback strategies (strategies 3-6). Without them, those users would have seen zero results.

### 3. Reactive Programming is Worth the Learning Curve

**Before (Blocking JDBC)**:
- Thread-per-request model
- 200 concurrent requests = 200 threads
- High memory consumption (1MB+ per thread stack)
- Context switching overhead

**After (R2DBC + Coroutines)**:
- Event-loop model
- 1,000 concurrent requests = ~10-20 threads
- Low memory footprint
- Better CPU utilization

**Result**: 5x improvement in throughput without hardware changes.

### 4. Caching Layer is Critical

Cache hit rate of 60% reduces database load by nearly **2/3**. Without caching:
- Database CPU would spike to 95%+
- Latencies would degrade under load
- Need more expensive database instance

**Investment**: 2 days to implement Caffeine caching
**Return**: $20/month savings on database costs + better performance

---

## <a name="recommendations"></a>When Should You Choose This Approach?

### Use PostgreSQL-Based Search When:

✅ **Data Volume**: < 50M documents (single instance) or < 500M (with read replicas)

✅ **Latency SLA**: P95 latency 100-500ms is acceptable

✅ **Query Patterns**: Primarily keyword search with filters, not complex relevance tuning

✅ **Consistency Requirements**: Strong consistency is valuable (transactional workflows)

✅ **Team Expertise**: Team knows PostgreSQL deeply but lacks Elasticsearch experience

✅ **Operational Constraints**: Limited ops resources, prefer simplicity over absolute performance

✅ **Cost Sensitivity**: Infrastructure budget is constrained

✅ **Use Case**: Product catalogs, document search, user directories, content management

### Use Elasticsearch When:

❌ **Massive Scale**: > 500M documents or need to shard across data centers

❌ **Ultra-Low Latency**: Need P95 latency < 50ms

❌ **Advanced Features**: Need learning-to-rank, advanced analytics, ML-powered search

❌ **Multi-Tenant**: Each tenant needs isolated search indices

❌ **Log Analytics**: Time-series data with rollover and retention policies

❌ **Complex Relevance**: Need sophisticated relevance tuning with A/B testing

❌ **Polyglot Persistence**: Already have multiple specialized data stores

### The Hybrid Approach

Some teams use **both**:
- PostgreSQL for transactional queries and simple searches
- Elasticsearch for complex full-text search and analytics

**When this makes sense**:
- You have distinct use cases (OLTP vs. analytics)
- You have ops expertise to manage both
- The cost is justified by business value

---

## Conclusion

Building Cerve's product search with PostgreSQL was a calculated risk that paid off. We achieved:

- **Sub-second search latency** (P95: 187ms)
- **80% cost reduction** vs. Elasticsearch approach
- **Simplified architecture** with single data store
- **Immediate consistency** for better UX

The key insight: **PostgreSQL's advanced search features + reactive architecture + intelligent multi-strategy search = production-ready search without dedicated search infrastructure**.

This doesn't mean Elasticsearch is wrong—it means the right choice depends on your specific constraints. For Cerve, PostgreSQL was the optimal solution.

### What's Next?

We're exploring:
1. **Query performance optimizations**: TimeScaleDB's `pg_textsearch` for ElasitcSearch -like performance

---

**About the Author**: This post was written by the Cerve Engineering Team, with contributions from backend engineers working on the ONDC marketplace platform.

**Want to discuss this approach?** We'd love to hear your experiences with PostgreSQL vs. Elasticsearch trade-offs. Reach out to us at engineering@cerve.in.

---

*Published on Cerve Engineering Blog | December 22, 2025*
