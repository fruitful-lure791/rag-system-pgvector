# First Issues for RAG System with pgvector

This document lists the first 5 issues/tasks for contributors to get started with.

## 🌟 Good First Issues

### Issue #1: Implement Automatic Re-indexing
**Difficulty:** Medium
**Labels:** `enhancement`, `automation`, `good-first-issue`

**Description:**
Implement an automatic re-indexing system that updates embeddings when documents are modified or when embedding models change.

**Requirements:**
- Detect changes in documents (content updates)
- Queue re-embedding tasks
- Batch process re-indexing jobs
- Handle model changes (re-index all with new model)
- Add progress tracking
- Support cancellation
- Idempotent operations

**Files to Create:**
- `src/services/re-indexer.ts`
- `src/queues/re-index-queue.ts`
- `tests/services/re-indexer.test.ts`

**Files to Modify:**
- `src/services/document-store.ts` - Add hooks
- `src/index.ts` - Export re-indexer
- `README.md` - Add documentation

**Example Usage:**
```typescript
import { ReIndexer } from './services/re-indexer';

const reIndexer = new ReIndexer({
  rag: ragSystem,
  batchSize: 100,
  concurrency: 5,
});

// Re-index changed documents
await reIndexer.reIndexChanged();

// Re-index with new model
await reIndexer.reIndexAll({
  newModel: 'text-embedding-3-large',
  dimensions: 3072,
});

// Check progress
const progress = await reIndexer.getProgress();
console.log(`${progress.processed}/${progress.total} documents`);
```

**Acceptance Criteria:**
- [ ] ReIndexer service implemented
- [ ] Document change detection working
- [ ] Batch processing efficient
- [ ] Progress tracking accurate
- [ ] Tests with >80% coverage
- [ ] Documentation with examples
- [ ] Error handling robust

---

### Issue #2: Add Multi-Query Retrieval
**Difficulty:** Medium
**Labels:** `enhancement`, `retrieval`, `good-first-issue`

**Description:**
Implement multi-query retrieval (also called query expansion) to improve retrieval by generating multiple queries from the user's question.

**Requirements:**
- Generate 3-5 related queries from original
- Use LLM to generate variations
- Execute searches in parallel
- Deduplicate results
- Re-rank combined results
- Configurable number of queries
- Track query expansion quality

**Files to Create:**
- `src/services/multi-query-retriever.ts`
- `tests/services/multi-query-retriever.test.ts`
- `docs/multi-query-guide.md`

**Files to Modify:**
- `src/rag-system.ts` - Integrate multi-query
- `README.md` - Add documentation

**Example Usage:**
```typescript
const response = await rag.query({
  question: 'How does TypeScript improve code quality?',
  retrieval: {
    strategy: 'multi-query',
    numQueries: 5,
    llmModel: 'gpt-4',
  },
  topK: 5,
});
```

**Algorithm:**
```
1. Original Query: "How does TypeScript improve code quality?"
2. Generate Variations:
   - "TypeScript benefits for code quality"
   - "Why use TypeScript for better code"
   - "TypeScript type safety advantages"
   - "Code quality improvements with TypeScript"
3. Search in parallel with all queries
4. Merge and deduplicate results
5. Re-rank by relevance to original query
6. Return top-K results
```

**Acceptance Criteria:**
- [ ] Multi-query retriever implemented
- [ ] Query generation with LLM working
- [ ] Parallel search efficient
- [ ] Deduplication effective
- [ ] Integration with RAG system
- [ ] Comprehensive tests
- [ ] Performance benchmarks
- [ ] Documentation with examples

---

### Issue #3: Build Performance Benchmarking Suite
**Difficulty:** Medium
**Labels:** `tooling`, `performance`, `testing`

**Description:**
Create a comprehensive benchmarking suite to measure and track performance across different scenarios and dataset sizes.

**Requirements:**
Create benchmarks for:

1. **Ingestion Performance**
   - Document insertion rate
   - Embedding generation speed
   - Batch vs single operations

2. **Search Performance**
   - Query latency (p50, p95, p99)
   - Throughput (queries/second)
   - Impact of database size
   - Different topK values

3. **Generation Performance**
   - LLM generation time
   - Token usage statistics
   - Streaming vs non-streaming

4. **Scaling Tests**
   - Different database sizes (1K, 10K, 100K, 1M docs)
   - Concurrent queries (1, 10, 100, 1000)
   - Document length variations

**Files to Create:**
- `tests/benchmarks/ingestion.bench.ts`
- `tests/benchmarks/search.bench.ts`
- `tests/benchmarks/generation.bench.ts`
- `tests/benchmarks/scaling.bench.ts`
- `tests/benchmarks/reporter.ts`
- `scripts/run-benchmarks.ts`

**Dependencies:**
```bash
npm install --save-dev tinybench
```

**Example Output:**
```
=== RAG System Benchmarks ===

Ingestion Performance:
- Single doc: 12ms
- Batch (10): 98ms (9.8ms/doc)
- Batch (100): 890ms (8.9ms/doc)
- Batch (1000): 8.2s (8.2ms/doc)

Search Performance (10K docs):
- TopK=5: 45ms
- TopK=10: 52ms
- TopK=20: 68ms

Throughput:
- Single thread: 22 queries/sec
- 10 concurrent: 180 queries/sec
- 100 concurrent: 1200 queries/sec

Scaling (Search Latency):
- 1K docs: 12ms
- 10K docs: 45ms
- 100K docs: 180ms
- 1M docs: 1.2s
```

**Acceptance Criteria:**
- [ ] Benchmark framework created
- [ ] 4+ benchmark categories
- [ ] Automated reporting
- [ ] CI integration
- [ ] Historical tracking
- [ ] Documentation on running benchmarks
- [ ] Performance regression detection

---

### Issue #4: Implement Document Summarization
**Difficulty:** Easy
**Labels:** `enhancement`, `feature`, `good-first-issue`

**Description:**
Add document summarization capabilities to automatically generate summaries for long documents before ingestion.

**Requirements:**
- Generate summaries for long documents (>2000 tokens)
- Store summaries alongside documents
- Use summaries for better context
- Support different summary lengths
- Configurable summarization model
- Cache summaries to avoid re-generation
- Handle summary updates

**Files to Create:**
- `src/services/summarizer.ts`
- `tests/services/summarizer.test.ts`

**Files to Modify:**
- `src/services/document-store.ts` - Integrate summarization
- `src/migrations/` - Add summary column
- `README.md` - Document usage

**Example Usage:**
```typescript
import { DocumentSummarizer } from './services/summarizer';

const summarizer = new DocumentSummarizer({
  llmModel: 'gpt-4',
  maxLength: 200, // words
  style: 'executive', // or 'technical', 'brief'
});

// Summarize during ingestion
await store.addDocument({
  content: longDocument,
  summarize: true,
  summaryOptions: {
    maxLength: 200,
    style: 'technical',
  },
});

// Search with summaries
const results = await store.search({
  query: 'machine learning',
  useSummary: true, // Search summaries first
  topK: 5,
});
```

**Acceptance Criteria:**
- [ ] Summarizer service implemented
- [ ] Integration with document store
- [ ] Multiple summary styles supported
- [ ] Summary search optimization
- [ ] Tests with >80% coverage
- [ ] Documentation with examples
- [ ] Performance benchmarks

---

### Issue #5: Create Query Understanding and Routing
**Difficulty:** Medium
**Labels:** `enhancement`, `routing`, `intelligence`

**Description:**
Implement query understanding to automatically detect query types and route to appropriate search strategies.

**Requirements:**
Detect query types:
- **Factual**: "What is the capital of France?" → Keyword search
- **Conceptual**: "How does machine learning work?" → Semantic search
- **Comparative**: "TypeScript vs JavaScript" → Hybrid search
- **Temporal**: "Recent developments in AI" → Time-weighted search
- **Code-related**: "How to use React hooks" → Code-specific search

Route to strategies:
- Exact match for names, IDs
- Semantic for concepts
- Hybrid for comparisons
- Recent-first for temporal
- Code-aware for programming queries

**Files to Create:**
- `src/services/query-router.ts`
- `src/services/query-classifier.ts`
- `tests/services/query-router.test.ts`

**Files to Modify:**
- `src/rag-system.ts` - Integrate router
- `README.md` - Document routing

**Example Usage:**
```typescript
const response = await rag.query({
  question: 'Compare TypeScript and JavaScript',
  routing: {
    enabled: true,
    strategy: 'auto', // or specify strategy
  },
});

// Router automatically:
// 1. Classifies as "comparative"
// 2. Routes to hybrid search
// 3. Uses comparative prompt template
// 4. Formats response as comparison table
```

**Implementation:**
```typescript
interface QueryAnalysis {
  type: 'factual' | 'conceptual' | 'comparative' | 'temporal' | 'code';
  confidence: number;
  entities: string[];
  keywords: string[];
  timeRange?: { start: Date; end: Date };
  recommendedStrategy: 'keyword' | 'semantic' | 'hybrid';
}

class QueryRouter {
  async analyze(query: string): Promise<QueryAnalysis> {
    // Use LLM to classify query
    // Extract entities and keywords
    // Detect time expressions
    // Recommend search strategy
  }

  async route(query: string, analysis: QueryAnalysis): Promise<SearchResults> {
    // Execute recommended strategy
    // Apply type-specific optimizations
    // Return results
  }
}
```

**Acceptance Criteria:**
- [ ] Query classifier implemented
- [ ] Query router working
- [ ] 5+ query types supported
- [ ] Strategy routing accurate (>85%)
- [ ] Integration with RAG system
- [ ] Comprehensive tests
- [ ] Documentation with examples
- [ ] Performance benchmarks

## 📋 How to Contribute

1. **Choose an issue** from the list above
2. **Comment on the issue** to claim it
3. **Read CONTRIBUTING.md** for setup
4. **Create a branch**: `git checkout -b issue-X-description`
5. **Implement with tests**
6. **Submit PR** with clear description

## 💡 Tips for First-Time Contributors

- Start with "easy" or "good-first-issue" labeled items
- Ask questions in the issue
- Focus on code quality and tests
- Consider performance implications
- Add comprehensive documentation
- Update CHANGELOG.md
- Follow database best practices

## 🎓 Learning Resources

- [pgvector GitHub](https://github.com/pgvector/pgvector)
- [Vector Search Guide](https://supabase.com/docs/guides/ai/vector-columns)
- [RAG Best Practices](./docs/best-practices.md)
- [PostgreSQL Performance](https://wiki.postgresql.org/wiki/Performance_Optimization)

## 🤝 Need Help?

- Open a [discussion](https://github.com/groovy-web/rag-system-pgvector/discussions)
- Visit [Groovy Web](https://www.groovyweb.co)
- Check [pgvector discussions](https://github.com/pgvector/pgvector/discussions)

---

Happy contributing! 🎉
