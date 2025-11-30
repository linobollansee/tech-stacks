# RAG-Systeme - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in RAG](#einführung-in-rag)
2. [RAG-Architektur](#rag-architektur)
3. [Document Loading & Processing](#document-loading--processing)
4. [Embeddings & Vector Stores](#embeddings--vector-stores)
5. [Retrieval Strategien](#retrieval-strategien)
6. [Advanced Retrieval](#advanced-retrieval)
7. [Generation & Prompting](#generation--prompting)
8. [RAG Optimization](#rag-optimization)
9. [Evaluation & Metrics](#evaluation--metrics)
10. [Production RAG](#production-rag)
11. [Praktische Projekte](#praktische-projekte)

---

## Einführung in RAG

### Was ist RAG?

**Retrieval Augmented Generation** kombiniert Information Retrieval mit LLM-basierter Textgenerierung.

### Warum RAG?

**Aktualität** 📅
- LLMs haben Knowledge Cut-off
- RAG nutzt aktuelle Daten
- Dynamische Information

**Genauigkeit** 🎯
- Faktenbasierte Antworten
- Quellenangaben möglich
- Weniger Halluzinationen

**Spezialisierung** 🔧
- Firmen-spezifisches Wissen
- Private Dokumente
- Domain-Expertise

### RAG vs Fine-Tuning

```
RAG:
  ✓ Schnell implementiert
  ✓ Einfach zu aktualisieren
  ✓ Transparente Quellen
  ✗ Retrieval Overhead

Fine-Tuning:
  ✓ Schnellere Inferenz
  ✓ Angepasster Stil
  ✗ Teuer zu trainieren
  ✗ Schwer zu aktualisieren
```

---

## RAG-Architektur

### Basic RAG Pipeline

```
User Query
    ↓
Embedding (Query)
    ↓
Vector Search
    ↓
Retrieve Documents
    ↓
Combine with Query
    ↓
LLM Generation
    ↓
Response
```

### Implementation

```typescript
import { ChatOpenAI, OpenAIEmbeddings } from '@langchain/openai';
import { MemoryVectorStore } from 'langchain/vectorstores/memory';
import { RecursiveCharacterTextSplitter } from 'langchain/text_splitter';
import { ChatPromptTemplate } from '@langchain/core/prompts';
import { StringOutputParser } from '@langchain/core/output_parsers';
import { RunnableSequence, RunnablePassthrough } from '@langchain/core/runnables';

class BasicRAG {
  private vectorStore: MemoryVectorStore;
  private model: ChatOpenAI;
  private embeddings: OpenAIEmbeddings;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
    this.embeddings = new OpenAIEmbeddings();
  }
  
  async initialize(documents: string[]) {
    // Split documents
    const splitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
      chunkOverlap: 200
    });
    
    const docs = await splitter.createDocuments(documents);
    
    // Create vector store
    this.vectorStore = await MemoryVectorStore.fromDocuments(
      docs,
      this.embeddings
    );
  }
  
  async query(question: string): Promise<string> {
    // Retrieve relevant documents
    const retriever = this.vectorStore.asRetriever({ k: 4 });
    
    // Create prompt
    const prompt = ChatPromptTemplate.fromTemplate(`
Answer the question based on the context below.

Context: {context}

Question: {question}

Answer:`);
    
    // Build chain
    const chain = RunnableSequence.from([
      {
        context: retriever.pipe(docs => 
          docs.map(d => d.pageContent).join('\n\n')
        ),
        question: new RunnablePassthrough()
      },
      prompt,
      this.model,
      new StringOutputParser()
    ]);
    
    return await chain.invoke(question);
  }
}

// Verwendung
const rag = new BasicRAG();
await rag.initialize([
  'TypeScript is a typed superset of JavaScript.',
  'React is a JavaScript library for building UIs.',
  'Next.js is a React framework for production.'
]);

const answer = await rag.query('What is TypeScript?');
console.log(answer);
```

---

## Document Loading & Processing

### Multi-Format Loader

```typescript
import { PDFLoader } from 'langchain/document_loaders/fs/pdf';
import { TextLoader } from 'langchain/document_loaders/fs/text';
import { CSVLoader } from 'langchain/document_loaders/fs/csv';
import { CheerioWebBaseLoader } from 'langchain/document_loaders/web/cheerio';

class DocumentLoader {
  async loadDocument(path: string) {
    const extension = path.split('.').pop()?.toLowerCase();
    
    switch (extension) {
      case 'pdf':
        return await new PDFLoader(path).load();
      case 'txt':
      case 'md':
        return await new TextLoader(path).load();
      case 'csv':
        return await new CSVLoader(path).load();
      default:
        throw new Error(`Unsupported format: ${extension}`);
    }
  }
  
  async loadWebsite(url: string) {
    return await new CheerioWebBaseLoader(url).load();
  }
  
  async loadMultiple(paths: string[]) {
    const allDocs = [];
    
    for (const path of paths) {
      const docs = await this.loadDocument(path);
      allDocs.push(...docs);
    }
    
    return allDocs;
  }
}
```

### Smart Text Splitting

```typescript
class SmartSplitter {
  async splitByType(docs: any[], type: string) {
    let splitter;
    
    switch (type) {
      case 'code':
        splitter = RecursiveCharacterTextSplitter.fromLanguage('typescript', {
          chunkSize: 2000,
          chunkOverlap: 200
        });
        break;
        
      case 'markdown':
        splitter = RecursiveCharacterTextSplitter.fromLanguage('markdown', {
          chunkSize: 1000,
          chunkOverlap: 100
        });
        break;
        
      default:
        splitter = new RecursiveCharacterTextSplitter({
          chunkSize: 1000,
          chunkOverlap: 200,
          separators: ['\n\n', '\n', '. ', ' ', '']
        });
    }
    
    return await splitter.splitDocuments(docs);
  }
  
  async splitWithMetadata(docs: any[]) {
    const splitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
      chunkOverlap: 200
    });
    
    const chunks = await splitter.splitDocuments(docs);
    
    // Add chunk metadata
    return chunks.map((chunk, index) => ({
      ...chunk,
      metadata: {
        ...chunk.metadata,
        chunkIndex: index,
        chunkSize: chunk.pageContent.length
      }
    }));
  }
}
```

### Document Preprocessing

```typescript
class DocumentPreprocessor {
  async preprocess(docs: any[]) {
    return docs.map(doc => ({
      ...doc,
      pageContent: this.cleanText(doc.pageContent),
      metadata: {
        ...doc.metadata,
        processedAt: new Date().toISOString()
      }
    }));
  }
  
  private cleanText(text: string): string {
    return text
      .replace(/\s+/g, ' ')           // Normalize whitespace
      .replace(/\n{3,}/g, '\n\n')     // Remove excess newlines
      .replace(/[^\S\n]+/g, ' ')      // Normalize spaces
      .trim();
  }
  
  async addSummaries(docs: any[], model: ChatOpenAI) {
    const withSummaries = [];
    
    for (const doc of docs) {
      const summary = await model.invoke(
        `Summarize in 2 sentences:\n${doc.pageContent}`
      );
      
      withSummaries.push({
        ...doc,
        metadata: {
          ...doc.metadata,
          summary: summary.content
        }
      });
    }
    
    return withSummaries;
  }
}
```

---

## Embeddings & Vector Stores

### Embedding Comparison

```typescript
import { OpenAIEmbeddings } from '@langchain/openai';
import { CohereEmbeddings } from '@langchain/cohere';

class EmbeddingManager {
  async compareEmbeddings(text: string) {
    const openai = new OpenAIEmbeddings({
      modelName: 'text-embedding-3-small'
    });
    
    const cohere = new CohereEmbeddings({
      model: 'embed-english-v3.0'
    });
    
    const [openaiVec, cohereVec] = await Promise.all([
      openai.embedQuery(text),
      cohere.embedQuery(text)
    ]);
    
    return {
      openai: { dimensions: openaiVec.length, sample: openaiVec.slice(0, 5) },
      cohere: { dimensions: cohereVec.length, sample: cohereVec.slice(0, 5) }
    };
  }
}
```

### Vector Store Selection

```typescript
import { HNSWLib } from '@langchain/community/vectorstores/hnswlib';
import { PineconeStore } from '@langchain/pinecone';
import { Pinecone } from '@pinecone-database/pinecone';

class VectorStoreManager {
  // In-Memory: Development/Testing
  async createMemoryStore(docs: any[], embeddings: OpenAIEmbeddings) {
    return await MemoryVectorStore.fromDocuments(docs, embeddings);
  }
  
  // Persistent: Local Files
  async createHNSWStore(docs: any[], embeddings: OpenAIEmbeddings) {
    const store = await HNSWLib.fromDocuments(docs, embeddings);
    await store.save('vectorstore');
    return store;
  }
  
  async loadHNSWStore(embeddings: OpenAIEmbeddings) {
    return await HNSWLib.load('vectorstore', embeddings);
  }
  
  // Cloud: Production
  async createPineconeStore(docs: any[], embeddings: OpenAIEmbeddings) {
    const pinecone = new Pinecone({
      apiKey: process.env.PINECONE_API_KEY!
    });
    
    const index = pinecone.index('my-index');
    
    return await PineconeStore.fromDocuments(
      docs,
      embeddings,
      { pineconeIndex: index }
    );
  }
}
```

### Hybrid Search

```typescript
class HybridSearchStore {
  private vectorStore: MemoryVectorStore;
  private keywordIndex: Map<string, Set<number>> = new Map();
  private documents: any[] = [];
  
  async initialize(docs: any[], embeddings: OpenAIEmbeddings) {
    // Vector store
    this.vectorStore = await MemoryVectorStore.fromDocuments(
      docs,
      embeddings
    );
    
    // Keyword index
    this.documents = docs;
    this.buildKeywordIndex(docs);
  }
  
  private buildKeywordIndex(docs: any[]) {
    docs.forEach((doc, idx) => {
      const words = doc.pageContent.toLowerCase().split(/\s+/);
      
      words.forEach(word => {
        if (!this.keywordIndex.has(word)) {
          this.keywordIndex.set(word, new Set());
        }
        this.keywordIndex.get(word)!.add(idx);
      });
    });
  }
  
  async hybridSearch(query: string, k = 5) {
    // Vector search
    const vectorResults = await this.vectorStore.similaritySearch(query, k * 2);
    
    // Keyword search
    const keywords = query.toLowerCase().split(/\s+/);
    const keywordMatches = new Set<number>();
    
    keywords.forEach(keyword => {
      const matches = this.keywordIndex.get(keyword);
      if (matches) {
        matches.forEach(idx => keywordMatches.add(idx));
      }
    });
    
    const keywordResults = Array.from(keywordMatches)
      .map(idx => this.documents[idx]);
    
    // Combine and deduplicate
    const combined = [...vectorResults, ...keywordResults];
    const unique = Array.from(
      new Map(combined.map(doc => [doc.pageContent, doc])).values()
    );
    
    return unique.slice(0, k);
  }
}
```

---

## Retrieval Strategien

### Multi-Query Retrieval

```typescript
import { MultiQueryRetriever } from 'langchain/retrievers/multi_query';

class AdvancedRetrieval {
  async multiQuery(vectorStore: MemoryVectorStore, query: string) {
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    
    const retriever = MultiQueryRetriever.fromLLM({
      llm: model,
      retriever: vectorStore.asRetriever(),
      verbose: true
    });
    
    // Generates multiple query variations
    return await retriever.getRelevantDocuments(query);
  }
}
```

### Contextual Compression

```typescript
import { ContextualCompressionRetriever } from 'langchain/retrievers/contextual_compression';
import { LLMChainExtractor } from 'langchain/retrievers/document_compressors/chain_extract';

async function compressedRetrieval(vectorStore: MemoryVectorStore, query: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  const baseRetriever = vectorStore.asRetriever({ k: 10 });
  const compressor = LLMChainExtractor.fromLLM(model);
  
  const retriever = new ContextualCompressionRetriever({
    baseCompressor: compressor,
    baseRetriever: baseRetriever
  });
  
  // Returns only relevant parts
  return await retriever.getRelevantDocuments(query);
}
```

### Parent Document Retrieval

```typescript
class ParentDocumentRetriever {
  private chunkStore: MemoryVectorStore;
  private parentDocs: Map<string, string> = new Map();
  
  async initialize(docs: any[], embeddings: OpenAIEmbeddings) {
    const chunks = [];
    
    for (const doc of docs) {
      const parentId = `parent-${Date.now()}-${Math.random()}`;
      this.parentDocs.set(parentId, doc.pageContent);
      
      // Create smaller chunks
      const chunkTexts = this.createChunks(doc.pageContent, 200);
      
      chunkTexts.forEach(text => {
        chunks.push({
          pageContent: text,
          metadata: { parentId, source: doc.metadata.source }
        });
      });
    }
    
    this.chunkStore = await MemoryVectorStore.fromDocuments(
      chunks,
      embeddings
    );
  }
  
  async retrieve(query: string, k = 4) {
    // Search chunks
    const chunks = await this.chunkStore.similaritySearch(query, k);
    
    // Get parent documents
    const parentIds = new Set(
      chunks.map(chunk => chunk.metadata.parentId)
    );
    
    return Array.from(parentIds).map(id => ({
      pageContent: this.parentDocs.get(id)!,
      metadata: { parentId: id }
    }));
  }
  
  private createChunks(text: string, size: number): string[] {
    const chunks = [];
    for (let i = 0; i < text.length; i += size) {
      chunks.push(text.slice(i, i + size));
    }
    return chunks;
  }
}
```

---

## Advanced Retrieval

### Reranking

```typescript
class RerankingRetriever {
  private vectorStore: MemoryVectorStore;
  private model: ChatOpenAI;
  
  constructor(vectorStore: MemoryVectorStore) {
    this.vectorStore = vectorStore;
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  async retrieve(query: string, k = 4) {
    // Initial retrieval (get more candidates)
    const candidates = await this.vectorStore.similaritySearch(query, k * 3);
    
    // Rerank with LLM
    const ranked = await this.rerank(query, candidates);
    
    return ranked.slice(0, k);
  }
  
  private async rerank(query: string, docs: any[]) {
    const scores = await Promise.all(
      docs.map(doc => this.scoreRelevance(query, doc))
    );
    
    return docs
      .map((doc, idx) => ({ doc, score: scores[idx] }))
      .sort((a, b) => b.score - a.score)
      .map(item => item.doc);
  }
  
  private async scoreRelevance(query: string, doc: any): Promise<number> {
    const prompt = `Rate relevance (0-10):
Query: ${query}
Document: ${doc.pageContent.slice(0, 500)}

Score:`;

    const response = await this.model.invoke(prompt);
    const match = (response.content as string).match(/\d+/);
    return match ? parseInt(match[0]) : 0;
  }
}
```

### Query Expansion

```typescript
class QueryExpander {
  private model: ChatOpenAI;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  async expandQuery(query: string): Promise<string[]> {
    const prompt = `Generate 3 alternative phrasings of this query:
"${query}"

Return only the alternatives, one per line.`;

    const response = await this.model.invoke(prompt);
    const queries = (response.content as string)
      .split('\n')
      .filter(q => q.trim().length > 0);
    
    return [query, ...queries];
  }
  
  async retrieveWithExpansion(
    vectorStore: MemoryVectorStore,
    query: string,
    k = 4
  ) {
    const queries = await this.expandQuery(query);
    
    // Search with all queries
    const allResults = await Promise.all(
      queries.map(q => vectorStore.similaritySearch(q, k))
    );
    
    // Deduplicate
    const unique = Array.from(
      new Map(
        allResults.flat().map(doc => [doc.pageContent, doc])
      ).values()
    );
    
    return unique.slice(0, k);
  }
}
```

### Metadata Filtering

```typescript
class FilteredRetriever {
  private vectorStore: MemoryVectorStore;
  
  async retrieve(query: string, filters: any = {}, k = 4) {
    const retriever = this.vectorStore.asRetriever({
      k: k * 2,
      filter: filters
    });
    
    const docs = await retriever.getRelevantDocuments(query);
    
    // Post-filter by metadata
    return docs
      .filter(doc => this.matchesFilters(doc, filters))
      .slice(0, k);
  }
  
  private matchesFilters(doc: any, filters: any): boolean {
    for (const [key, value] of Object.entries(filters)) {
      if (doc.metadata[key] !== value) {
        return false;
      }
    }
    return true;
  }
  
  // Beispiel
  async searchBySource(query: string, source: string) {
    return await this.retrieve(query, { source }, 4);
  }
  
  async searchByDate(query: string, startDate: Date, endDate: Date) {
    const docs = await this.retrieve(query, {}, 20);
    
    return docs.filter(doc => {
      const docDate = new Date(doc.metadata.date);
      return docDate >= startDate && docDate <= endDate;
    });
  }
}
```

---

## Generation & Prompting

### RAG Prompt Templates

```typescript
class RAGPrompts {
  static basic() {
    return ChatPromptTemplate.fromTemplate(`
Answer based on the context below. If you cannot answer from the context, say so.

Context:
{context}

Question: {question}

Answer:`);
  }
  
  static withSources() {
    return ChatPromptTemplate.fromTemplate(`
Answer based on the context below and cite sources.

Context:
{context}

Question: {question}

Provide a detailed answer with source references [1], [2], etc.`);
  }
  
  static conversational() {
    return ChatPromptTemplate.fromMessages([
      ['system', `You are a helpful assistant. Use this context to answer:
{context}`],
      ['placeholder', '{chat_history}'],
      ['human', '{question}']
    ]);
  }
  
  static structured() {
    return ChatPromptTemplate.fromTemplate(`
Use the context to provide a structured answer.

Context:
{context}

Question: {question}

Format your answer as:
Summary: [brief answer]
Details: [detailed explanation]
Sources: [relevant sources]`);
  }
}
```

### Citation Generation

```typescript
class CitationRAG {
  private vectorStore: MemoryVectorStore;
  private model: ChatOpenAI;
  
  async queryWithCitations(question: string) {
    // Retrieve
    const docs = await this.vectorStore.similaritySearch(question, 4);
    
    // Build context with citations
    const context = docs
      .map((doc, idx) => `[${idx + 1}] ${doc.pageContent}`)
      .join('\n\n');
    
    // Generate with citations
    const prompt = `Answer using citations [1], [2], etc.

Context:
${context}

Question: ${question}

Answer with inline citations:`;

    const response = await this.model.invoke(prompt);
    
    return {
      answer: response.content,
      sources: docs.map((doc, idx) => ({
        id: idx + 1,
        content: doc.pageContent,
        metadata: doc.metadata
      }))
    };
  }
}
```

### Answer Validation

```typescript
class ValidatedRAG {
  private model: ChatOpenAI;
  
  async generateWithValidation(question: string, context: string) {
    // Generate answer
    const answer = await this.generate(question, context);
    
    // Validate answer
    const isValid = await this.validate(question, context, answer);
    
    if (!isValid) {
      // Regenerate with stricter prompt
      return await this.generateStrict(question, context);
    }
    
    return answer;
  }
  
  private async generate(question: string, context: string) {
    const prompt = `Context: ${context}\n\nQuestion: ${question}\n\nAnswer:`;
    const response = await this.model.invoke(prompt);
    return response.content as string;
  }
  
  private async validate(question: string, context: string, answer: string) {
    const prompt = `Is this answer supported by the context?

Context: ${context}
Question: ${question}
Answer: ${answer}

Reply only 'yes' or 'no':`;

    const response = await this.model.invoke(prompt);
    return (response.content as string).toLowerCase().includes('yes');
  }
  
  private async generateStrict(question: string, context: string) {
    const prompt = `Answer ONLY using information from the context. Do not add external knowledge.

Context: ${context}

Question: ${question}

Answer:`;

    const response = await this.model.invoke(prompt);
    return response.content as string;
  }
}
```

---

## RAG Optimization

### Chunk Size Optimization

```typescript
class ChunkOptimizer {
  async findOptimalSize(docs: any[], testQueries: string[]) {
    const sizes = [250, 500, 1000, 2000];
    const results = [];
    
    for (const size of sizes) {
      console.log(`Testing chunk size: ${size}`);
      
      const splitter = new RecursiveCharacterTextSplitter({
        chunkSize: size,
        chunkOverlap: size * 0.2
      });
      
      const chunks = await splitter.splitDocuments(docs);
      const embeddings = new OpenAIEmbeddings();
      const vectorStore = await MemoryVectorStore.fromDocuments(
        chunks,
        embeddings
      );
      
      // Test retrieval quality
      const scores = await Promise.all(
        testQueries.map(q => this.evaluateRetrieval(vectorStore, q))
      );
      
      const avgScore = scores.reduce((a, b) => a + b, 0) / scores.length;
      
      results.push({ size, avgScore, numChunks: chunks.length });
    }
    
    return results.sort((a, b) => b.avgScore - a.avgScore);
  }
  
  private async evaluateRetrieval(store: MemoryVectorStore, query: string) {
    const docs = await store.similaritySearchWithScore(query, 3);
    return docs.reduce((sum, [, score]) => sum + score, 0) / docs.length;
  }
}
```

### Caching

```typescript
class CachedRAG {
  private cache: Map<string, any> = new Map();
  private vectorStore: MemoryVectorStore;
  private model: ChatOpenAI;
  
  async query(question: string) {
    // Check cache
    const cacheKey = this.getCacheKey(question);
    
    if (this.cache.has(cacheKey)) {
      console.log('Cache hit');
      return this.cache.get(cacheKey);
    }
    
    console.log('Cache miss');
    
    // Retrieve and generate
    const docs = await this.vectorStore.similaritySearch(question, 4);
    const context = docs.map(d => d.pageContent).join('\n\n');
    
    const response = await this.model.invoke(
      `Context: ${context}\n\nQuestion: ${question}\n\nAnswer:`
    );
    
    const result = {
      answer: response.content,
      sources: docs
    };
    
    // Cache result
    this.cache.set(cacheKey, result);
    
    return result;
  }
  
  private getCacheKey(question: string): string {
    return question.toLowerCase().trim();
  }
  
  clearCache() {
    this.cache.clear();
  }
}
```

---

## Evaluation & Metrics

### RAG Metrics

```typescript
class RAGEvaluator {
  private model: ChatOpenAI;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  async evaluateRetrieval(query: string, retrieved: any[], expected: string[]) {
    // Precision: relevant docs / retrieved docs
    const relevant = retrieved.filter(doc =>
      expected.some(exp => doc.pageContent.includes(exp))
    );
    
    const precision = relevant.length / retrieved.length;
    
    // Recall: relevant docs / total relevant
    const recall = relevant.length / expected.length;
    
    // F1 Score
    const f1 = 2 * (precision * recall) / (precision + recall);
    
    return { precision, recall, f1 };
  }
  
  async evaluateGeneration(question: string, answer: string, groundTruth: string) {
    // Faithfulness: answer supported by context
    const faithfulness = await this.scoreFaithfulness(answer);
    
    // Relevance: answer addresses question
    const relevance = await this.scoreRelevance(question, answer);
    
    // Correctness: compared to ground truth
    const correctness = await this.scoreCorrectness(answer, groundTruth);
    
    return { faithfulness, relevance, correctness };
  }
  
  private async scoreFaithfulness(answer: string): Promise<number> {
    const prompt = `Rate answer faithfulness (0-1):
Answer: ${answer}

Score:`;

    const response = await this.model.invoke(prompt);
    const match = (response.content as string).match(/[\d.]+/);
    return match ? parseFloat(match[0]) : 0;
  }
  
  private async scoreRelevance(question: string, answer: string): Promise<number> {
    const prompt = `Rate answer relevance (0-1):
Question: ${question}
Answer: ${answer}

Score:`;

    const response = await this.model.invoke(prompt);
    const match = (response.content as string).match(/[\d.]+/);
    return match ? parseFloat(match[0]) : 0;
  }
  
  private async scoreCorrectness(answer: string, groundTruth: string): Promise<number> {
    const prompt = `Compare answers (0-1):
Answer: ${answer}
Ground Truth: ${groundTruth}

Score:`;

    const response = await this.model.invoke(prompt);
    const match = (response.content as string).match(/[\d.]+/);
    return match ? parseFloat(match[0]) : 0;
  }
}
```

---

## Production RAG

### Production-Ready System

```typescript
class ProductionRAG {
  private vectorStore: any;
  private model: ChatOpenAI;
  private embeddings: OpenAIEmbeddings;
  private cache: Map<string, any> = new Map();
  
  async initialize() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4', temperature: 0 });
    this.embeddings = new OpenAIEmbeddings();
    
    // Load persistent vector store
    const pinecone = new Pinecone({
      apiKey: process.env.PINECONE_API_KEY!
    });
    
    const index = pinecone.index('production-index');
    
    this.vectorStore = await PineconeStore.fromExistingIndex(
      this.embeddings,
      { pineconeIndex: index }
    );
  }
  
  async query(question: string, options: any = {}) {
    try {
      // Check cache
      if (!options.skipCache) {
        const cached = this.cache.get(question);
        if (cached) return cached;
      }
      
      // Retrieve
      const docs = await this.retrieve(question, options);
      
      // Generate
      const answer = await this.generate(question, docs, options);
      
      // Validate
      if (options.validate) {
        const isValid = await this.validate(question, answer, docs);
        if (!isValid) {
          throw new Error('Answer validation failed');
        }
      }
      
      // Cache
      const result = { answer, sources: docs, timestamp: Date.now() };
      this.cache.set(question, result);
      
      return result;
      
    } catch (error) {
      console.error('RAG query failed:', error);
      return {
        answer: 'Sorry, I encountered an error processing your question.',
        error: error.message
      };
    }
  }
  
  private async retrieve(question: string, options: any) {
    const k = options.k || 4;
    return await this.vectorStore.similaritySearch(question, k);
  }
  
  private async generate(question: string, docs: any[], options: any) {
    const context = docs.map(d => d.pageContent).join('\n\n');
    
    const prompt = options.withCitations
      ? this.buildCitationPrompt(question, docs)
      : this.buildBasicPrompt(question, context);
    
    const response = await this.model.invoke(prompt);
    return response.content;
  }
  
  private buildBasicPrompt(question: string, context: string) {
    return `Answer based on context:

${context}

Question: ${question}

Answer:`;
  }
  
  private buildCitationPrompt(question: string, docs: any[]) {
    const context = docs
      .map((d, i) => `[${i + 1}] ${d.pageContent}`)
      .join('\n\n');
    
    return `Answer with citations:

${context}

Question: ${question}

Answer with [1], [2] citations:`;
  }
  
  private async validate(question: string, answer: string, docs: any[]) {
    // Basic validation logic
    return answer.length > 10 && !answer.includes('I don\'t know');
  }
  
  async addDocuments(docs: any[]) {
    const splitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
      chunkOverlap: 200
    });
    
    const chunks = await splitter.splitDocuments(docs);
    await this.vectorStore.addDocuments(chunks);
    
    // Clear cache after adding documents
    this.cache.clear();
  }
}
```

---

## Praktische Projekte

### Projekt 1: Documentation Q&A System

```typescript
class DocQASystem {
  private rag: ProductionRAG;
  
  async initialize(docPaths: string[]) {
    this.rag = new ProductionRAG();
    await this.rag.initialize();
    
    // Load documentation
    const loader = new DocumentLoader();
    const docs = await loader.loadMultiple(docPaths);
    
    // Add to RAG
    await this.rag.addDocuments(docs);
  }
  
  async ask(question: string) {
    return await this.rag.query(question, {
      withCitations: true,
      validate: true,
      k: 5
    });
  }
  
  async batchQuestions(questions: string[]) {
    return await Promise.all(
      questions.map(q => this.ask(q))
    );
  }
}

// Verwendung
const qaSystem = new DocQASystem();
await qaSystem.initialize([
  'docs/typescript.md',
  'docs/react.md',
  'docs/nextjs.md'
]);

const result = await qaSystem.ask('How do I use TypeScript with React?');
console.log(result.answer);
console.log('Sources:', result.sources);
```

### Projekt 2: Conversational RAG

```typescript
import { BufferMemory } from 'langchain/memory';

class ConversationalRAG {
  private rag: ProductionRAG;
  private memory: BufferMemory;
  
  async initialize() {
    this.rag = new ProductionRAG();
    await this.rag.initialize();
    
    this.memory = new BufferMemory({
      returnMessages: true,
      memoryKey: 'chat_history'
    });
  }
  
  async chat(message: string) {
    // Get chat history
    const history = await this.memory.loadMemoryVariables({});
    
    // Build context-aware query
    const contextualQuery = await this.buildContextualQuery(
      message,
      history.chat_history
    );
    
    // Query RAG
    const result = await this.rag.query(contextualQuery);
    
    // Save to memory
    await this.memory.saveContext(
      { input: message },
      { output: result.answer }
    );
    
    return result;
  }
  
  private async buildContextualQuery(message: string, history: any[]) {
    if (history.length === 0) {
      return message;
    }
    
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    
    const prompt = `Given this conversation history, rephrase the question to be standalone:

History:
${history.map(m => m.content).join('\n')}

New question: ${message}

Standalone question:`;

    const response = await model.invoke(prompt);
    return response.content as string;
  }
  
  resetConversation() {
    this.memory = new BufferMemory({
      returnMessages: true,
      memoryKey: 'chat_history'
    });
  }
}
```

### Projekt 3: Multi-Modal RAG

```typescript
class MultiModalRAG {
  private textRAG: ProductionRAG;
  private imageDescriptions: Map<string, string> = new Map();
  
  async initialize() {
    this.textRAG = new ProductionRAG();
    await this.textRAG.initialize();
  }
  
  async addDocument(path: string, images?: string[]) {
    // Load text
    const loader = new DocumentLoader();
    const docs = await loader.loadDocument(path);
    
    // Process images if present
    if (images) {
      for (const imagePath of images) {
        const description = await this.describeImage(imagePath);
        this.imageDescriptions.set(imagePath, description);
        
        // Add image description as document
        docs.push({
          pageContent: `Image: ${imagePath}\nDescription: ${description}`,
          metadata: { type: 'image', path: imagePath }
        });
      }
    }
    
    await this.textRAG.addDocuments(docs);
  }
  
  private async describeImage(imagePath: string): Promise<string> {
    // Simulate image description (use actual vision model in production)
    return `Description of ${imagePath}`;
  }
  
  async query(question: string) {
    const result = await this.textRAG.query(question);
    
    // Check if response references images
    const imageRefs = this.extractImageReferences(result.sources);
    
    return {
      ...result,
      images: imageRefs
    };
  }
  
  private extractImageReferences(sources: any[]) {
    return sources
      .filter(doc => doc.metadata.type === 'image')
      .map(doc => ({
        path: doc.metadata.path,
        description: this.imageDescriptions.get(doc.metadata.path)
      }));
  }
}
```

---

## Zusammenfassung

Du hast gelernt:

✅ **RAG-Grundlagen**: Architektur, Pipeline, vs Fine-Tuning
✅ **Document Processing**: Loading, Splitting, Preprocessing
✅ **Embeddings**: Vector Stores, Hybrid Search
✅ **Retrieval**: Multi-Query, Compression, Parent Documents
✅ **Advanced Retrieval**: Reranking, Query Expansion, Filtering
✅ **Generation**: Prompt Templates, Citations, Validation
✅ **Optimization**: Chunk Size, Caching
✅ **Evaluation**: Metrics, Precision, Recall
✅ **Production**: Error Handling, Caching, Scaling
✅ **Projekte**: Doc Q&A, Conversational RAG, Multi-Modal

### Best Practices

- [ ] Chunk Size für Daten optimieren (500-1000 Tokens)
- [ ] Overlap von 10-20% verwenden
- [ ] Metadata für Filterung nutzen
- [ ] Retrieval evaluieren und iterieren
- [ ] Citations für Transparenz
- [ ] Caching für Performance
- [ ] Error Handling implementieren
- [ ] Monitoring & Logging

Viel Erfolg mit RAG-Systemen! 🔍
