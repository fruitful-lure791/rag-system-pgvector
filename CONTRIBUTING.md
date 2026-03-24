# Contributing to RAG System with pgvector

Thank you for your interest in contributing! This is a production system used by many applications, so quality and reliability are critical.

## 🚀 Getting Started

### Prerequisites

- PostgreSQL 15+ with pgvector extension
- Node.js 18+
- Docker (for local development)
- OpenAI API key (or alternatives)

### Development Setup

1. Fork and clone the repository
2. Install dependencies:

```bash
npm install
```

3. Start PostgreSQL with pgvector:

```bash
docker-compose up -d
```

4. Run migrations:

```bash
npm run db:migrate
npm run db:seed
```

5. Create a feature branch:

```bash
git checkout -b feature/your-feature-name
```

## 🏗️ Architecture Overview

### System Components

```
┌─────────────────────────────────────────┐
│         Application Layer               │
│  (RAGSystem, DocumentStore, etc.)       │
└───────────────┬─────────────────────────┘
                │
┌───────────────▼─────────────────────────┐
│         Service Layer                   │
│  (Embedding, Search, Generation)        │
└───────────────┬─────────────────────────┘
                │
┌───────────────▼─────────────────────────┐
│         Data Layer                      │
│  (PostgreSQL + pgvector)                │
└─────────────────────────────────────────┘
```

### Key Modules

- **RAGSystem**: Main orchestration
- **DocumentStore**: CRUD operations
- **EmbeddingService**: Vector generation
- **SearchService**: Similarity search
- **GenerationService**: LLM integration
- **Cache**: Result caching
- **Monitoring**: Logging and metrics

## 📝 Contribution Guidelines

### Types of Contributions

We welcome:

- **Bug Fixes**: Issues with the system
- **New Features**: Additional capabilities
- **Performance**: Optimization and benchmarking
- **Documentation**: Guides and examples
- **Tests**: Increased coverage

### Code Style

- Use TypeScript for all code
- Follow functional patterns
- Add comprehensive JSDoc comments
- Handle errors gracefully
- Write tests for all code

```typescript
/**
 * Performs semantic search over documents
 * @param query - Search query text
 * @param options - Search options
 * @returns Promise with search results
 * @throws {DatabaseError} If query fails
 * @example
 * ```typescript
 * const results = await searchService.search({
 *   query: 'machine learning',
 *   topK: 5,
 *   filters: { type: 'paper' }
 * });
 * ```
 */
export async function search(
  query: string,
  options: SearchOptions
): Promise<SearchResult[]> {
  try {
    // Implementation
  } catch (error) {
    throw new DatabaseError(`Search failed: ${error.message}`);
  }
}
```

### Adding New Features

1. **Open an issue** to discuss
2. **Plan the implementation**
3. **Write tests first** (TDD)
4. **Implement with types**
5. **Add documentation**
6. **Update CHANGELOG**

### Adding Embedding Providers

Create in `src/embeddings/`:

```typescript
import { EmbeddingProvider } from '../types';

export class CustomEmbeddingProvider implements EmbeddingProvider {
  async embed(text: string): Promise<number[]> {
    // Implementation
  }

  async embedBatch(texts: string[]): Promise<number[][]> {
    // Batch implementation
  }

  getDimensions(): number {
    return 768; // Model dimensions
  }
}
```

Add to `src/embeddings/index.ts` and update README.

### Adding Chunking Strategies

Create in `src/chunkers/`:

```typescript
import { Chunker, Document } from '../types';

export class CustomChunker implements Chunker {
  constructor(private options: ChunkerOptions) {}

  async split(document: Document): Promise<Chunk[]> {
    // Implementation
  }
}
```

## 🧪 Testing

### Unit Tests

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { EmbeddingService } from '../src/embeddings';

describe('EmbeddingService', () => {
  let service: EmbeddingService;

  beforeEach(() => {
    service = new EmbeddingService({
      provider: 'openai',
      model: 'text-embedding-3-small',
    });
  });

  it('should generate embeddings for text', async () => {
    const embedding = await service.embed('test text');

    expect(embedding).toBeInstanceOf(Array);
    expect(embedding.length).toBe(1536);
    expect(embedding[0]).toBeGreaterThanOrEqual(-1);
    expect(embedding[0]).toBeLessThanOrEqual(1);
  });

  it('should handle batch embeddings', async () => {
    const texts = ['text1', 'text2', 'text3'];
    const embeddings = await service.embedBatch(texts);

    expect(embeddings).toHaveLength(3);
    embeddings.forEach(embedding => {
      expect(embedding.length).toBe(1536);
    });
  });

  it('should handle empty text', async () => {
    await expect(
      service.embed('')
    ).rejects.toThrow('Text cannot be empty');
  });
});
```

### Integration Tests

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { RAGSystem } from '../src';
import { TestDatabase } from './helpers';

describe('RAG Integration', () => {
  let db: TestDatabase;
  let rag: RAGSystem;

  beforeAll(async () => {
    db = new TestDatabase();
    await db.setup();

    rag = new RAGSystem({
      connectionString: db.getConnectionString(),
      embeddingModel: 'text-embedding-3-small',
      llmModel: 'gpt-4',
    });
  });

  afterAll(async () => {
    await db.teardown();
  });

  it('should perform full RAG pipeline', async () => {
    // Add document
    await rag.addDocument({
      content: 'TypeScript adds static typing to JavaScript',
      metadata: { topic: 'typescript' },
    });

    // Wait for indexing
    await new Promise(resolve => setTimeout(resolve, 1000));

    // Query
    const response = await rag.query({
      question: 'What does TypeScript add to JavaScript?',
      topK: 1,
    });

    expect(response.answer).toBeDefined();
    expect(response.sources).toHaveLength(1);
    expect(response.sources[0].metadata.topic).toBe('typescript');
  });
});
```

### Running Tests

```bash
# Unit tests
npm test

# Integration tests
npm run test:integration

# Coverage
npm run test:coverage

# Watch mode
npm run test:watch
```

## 📤 Submitting Changes

### Commit Messages

Use conventional commits:

```
feat(search): add hybrid search with keyword matching

- Implement keyword search using PostgreSQL full-text search
- Add semantic + keyword result fusion
- Support configurable weights
- Include tests and benchmarks

Closes #123

Performance: 40% faster for keyword-heavy queries
```

### Pull Request Process

1. **Ensure tests pass**: `npm test`
2. **Check types**: `npm run type-check`
3. **Format code**: `npm run format`
4. **Lint**: `npm run lint`
5. **Update documentation**
6. **Submit PR** with:

```markdown
## Description
Brief description of changes

## Type
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Performance improvement
- [ ] Documentation

## Changes
- List of changes

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] All tests passing
- [ ] Manual testing completed

## Performance Impact
- [ ] No performance change
- [ ] Improved performance (details)
- [ ] Performance regression (details)

## Breaking Changes
If applicable, describe breaking changes and migration path.

## Documentation
- [ ] README updated
- [ ] API docs updated
- [ ] Examples added
- [ ] Migration guide (if breaking)

## Checklist
- [ ] Code follows style guide
- [ ] Self-review completed
- [ ] No new warnings
- [ ] Backward compatible (or migration provided)
```

## 🔒 Security Issues

For security vulnerabilities:

1. **Do NOT open a public issue**
2. Email: security@groovy-web.dev
3. Include details and reproduction steps
4. We'll respond within 48 hours

## 🐛 Reporting Bugs

### Bug Report Template

```markdown
## Bug Description
Clear description of the issue

## Reproduction Steps
1. Setup: '...'
2. Query: '....'
3. Error: '....'

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- PostgreSQL version: [e.g. 15.2]
- pgvector version: [e.g. 0.5.0]
- Node version: [e.g. 18.17.0]
- Package version: [e.g. 1.2.3]

## Database Schema
\`\`\`sql
-- Relevant schema
\`\`\`

## Code Sample
\`\`\`typescript
// Code that reproduces the issue
\`\`\`

## Error Message
\`\`\`
Paste error and stack trace
\`\`\`

## Additional Context
Any other relevant information
```

## 💡 Contribution Ideas

Looking to contribute? Here are some ideas:

- Add more embedding providers (SentenceTransformers, HuggingFace)
- Implement advanced reranking (Cohere, custom models)
- Build admin dashboard UI
- Add multi-language support
- Implement document versioning
- Add real-time streaming updates
- Build caching strategies (Redis, in-memory)
- Add analytics dashboard
- Implement automatic re-indexing
- Build batch processing for large datasets
- Add GraphQL API
- Implement rate limiting
- Build observability tools
- Add A/B testing framework
- Implement federated search

## 📈 Performance Guidelines

### Benchmarking

When making performance changes:

1. **Measure before** and after
2. **Use realistic datasets**
3. **Test under load**
4. **Profile bottlenecks**
5. **Document results**

Example benchmark:

```typescript
import { benchmark } from '../tests/helpers/benchmark';

describe('Performance', () => {
  it('should handle 100 concurrent queries', async () => {
    const result = await benchmark(async () => {
      const queries = Array(100).fill(null).map((_, i) =>
        rag.query({
          question: `Test query ${i}`,
          topK: 5,
        })
      );

      await Promise.all(queries);
    });

    console.log({
      totalTime: result.totalTime,
      avgTime: result.avgTime,
      minTime: result.minTime,
      maxTime: result.maxTime,
      queriesPerSecond: 100 / result.totalTime,
    });

    expect(result.avgTime).toBeLessThan(1000); // < 1s avg
  });
});
```

## 📜 Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Focus on what is best for the community
- Show empathy towards other community members

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.

## 🆘 Getting Help

- Read existing documentation
- Search existing issues
- Join our Discord community
- Open a discussion

---

Thank you for contributing to RAG System with pgvector!
