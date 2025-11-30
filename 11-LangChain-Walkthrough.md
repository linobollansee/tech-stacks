# LangChain - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in LangChain](#einführung-in-langchain)
2. [Setup & Installation](#setup--installation)
3. [LLMs & Chat Models](#llms--chat-models)
4. [Prompts & Prompt Templates](#prompts--prompt-templates)
5. [Chains](#chains)
6. [Memory](#memory)
7. [Agents](#agents)
8. [Tools](#tools)
9. [Document Loaders](#document-loaders)
10. [Text Splitters](#text-splitters)
11. [Vector Stores & Embeddings](#vector-stores--embeddings)
12. [Retrievers](#retrievers)
13. [Output Parsers](#output-parsers)
14. [LangChain Expression Language (LCEL)](#langchain-expression-language-lcel)
15. [Praktische Projekte](#praktische-projekte)

---

## Einführung in LangChain

### Was ist LangChain?

**LangChain** ist ein Framework zum Entwickeln von Anwendungen mit Large Language Models (LLMs).

### Hauptkonzepte

```
LangChain
  ├─ Models        → LLMs, Chat Models
  ├─ Prompts       → Prompt Templates, Few-Shot Learning
  ├─ Chains        → Verkettung von Komponenten
  ├─ Memory        → Konversations-Kontext speichern
  ├─ Agents        → Autonome Entscheidungen treffen
  ├─ Tools         → Externe Funktionen aufrufen
  └─ RAG           → Retrieval Augmented Generation
```

### Anwendungsfälle

**Chatbots** 💬
- Customer Support
- Persönliche Assistenten
- FAQ-Bots

**Dokumenten-Analyse** 📄
- PDF-Zusammenfassungen
- Dokument-Q&A
- Knowledge Base

**Code-Generierung** 💻
- Code-Assistent
- Bug-Fixing
- Dokumentation

**Data Analysis** 📊
- SQL-Generierung
- Daten-Extraktion
- Report-Generierung

---

## Setup & Installation

### Installation

```bash
# LangChain Core
npm install langchain

# OpenAI Integration
npm install @langchain/openai

# Community Packages
npm install @langchain/community

# TypeScript
npm install --save-dev typescript @types/node

# Optional: Vector Stores
npm install hnswlib-node  # HNSWLib
npm install @pinecone-database/pinecone  # Pinecone
npm install chromadb  # ChromaDB
```

### Environment Setup

```bash
# .env
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=...
PINECONE_API_KEY=...
```

### Basis-Setup

```typescript
// src/index.ts
import { ChatOpenAI } from '@langchain/openai';
import * as dotenv from 'dotenv';

dotenv.config();

// Model initialisieren
const model = new ChatOpenAI({
  modelName: 'gpt-4',
  temperature: 0.7,
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// Einfache Anfrage
const response = await model.invoke('Hello, how are you?');
console.log(response.content);
```

---

## LLMs & Chat Models

### Chat Models

```typescript
import { ChatOpenAI } from '@langchain/openai';
import { HumanMessage, SystemMessage, AIMessage } from '@langchain/core/messages';

const chat = new ChatOpenAI({
  modelName: 'gpt-4',
  temperature: 0.7,
});

// Einzelne Message
const response1 = await chat.invoke([
  new HumanMessage('What is the capital of France?')
]);

console.log(response1.content);  // "Paris"

// Mit System Message
const response2 = await chat.invoke([
  new SystemMessage('You are a helpful assistant that speaks like a pirate.'),
  new HumanMessage('What is the capital of France?')
]);

console.log(response2.content);  // "Ahoy! The capital be Paris, matey!"

// Konversation
const messages = [
  new SystemMessage('You are a helpful AI assistant.'),
  new HumanMessage('Hello!'),
  new AIMessage('Hi! How can I help you today?'),
  new HumanMessage('What is 2+2?')
];

const response3 = await chat.invoke(messages);
console.log(response3.content);  // "2+2 equals 4."
```

### Streaming

```typescript
const chat = new ChatOpenAI({
  modelName: 'gpt-4',
  streaming: true,
});

const stream = await chat.stream([
  new HumanMessage('Write a short poem about TypeScript.')
]);

for await (const chunk of stream) {
  process.stdout.write(chunk.content);
}
```

### Model Parameters

```typescript
const chat = new ChatOpenAI({
  modelName: 'gpt-4',
  temperature: 0.7,      // Kreativität (0-2)
  maxTokens: 1000,       // Max Tokens
  topP: 1,               // Nucleus sampling
  frequencyPenalty: 0,   // Wiederholungen vermeiden
  presencePenalty: 0,    // Neue Themen fördern
  n: 1,                  // Anzahl Responses
});
```

### Alternative Models

```typescript
// Anthropic Claude
import { ChatAnthropic } from '@langchain/anthropic';

const claude = new ChatAnthropic({
  modelName: 'claude-3-opus-20240229',
  anthropicApiKey: process.env.ANTHROPIC_API_KEY,
});

// Google Gemini
import { ChatGoogleGenerativeAI } from '@langchain/google-genai';

const gemini = new ChatGoogleGenerativeAI({
  modelName: 'gemini-pro',
  apiKey: process.env.GOOGLE_API_KEY,
});

// Ollama (Local)
import { ChatOllama } from '@langchain/community/chat_models/ollama';

const ollama = new ChatOllama({
  baseUrl: 'http://localhost:11434',
  model: 'llama2',
});
```

---

## Prompts & Prompt Templates

### Prompt Templates

```typescript
import { PromptTemplate } from '@langchain/core/prompts';

// Einfaches Template
const template = PromptTemplate.fromTemplate(
  'Tell me a {adjective} joke about {topic}.'
);

const prompt = await template.format({
  adjective: 'funny',
  topic: 'programming'
});

console.log(prompt);
// "Tell me a funny joke about programming."

// Mit Model verwenden
const response = await model.invoke(prompt);
```

### Chat Prompt Templates

```typescript
import { 
  ChatPromptTemplate,
  SystemMessagePromptTemplate,
  HumanMessagePromptTemplate
} from '@langchain/core/prompts';

const chatPrompt = ChatPromptTemplate.fromMessages([
  SystemMessagePromptTemplate.fromTemplate(
    'You are a helpful assistant that translates {input_language} to {output_language}.'
  ),
  HumanMessagePromptTemplate.fromTemplate('{text}')
]);

const messages = await chatPrompt.formatMessages({
  input_language: 'English',
  output_language: 'German',
  text: 'Hello, how are you?'
});

const response = await chat.invoke(messages);
console.log(response.content);  // "Hallo, wie geht es dir?"
```

### Few-Shot Prompting

```typescript
import { FewShotPromptTemplate } from '@langchain/core/prompts';

// Beispiele
const examples = [
  { input: 'happy', output: 'sad' },
  { input: 'tall', output: 'short' },
  { input: 'hot', output: 'cold' }
];

// Example Template
const exampleTemplate = PromptTemplate.fromTemplate(
  'Input: {input}\nOutput: {output}'
);

// Few-Shot Template
const fewShotPrompt = new FewShotPromptTemplate({
  examples,
  examplePrompt: exampleTemplate,
  prefix: 'Give the antonym of every input',
  suffix: 'Input: {word}\nOutput:',
  inputVariables: ['word']
});

const prompt = await fewShotPrompt.format({ word: 'big' });
console.log(prompt);

const response = await model.invoke(prompt);
console.log(response.content);  // "small"
```

### Partial Templates

```typescript
const template = PromptTemplate.fromTemplate(
  'Tell me a joke about {topic} in the style of {style}'
);

// Partial mit fixem Wert
const partialPrompt = await template.partial({ style: 'Shakespeare' });

// Nur topic muss noch angegeben werden
const prompt = await partialPrompt.format({ topic: 'programming' });
```

---

## Chains

### Simple Chain

```typescript
import { LLMChain } from 'langchain/chains';

const prompt = PromptTemplate.fromTemplate(
  'What is a good name for a company that makes {product}?'
);

const chain = new LLMChain({
  llm: model,
  prompt: prompt
});

const result = await chain.invoke({ product: 'colorful socks' });
console.log(result.text);
```

### Sequential Chain

```typescript
import { SimpleSequentialChain } from 'langchain/chains';

// Chain 1: Generiere Firmennamen
const nameChain = new LLMChain({
  llm: model,
  prompt: PromptTemplate.fromTemplate(
    'What is a good name for a company that makes {product}?'
  )
});

// Chain 2: Schreibe Slogan
const sloganChain = new LLMChain({
  llm: model,
  prompt: PromptTemplate.fromTemplate(
    'Write a catchy slogan for the following company: {company_name}'
  )
});

// Verkette beide
const overallChain = new SimpleSequentialChain({
  chains: [nameChain, sloganChain],
  verbose: true
});

const result = await overallChain.invoke({ input: 'colorful socks' });
console.log(result.output);
```

### LCEL (LangChain Expression Language)

```typescript
import { RunnableSequence } from '@langchain/core/runnables';

// Modern: Mit LCEL
const chain = RunnableSequence.from([
  prompt,
  model,
  new StringOutputParser()
]);

const result = await chain.invoke({ product: 'colorful socks' });
```

### Router Chain

```typescript
import { MultiPromptChain } from 'langchain/chains';

const physicsPrompt = PromptTemplate.fromTemplate(
  'You are a physics expert. Answer: {input}'
);

const mathPrompt = PromptTemplate.fromTemplate(
  'You are a math expert. Answer: {input}'
);

const promptInfos = [
  {
    name: 'physics',
    description: 'Good for answering physics questions',
    promptTemplate: physicsPrompt
  },
  {
    name: 'math',
    description: 'Good for answering math questions',
    promptTemplate: mathPrompt
  }
];

const chain = MultiPromptChain.fromPrompts(model, promptInfos);

const result = await chain.invoke({
  input: 'What is the speed of light?'
});
```

---

## Memory

### Buffer Memory

```typescript
import { BufferMemory } from 'langchain/memory';
import { ConversationChain } from 'langchain/chains';

const memory = new BufferMemory();

const chain = new ConversationChain({
  llm: model,
  memory: memory
});

// Erste Nachricht
await chain.invoke({ input: 'Hi! My name is Max.' });
// "Hello Max! How can I help you today?"

// Zweite Nachricht - erinnert sich an Namen
await chain.invoke({ input: 'What is my name?' });
// "Your name is Max."

// Memory anzeigen
console.log(await memory.loadMemoryVariables({}));
```

### Buffer Window Memory

```typescript
import { BufferWindowMemory } from 'langchain/memory';

// Nur die letzten k Nachrichten speichern
const memory = new BufferWindowMemory({
  k: 3,  // Nur letzte 3 Interaktionen
  returnMessages: true
});

const chain = new ConversationChain({
  llm: model,
  memory: memory
});
```

### Summary Memory

```typescript
import { ConversationSummaryMemory } from 'langchain/memory';

const memory = new ConversationSummaryMemory({
  llm: model,
  maxTokenLimit: 100
});

// Fasst lange Konversationen zusammen
const chain = new ConversationChain({
  llm: model,
  memory: memory
});
```

### Entity Memory

```typescript
import { EntityMemory } from 'langchain/memory';

const memory = new EntityMemory({
  llm: model
});

// Extrahiert und speichert Entities
const chain = new ConversationChain({
  llm: model,
  memory: memory
});

await chain.invoke({ input: 'My friend Alice works at Google.' });
await chain.invoke({ input: 'Where does Alice work?' });
// "Alice works at Google."
```

---

## Agents

### Agent Basics

```typescript
import { initializeAgentExecutorWithOptions } from 'langchain/agents';
import { Calculator } from 'langchain/tools/calculator';

const tools = [new Calculator()];

const executor = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'zero-shot-react-description',
  verbose: true
});

const result = await executor.invoke({
  input: 'What is 25 * 4 + 10?'
});

console.log(result.output);  // "110"
```

### Custom Agent

```typescript
import { Tool } from '@langchain/core/tools';

class WeatherTool extends Tool {
  name = 'weather';
  description = 'Get the current weather for a location';

  async _call(location: string): Promise<string> {
    // API-Call simulieren
    const weatherData = {
      'Berlin': 'Sunny, 22°C',
      'London': 'Rainy, 15°C',
      'New York': 'Cloudy, 18°C'
    };

    return weatherData[location] || 'Weather data not available';
  }
}

const tools = [new WeatherTool()];

const executor = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'zero-shot-react-description'
});

const result = await executor.invoke({
  input: 'What is the weather in Berlin?'
});
```

### Agent with Multiple Tools

```typescript
import { DuckDuckGoSearch } from '@langchain/community/tools/duckduckgo_search';
import { Calculator } from '@langchain/community/tools/calculator';

const tools = [
  new DuckDuckGoSearch(),
  new Calculator(),
  new WeatherTool()
];

const executor = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'zero-shot-react-description',
  verbose: true
});

const result = await executor.invoke({
  input: 'Search for the population of Berlin and multiply it by 2'
});
```

---

## Tools

### Built-in Tools

```typescript
import { Calculator } from '@langchain/community/tools/calculator';
import { WikipediaQueryRun } from '@langchain/community/tools/wikipedia_query_run';
import { RequestsGetTool } from 'langchain/tools';

const tools = [
  new Calculator(),
  new WikipediaQueryRun(),
  new RequestsGetTool()
];
```

### Custom Tool

```typescript
import { DynamicTool } from '@langchain/core/tools';

const customTool = new DynamicTool({
  name: 'get-user-info',
  description: 'Get information about a user by ID',
  func: async (userId: string) => {
    // API-Call
    const users = {
      '1': { name: 'Max', email: 'max@example.com' },
      '2': { name: 'Anna', email: 'anna@example.com' }
    };

    const user = users[userId];
    return user ? JSON.stringify(user) : 'User not found';
  }
});

const tools = [customTool];
```

### Structured Tool

```typescript
import { StructuredTool } from '@langchain/core/tools';
import { z } from 'zod';

class UserSearchTool extends StructuredTool {
  name = 'search-users';
  description = 'Search for users by name and age';

  schema = z.object({
    name: z.string().describe('User name to search for'),
    age: z.number().optional().describe('User age (optional)')
  });

  async _call({ name, age }: { name: string; age?: number }): Promise<string> {
    // Suche implementieren
    const results = await searchUsers(name, age);
    return JSON.stringify(results);
  }
}

const tool = new UserSearchTool();
```

---

## Document Loaders

### Text Loader

```typescript
import { TextLoader } from 'langchain/document_loaders/fs/text';

const loader = new TextLoader('path/to/file.txt');
const docs = await loader.load();

console.log(docs[0].pageContent);
console.log(docs[0].metadata);
```

### PDF Loader

```typescript
import { PDFLoader } from 'langchain/document_loaders/fs/pdf';

const loader = new PDFLoader('path/to/file.pdf');
const docs = await loader.load();

// Jede Seite ist ein Document
docs.forEach((doc, i) => {
  console.log(`Page ${i + 1}:`, doc.pageContent);
});
```

### CSV Loader

```typescript
import { CSVLoader } from 'langchain/document_loaders/fs/csv';

const loader = new CSVLoader('path/to/file.csv');
const docs = await loader.load();

// Jede Zeile ist ein Document
docs.forEach(doc => {
  console.log(doc.pageContent);
  console.log(doc.metadata);
});
```

### Web Loader

```typescript
import { CheerioWebBaseLoader } from 'langchain/document_loaders/web/cheerio';

const loader = new CheerioWebBaseLoader('https://example.com');
const docs = await loader.load();

console.log(docs[0].pageContent);
```

### Directory Loader

```typescript
import { DirectoryLoader } from 'langchain/document_loaders/fs/directory';
import { TextLoader } from 'langchain/document_loaders/fs/text';

const loader = new DirectoryLoader(
  'path/to/directory',
  {
    '.txt': (path) => new TextLoader(path),
    '.md': (path) => new TextLoader(path)
  }
);

const docs = await loader.load();
```

---

## Text Splitters

### Character Text Splitter

```typescript
import { CharacterTextSplitter } from 'langchain/text_splitter';

const text = 'Long text content...';

const splitter = new CharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
  separator: '\n'
});

const chunks = await splitter.splitText(text);
```

### Recursive Character Text Splitter

```typescript
import { RecursiveCharacterTextSplitter } from 'langchain/text_splitter';

const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
  separators: ['\n\n', '\n', ' ', '']
});

const chunks = await splitter.splitText(text);
```

### Code Splitter

```typescript
import { RecursiveCharacterTextSplitter } from 'langchain/text_splitter';

const splitter = RecursiveCharacterTextSplitter.fromLanguage('typescript', {
  chunkSize: 1000,
  chunkOverlap: 100
});

const code = `
  function hello() {
    console.log("Hello World");
  }
`;

const chunks = await splitter.splitText(code);
```

### Split Documents

```typescript
const loader = new TextLoader('file.txt');
const docs = await loader.load();

const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 500,
  chunkOverlap: 50
});

const splitDocs = await splitter.splitDocuments(docs);

// Metadata bleibt erhalten
splitDocs.forEach(doc => {
  console.log(doc.pageContent);
  console.log(doc.metadata);
});
```

---

## Vector Stores & Embeddings

### Embeddings

```typescript
import { OpenAIEmbeddings } from '@langchain/openai';

const embeddings = new OpenAIEmbeddings({
  modelName: 'text-embedding-ada-002',
  openAIApiKey: process.env.OPENAI_API_KEY
});

// Text zu Vector
const vector = await embeddings.embedQuery('Hello World');
console.log(vector);  // [0.123, -0.456, ...]

// Multiple Texte
const vectors = await embeddings.embedDocuments([
  'Hello',
  'World',
  'TypeScript'
]);
```

### In-Memory Vector Store

```typescript
import { MemoryVectorStore } from 'langchain/vectorstores/memory';

// Dokumente erstellen
const docs = [
  { pageContent: 'TypeScript is a programming language', metadata: {} },
  { pageContent: 'Python is used for data science', metadata: {} },
  { pageContent: 'JavaScript runs in the browser', metadata: {} }
];

// Vector Store erstellen
const vectorStore = await MemoryVectorStore.fromDocuments(
  docs,
  embeddings
);

// Similarity Search
const results = await vectorStore.similaritySearch('programming', 2);
console.log(results);
```

### HNSWLib (Persistent)

```typescript
import { HNSWLib } from '@langchain/community/vectorstores/hnswlib';

// Erstellen und speichern
const vectorStore = await HNSWLib.fromDocuments(docs, embeddings);
await vectorStore.save('path/to/directory');

// Laden
const loadedVectorStore = await HNSWLib.load(
  'path/to/directory',
  embeddings
);

const results = await loadedVectorStore.similaritySearch('query', 3);
```

### Pinecone

```typescript
import { PineconeStore } from '@langchain/pinecone';
import { Pinecone } from '@pinecone-database/pinecone';

const pinecone = new Pinecone({
  apiKey: process.env.PINECONE_API_KEY!
});

const index = pinecone.index('my-index');

const vectorStore = await PineconeStore.fromDocuments(
  docs,
  embeddings,
  { pineconeIndex: index }
);

const results = await vectorStore.similaritySearch('query', 5);
```

### Similarity Search with Score

```typescript
const results = await vectorStore.similaritySearchWithScore('query', 3);

results.forEach(([doc, score]) => {
  console.log(`Score: ${score}`);
  console.log(`Content: ${doc.pageContent}`);
});
```

---

## Retrievers

### Vector Store Retriever

```typescript
const retriever = vectorStore.asRetriever({
  k: 5,  // Top 5 Ergebnisse
});

const docs = await retriever.getRelevantDocuments('What is TypeScript?');
```

### Multi-Query Retriever

```typescript
import { MultiQueryRetriever } from 'langchain/retrievers/multi_query';

const retriever = MultiQueryRetriever.fromLLM({
  llm: model,
  retriever: vectorStore.asRetriever(),
  verbose: true
});

// Generiert automatisch mehrere Queries
const docs = await retriever.getRelevantDocuments('programming languages');
```

### Contextual Compression

```typescript
import { ContextualCompressionRetriever } from 'langchain/retrievers/contextual_compression';
import { LLMChainExtractor } from 'langchain/retrievers/document_compressors/chain_extract';

const baseRetriever = vectorStore.asRetriever();
const compressor = LLMChainExtractor.fromLLM(model);

const retriever = new ContextualCompressionRetriever({
  baseCompressor: compressor,
  baseRetriever: baseRetriever
});

// Komprimiert relevante Teile
const docs = await retriever.getRelevantDocuments('query');
```

---

## Output Parsers

### String Output Parser

```typescript
import { StringOutputParser } from '@langchain/core/output_parsers';

const chain = prompt.pipe(model).pipe(new StringOutputParser());

const result = await chain.invoke({ topic: 'AI' });
// result ist string
```

### Structured Output Parser

```typescript
import { StructuredOutputParser } from 'langchain/output_parsers';

const parser = StructuredOutputParser.fromNamesAndDescriptions({
  name: "Person's name",
  age: "Person's age",
  email: "Person's email"
});

const formatInstructions = parser.getFormatInstructions();

const prompt = PromptTemplate.fromTemplate(
  `Extract information about the person.
  
  {format_instructions}
  
  Text: {text}`
);

const chain = prompt.pipe(model).pipe(parser);

const result = await chain.invoke({
  text: 'John is 25 years old and his email is john@example.com',
  format_instructions: formatInstructions
});

console.log(result);
// { name: 'John', age: '25', email: 'john@example.com' }
```

### Zod Schema Parser

```typescript
import { z } from 'zod';
import { StructuredOutputParser } from '@langchain/core/output_parsers';

const schema = z.object({
  name: z.string().describe("Person's name"),
  age: z.number().describe("Person's age"),
  email: z.string().email().describe("Person's email")
});

const parser = StructuredOutputParser.fromZodSchema(schema);

const formatInstructions = parser.getFormatInstructions();

const prompt = PromptTemplate.fromTemplate(
  `Extract person information.
  
  {format_instructions}
  
  Text: {text}`
);

const chain = prompt.pipe(model).pipe(parser);

const result = await chain.invoke({
  text: 'Max is 30 and can be reached at max@example.com',
  format_instructions: formatInstructions
});

// Type-safe!
console.log(result.name);   // string
console.log(result.age);    // number
console.log(result.email);  // string
```

### List Output Parser

```typescript
import { CommaSeparatedListOutputParser } from '@langchain/core/output_parsers';

const parser = new CommaSeparatedListOutputParser();

const prompt = PromptTemplate.fromTemplate(
  `List 5 {topic}.
  
  {format_instructions}`
);

const chain = prompt.pipe(model).pipe(parser);

const result = await chain.invoke({
  topic: 'programming languages',
  format_instructions: parser.getFormatInstructions()
});

console.log(result);  // ['Python', 'JavaScript', 'TypeScript', 'Java', 'C++']
```

---

## LangChain Expression Language (LCEL)

### Chains mit LCEL

```typescript
import { RunnableSequence, RunnablePassthrough } from '@langchain/core/runnables';

// Einfache Chain
const chain = RunnableSequence.from([
  prompt,
  model,
  new StringOutputParser()
]);

// Mit Passthrough
const chain2 = RunnableSequence.from([
  {
    context: retriever,
    question: new RunnablePassthrough()
  },
  prompt,
  model,
  new StringOutputParser()
]);

const result = await chain2.invoke('What is TypeScript?');
```

### Parallel Execution

```typescript
import { RunnableParallel } from '@langchain/core/runnables';

const chain = RunnableParallel.from({
  joke: jokeChain,
  poem: poemChain,
  story: storyChain
});

const result = await chain.invoke({ topic: 'AI' });
// {
//   joke: "Why did the AI...",
//   poem: "Roses are red...",
//   story: "Once upon a time..."
// }
```

### Branching

```typescript
import { RunnableBranch } from '@langchain/core/runnables';

const branch = RunnableBranch.from([
  [
    (input: string) => input.includes('math'),
    mathChain
  ],
  [
    (input: string) => input.includes('physics'),
    physicsChain
  ],
  defaultChain  // Fallback
]);

const result = await branch.invoke('What is 2+2?');
```

### Streaming mit LCEL

```typescript
const chain = prompt.pipe(model).pipe(new StringOutputParser());

const stream = await chain.stream({ topic: 'TypeScript' });

for await (const chunk of stream) {
  process.stdout.write(chunk);
}
```

---

## Praktische Projekte

### Projekt 1: RAG System (Retrieval Augmented Generation)

```typescript
import { ChatOpenAI } from '@langchain/openai';
import { OpenAIEmbeddings } from '@langchain/openai';
import { MemoryVectorStore } from 'langchain/vectorstores/memory';
import { RecursiveCharacterTextSplitter } from 'langchain/text_splitter';
import { PDFLoader } from 'langchain/document_loaders/fs/pdf';
import { ChatPromptTemplate } from '@langchain/core/prompts';
import { StringOutputParser } from '@langchain/core/output_parsers';
import { RunnableSequence, RunnablePassthrough } from '@langchain/core/runnables';

// 1. Dokumente laden
const loader = new PDFLoader('documentation.pdf');
const docs = await loader.load();

// 2. Text splitten
const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200
});

const splitDocs = await splitter.splitDocuments(docs);

// 3. Embeddings erstellen und Vector Store
const embeddings = new OpenAIEmbeddings();
const vectorStore = await MemoryVectorStore.fromDocuments(
  splitDocs,
  embeddings
);

// 4. Retriever erstellen
const retriever = vectorStore.asRetriever({ k: 4 });

// 5. Prompt Template
const prompt = ChatPromptTemplate.fromTemplate(`
Answer the question based only on the following context:

{context}

Question: {question}

Answer in a clear and concise way. If you don't know the answer, say "I don't know".
`);

// 6. Model
const model = new ChatOpenAI({
  modelName: 'gpt-4',
  temperature: 0
});

// 7. Chain mit LCEL
const chain = RunnableSequence.from([
  {
    context: retriever.pipe((docs) => docs.map(d => d.pageContent).join('\n\n')),
    question: new RunnablePassthrough()
  },
  prompt,
  model,
  new StringOutputParser()
]);

// 8. Frage stellen
const answer = await chain.invoke('What is TypeScript?');
console.log(answer);
```

### Projekt 2: Conversational RAG mit Memory

```typescript
import { BufferMemory } from 'langchain/memory';
import { ConversationChain } from 'langchain/chains';

// RAG Setup (wie oben)
const vectorStore = await MemoryVectorStore.fromDocuments(docs, embeddings);
const retriever = vectorStore.asRetriever();

// Memory
const memory = new BufferMemory({
  returnMessages: true,
  memoryKey: 'chat_history'
});

// Conversational Prompt
const prompt = ChatPromptTemplate.fromMessages([
  ['system', `You are a helpful assistant. Use the following context to answer questions:

{context}

If you don't know the answer, say "I don't know based on the provided context."`],
  ['placeholder', '{chat_history}'],
  ['human', '{question}']
]);

// Chain
const chain = RunnableSequence.from([
  {
    context: async (input: { question: string; chat_history: any[] }) => {
      const docs = await retriever.getRelevantDocuments(input.question);
      return docs.map(d => d.pageContent).join('\n\n');
    },
    question: (input) => input.question,
    chat_history: (input) => input.chat_history
  },
  prompt,
  model,
  new StringOutputParser()
]);

// Chat Loop
async function chat(question: string) {
  const chatHistory = await memory.chatHistory.getMessages();
  
  const answer = await chain.invoke({
    question,
    chat_history: chatHistory
  });
  
  // Memory aktualisieren
  await memory.chatHistory.addUserMessage(question);
  await memory.chatHistory.addAIChatMessage(answer);
  
  return answer;
}

// Verwendung
console.log(await chat('What is TypeScript?'));
console.log(await chat('What are its main features?'));  // Erinnert sich an Kontext
```

### Projekt 3: Agent mit Tools

```typescript
import { initializeAgentExecutorWithOptions } from 'langchain/agents';
import { DynamicTool } from '@langchain/core/tools';

// Custom Tools
const weatherTool = new DynamicTool({
  name: 'get-weather',
  description: 'Get current weather for a city. Input should be a city name.',
  func: async (city: string) => {
    // API-Call simulieren
    const weather = {
      'Berlin': '22°C, Sunny',
      'London': '15°C, Rainy',
      'Paris': '18°C, Cloudy'
    };
    return weather[city] || 'Weather data not available';
  }
});

const searchTool = new DynamicTool({
  name: 'search-database',
  description: 'Search for information in the database. Input should be a search query.',
  func: async (query: string) => {
    // Database search simulieren
    const results = await searchDatabase(query);
    return JSON.stringify(results);
  }
});

const calculatorTool = new DynamicTool({
  name: 'calculator',
  description: 'Perform mathematical calculations. Input should be a math expression.',
  func: async (expression: string) => {
    try {
      const result = eval(expression);
      return String(result);
    } catch (error) {
      return 'Invalid expression';
    }
  }
});

// Agent erstellen
const tools = [weatherTool, searchTool, calculatorTool];

const executor = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'zero-shot-react-description',
  verbose: true
});

// Agent verwenden
const result1 = await executor.invoke({
  input: 'What is the weather in Berlin?'
});

const result2 = await executor.invoke({
  input: 'Calculate 25 * 4 + 10 and tell me the weather in London'
});

console.log(result1.output);
console.log(result2.output);
```

### Projekt 4: Document Q&A System

```typescript
// Vollständiges Q&A System
class DocumentQASystem {
  private vectorStore: MemoryVectorStore;
  private chain: any;
  
  async initialize(filePaths: string[]) {
    // 1. Alle Dokumente laden
    const allDocs = [];
    
    for (const path of filePaths) {
      let loader;
      if (path.endsWith('.pdf')) {
        loader = new PDFLoader(path);
      } else if (path.endsWith('.txt')) {
        loader = new TextLoader(path);
      }
      
      const docs = await loader.load();
      allDocs.push(...docs);
    }
    
    // 2. Splitten
    const splitter = new RecursiveCharacterTextSplitter({
      chunkSize: 1000,
      chunkOverlap: 200
    });
    
    const splitDocs = await splitter.splitDocuments(allDocs);
    
    // 3. Vector Store erstellen
    const embeddings = new OpenAIEmbeddings();
    this.vectorStore = await MemoryVectorStore.fromDocuments(
      splitDocs,
      embeddings
    );
    
    // 4. Chain erstellen
    const retriever = this.vectorStore.asRetriever({ k: 4 });
    
    const prompt = ChatPromptTemplate.fromTemplate(`
    Answer based on the context below:
    
    Context: {context}
    
    Question: {question}
    
    Provide a detailed answer with sources if possible.
    `);
    
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    
    this.chain = RunnableSequence.from([
      {
        context: retriever.pipe(docs => 
          docs.map(d => `[${d.metadata.source}]\n${d.pageContent}`).join('\n\n')
        ),
        question: new RunnablePassthrough()
      },
      prompt,
      model,
      new StringOutputParser()
    ]);
  }
  
  async ask(question: string): Promise<string> {
    return await this.chain.invoke(question);
  }
  
  async similarDocuments(query: string, k: number = 4) {
    return await this.vectorStore.similaritySearch(query, k);
  }
}

// Verwendung
const qaSystem = new DocumentQASystem();
await qaSystem.initialize([
  'docs/typescript.pdf',
  'docs/react.pdf',
  'docs/nextjs.pdf'
]);

const answer = await qaSystem.ask('How do I use TypeScript with React?');
console.log(answer);
```

---

## Zusammenfassung

Du hast nun gelernt:

✅ **LangChain Basics**: Setup, Konzepte, Models
✅ **LLMs & Chat Models**: OpenAI, Anthropic, Streaming
✅ **Prompts**: Templates, Few-Shot, Partial Templates
✅ **Chains**: Simple, Sequential, Router Chains
✅ **Memory**: Buffer, Window, Summary, Entity Memory
✅ **Agents**: Zero-Shot, Custom Tools, Multi-Tool Agents
✅ **Tools**: Built-in, Custom, Structured Tools
✅ **Document Loaders**: Text, PDF, CSV, Web, Directory
✅ **Text Splitters**: Character, Recursive, Code Splitting
✅ **Vector Stores**: Embeddings, Similarity Search, Pinecone
✅ **Retrievers**: Vector Store, Multi-Query, Contextual Compression
✅ **Output Parsers**: String, Structured, Zod, List
✅ **LCEL**: Chains, Parallel, Branching, Streaming
✅ **Praktische Projekte**: RAG, Conversational RAG, Agents, Document Q&A

### LangChain Best Practices

- [ ] Prompt Engineering für bessere Ergebnisse
- [ ] Chunking-Strategie für Dokumente optimieren
- [ ] Vector Store für Produktivität persistent speichern
- [ ] Memory für lange Konversationen nutzen
- [ ] Agents mit klaren Tool-Beschreibungen
- [ ] Error Handling implementieren
- [ ] Kosten mit Token-Tracking monitoren
- [ ] Caching für wiederholte Queries

### Hilfreiche Ressourcen

- [LangChain Documentation](https://js.langchain.com/docs)
- [LangChain GitHub](https://github.com/langchain-ai/langchainjs)
- [OpenAI Cookbook](https://github.com/openai/openai-cookbook)
- [Pinecone Documentation](https://docs.pinecone.io/)

Viel Erfolg mit LangChain! 🚀
