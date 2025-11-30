# Prompt Chaining - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in Prompt Chaining](#einführung-in-prompt-chaining)
2. [Grundkonzepte](#grundkonzepte)
3. [Einfache Sequential Chains](#einfache-sequential-chains)
4. [Transformation Chains](#transformation-chains)
5. [Router Chains](#router-chains)
6. [Map-Reduce Patterns](#map-reduce-patterns)
7. [Conditional Chains](#conditional-chains)
8. [Error Handling & Fallbacks](#error-handling--fallbacks)
9. [Chain Optimization](#chain-optimization)
10. [Advanced Patterns](#advanced-patterns)
11. [Praktische Projekte](#praktische-projekte)

---

## Einführung in Prompt Chaining

### Was ist Prompt Chaining?

**Prompt Chaining** bedeutet, mehrere LLM-Aufrufe sequenziell zu verketten, wobei der Output eines Schrittes als Input für den nächsten dient.

### Vorteile

**Modularität** 🔧
- Wiederverwendbare Komponenten
- Einfachere Wartung
- Bessere Testbarkeit

**Komplexität** 🎯
- Lösung komplexer Aufgaben
- Schrittweise Verarbeitung
- Spezialisierte Prompts

**Qualität** ✨
- Bessere Ergebnisse durch Fokussierung
- Validierung zwischen Schritten
- Fehlerbehandlung

### Use Cases

```
Content Creation Pipeline:
  Research → Outline → Draft → Edit → Format

Data Processing:
  Extract → Transform → Validate → Summarize

Customer Support:
  Classify → Retrieve Context → Generate Response → Quality Check
```

---

## Grundkonzepte

### Einfaches Chain Pattern

```typescript
import { ChatOpenAI } from '@langchain/openai';
import { PromptTemplate } from '@langchain/core/prompts';
import { StringOutputParser } from '@langchain/core/output_parsers';

const model = new ChatOpenAI({ modelName: 'gpt-4' });

// Step 1: Generiere Thema
const topicPrompt = PromptTemplate.fromTemplate(
  'Generate a creative topic for a {genre} story.'
);

const topicChain = topicPrompt.pipe(model).pipe(new StringOutputParser());

// Step 2: Schreibe Story
const storyPrompt = PromptTemplate.fromTemplate(
  'Write a short story about: {topic}'
);

const storyChain = storyPrompt.pipe(model).pipe(new StringOutputParser());

// Verkettung
const topic = await topicChain.invoke({ genre: 'sci-fi' });
const story = await storyChain.invoke({ topic });

console.log('Topic:', topic);
console.log('Story:', story);
```

### LCEL Chain Composition

```typescript
import { RunnableSequence } from '@langchain/core/runnables';

// Moderne Verkettung mit LCEL
const composedChain = RunnableSequence.from([
  topicPrompt,
  model,
  new StringOutputParser(),
  { topic: (text: string) => text },
  storyPrompt,
  model,
  new StringOutputParser()
]);

const result = await composedChain.invoke({ genre: 'fantasy' });
```

### Chain mit Zwischenspeicherung

```typescript
interface ChainState {
  topic?: string;
  outline?: string;
  draft?: string;
  edited?: string;
}

async function contentPipeline(genre: string): Promise<ChainState> {
  const state: ChainState = {};
  
  // Step 1: Topic
  state.topic = await topicChain.invoke({ genre });
  console.log('✓ Topic generated');
  
  // Step 2: Outline
  const outlineChain = PromptTemplate.fromTemplate(
    'Create an outline for: {topic}'
  ).pipe(model).pipe(new StringOutputParser());
  
  state.outline = await outlineChain.invoke({ topic: state.topic });
  console.log('✓ Outline created');
  
  // Step 3: Draft
  const draftChain = PromptTemplate.fromTemplate(
    'Write based on this outline:\n{outline}'
  ).pipe(model).pipe(new StringOutputParser());
  
  state.draft = await draftChain.invoke({ outline: state.outline });
  console.log('✓ Draft written');
  
  return state;
}

const result = await contentPipeline('mystery');
```

---

## Einfache Sequential Chains

### Multi-Step Processing

```typescript
// Blog Post Pipeline
async function createBlogPost(topic: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Step 1: Research
  const researchPrompt = PromptTemplate.fromTemplate(
    'List 5 key points about: {topic}'
  );
  
  const keyPoints = await researchPrompt
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ topic });
  
  // Step 2: Outline
  const outlinePrompt = PromptTemplate.fromTemplate(
    'Create a blog outline using these points:\n{keyPoints}'
  );
  
  const outline = await outlinePrompt
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ keyPoints });
  
  // Step 3: Write
  const writePrompt = PromptTemplate.fromTemplate(
    'Write a blog post following this outline:\n{outline}'
  );
  
  const blogPost = await writePrompt
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ outline });
  
  // Step 4: Add SEO
  const seoPrompt = PromptTemplate.fromTemplate(
    'Add SEO meta description and tags for:\n{blogPost}'
  );
  
  const withSEO = await seoPrompt
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ blogPost });
  
  return withSEO;
}
```

### Reusable Chain Components

```typescript
class ChainBuilder {
  private model: ChatOpenAI;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  createStep(template: string) {
    return PromptTemplate.fromTemplate(template)
      .pipe(this.model)
      .pipe(new StringOutputParser());
  }
  
  async executeSequence(steps: Array<{ prompt: string; input?: any }>) {
    let result = steps[0].input || {};
    
    for (const step of steps) {
      const chain = this.createStep(step.prompt);
      result = await chain.invoke(result);
      result = { ...result, output: result };
    }
    
    return result;
  }
}

// Verwendung
const builder = new ChainBuilder();

const result = await builder.executeSequence([
  { prompt: 'Generate topic about {subject}', input: { subject: 'AI' } },
  { prompt: 'Create outline for: {output}' },
  { prompt: 'Write article from: {output}' }
]);
```

---

## Transformation Chains

### Data Extraction Pipeline

```typescript
import { z } from 'zod';
import { StructuredOutputParser } from '@langchain/core/output_parsers';

// Schema für strukturierte Daten
const personSchema = z.object({
  name: z.string(),
  age: z.number(),
  email: z.string(),
  occupation: z.string()
});

const parser = StructuredOutputParser.fromZodSchema(personSchema);

async function extractAndEnrich(text: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Step 1: Extract
  const extractPrompt = PromptTemplate.fromTemplate(
    `Extract person information:
    
    {format_instructions}
    
    Text: {text}`
  );
  
  const personData = await extractPrompt
    .pipe(model)
    .pipe(parser)
    .invoke({
      text,
      format_instructions: parser.getFormatInstructions()
    });
  
  // Step 2: Enrich
  const enrichPrompt = PromptTemplate.fromTemplate(
    `Based on this person data: {data}
    
    Suggest 3 career development opportunities.`
  );
  
  const suggestions = await enrichPrompt
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ data: JSON.stringify(personData) });
  
  return {
    person: personData,
    suggestions
  };
}
```

### Translation Pipeline

```typescript
async function translateAndAdapt(text: string, targetLanguage: string, targetAudience: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Step 1: Translate
  const translated = await PromptTemplate.fromTemplate(
    'Translate to {language}: {text}'
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ text, language: targetLanguage });
  
  // Step 2: Cultural Adaptation
  const adapted = await PromptTemplate.fromTemplate(
    'Adapt this text for {audience} audience, keeping cultural nuances:\n{text}'
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ text: translated, audience: targetAudience });
  
  // Step 3: Quality Check
  const quality = await PromptTemplate.fromTemplate(
    'Rate translation quality (1-10) and suggest improvements:\n{text}'
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ text: adapted });
  
  return { translated, adapted, quality };
}
```

---

## Router Chains

### Dynamic Routing

```typescript
import { RunnableBranch } from '@langchain/core/runnables';

// Classification Chain
const classifyPrompt = PromptTemplate.fromTemplate(
  'Classify this query as: technical, sales, support, or general\nQuery: {query}\nCategory:'
);

const classifyChain = classifyPrompt
  .pipe(model)
  .pipe(new StringOutputParser());

// Specialized Chains
const technicalChain = PromptTemplate.fromTemplate(
  'Technical response for: {query}'
).pipe(model).pipe(new StringOutputParser());

const salesChain = PromptTemplate.fromTemplate(
  'Sales response for: {query}'
).pipe(model).pipe(new StringOutputParser());

const supportChain = PromptTemplate.fromTemplate(
  'Support response for: {query}'
).pipe(model).pipe(new StringOutputParser());

// Router
async function routeQuery(query: string) {
  const category = (await classifyChain.invoke({ query })).toLowerCase().trim();
  
  console.log('Routed to:', category);
  
  switch (category) {
    case 'technical':
      return await technicalChain.invoke({ query });
    case 'sales':
      return await salesChain.invoke({ query });
    case 'support':
      return await supportChain.invoke({ query });
    default:
      return 'Unable to process query';
  }
}
```

### Multi-Expert System

```typescript
interface Expert {
  name: string;
  specialty: string;
  chain: any;
}

class ExpertRouter {
  private experts: Expert[] = [];
  private model: ChatOpenAI;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  addExpert(name: string, specialty: string, promptTemplate: string) {
    const chain = PromptTemplate.fromTemplate(promptTemplate)
      .pipe(this.model)
      .pipe(new StringOutputParser());
    
    this.experts.push({ name, specialty, chain });
  }
  
  async route(query: string) {
    // Select best expert
    const expertNames = this.experts.map(e => `${e.name} (${e.specialty})`).join(', ');
    
    const selectionPrompt = PromptTemplate.fromTemplate(
      `Select the best expert for this query:
      Experts: {experts}
      Query: {query}
      
      Respond with only the expert name.`
    );
    
    const selected = await selectionPrompt
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ query, experts: expertNames });
    
    // Find and execute
    const expert = this.experts.find(e => selected.includes(e.name));
    
    if (expert) {
      console.log(`Routing to: ${expert.name}`);
      return await expert.chain.invoke({ query });
    }
    
    return 'No suitable expert found';
  }
}

// Setup
const router = new ExpertRouter();

router.addExpert(
  'CodeMaster',
  'Programming',
  'As a programming expert, answer: {query}'
);

router.addExpert(
  'DataGuru',
  'Data Science',
  'As a data science expert, answer: {query}'
);

router.addExpert(
  'CloudNinja',
  'Cloud Architecture',
  'As a cloud architecture expert, answer: {query}'
);

// Usage
const answer = await router.route('How do I optimize a SQL query?');
```

---

## Map-Reduce Patterns

### Parallel Processing

```typescript
import { RunnableParallel } from '@langchain/core/runnables';

async function analyzeFromMultiplePerspectives(text: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Map Phase: Parallel Analysis
  const analyses = await RunnableParallel.from({
    sentiment: PromptTemplate.fromTemplate(
      'Analyze sentiment: {text}'
    ).pipe(model).pipe(new StringOutputParser()),
    
    keywords: PromptTemplate.fromTemplate(
      'Extract 5 keywords from: {text}'
    ).pipe(model).pipe(new StringOutputParser()),
    
    summary: PromptTemplate.fromTemplate(
      'Summarize in 2 sentences: {text}'
    ).pipe(model).pipe(new StringOutputParser()),
    
    tone: PromptTemplate.fromTemplate(
      'Describe the tone: {text}'
    ).pipe(model).pipe(new StringOutputParser())
  }).invoke({ text });
  
  // Reduce Phase: Combine Results
  const combinedPrompt = PromptTemplate.fromTemplate(
    `Create comprehensive analysis report:
    
    Sentiment: {sentiment}
    Keywords: {keywords}
    Summary: {summary}
    Tone: {tone}
    
    Write a unified analysis.`
  );
  
  const finalReport = await combinedPrompt
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke(analyses);
  
  return { analyses, finalReport };
}
```

### Document Processing

```typescript
async function processLargeDocument(document: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Split document into chunks
  const chunks = document.match(/.{1,2000}/g) || [];
  
  // Map: Summarize each chunk
  const summaries = await Promise.all(
    chunks.map(chunk =>
      PromptTemplate.fromTemplate('Summarize: {chunk}')
        .pipe(model)
        .pipe(new StringOutputParser())
        .invoke({ chunk })
    )
  );
  
  // Reduce: Combine summaries
  const finalSummary = await PromptTemplate.fromTemplate(
    'Combine these summaries into one coherent summary:\n{summaries}'
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ summaries: summaries.join('\n\n') });
  
  return finalSummary;
}
```

---

## Conditional Chains

### Validation Chain

```typescript
async function generateWithValidation(prompt: string, maxRetries = 3) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    console.log(`Attempt ${attempt}/${maxRetries}`);
    
    // Generate
    const response = await PromptTemplate.fromTemplate(prompt)
      .pipe(model)
      .pipe(new StringOutputParser())
      .invoke({});
    
    // Validate
    const isValid = await PromptTemplate.fromTemplate(
      `Is this response valid and complete? Answer only 'yes' or 'no':
      {response}`
    )
      .pipe(model)
      .pipe(new StringOutputParser())
      .invoke({ response });
    
    if (isValid.toLowerCase().includes('yes')) {
      console.log('✓ Validation passed');
      return response;
    }
    
    console.log('✗ Validation failed, retrying...');
  }
  
  throw new Error('Max retries reached');
}
```

### Self-Correction Chain

```typescript
async function generateWithSelfCorrection(task: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Initial Generation
  let response = await PromptTemplate.fromTemplate(
    'Complete this task: {task}'
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ task });
  
  // Self-Review
  const review = await PromptTemplate.fromTemplate(
    `Review this response and list any issues:
    Task: {task}
    Response: {response}
    
    Issues (or 'none' if perfect):`
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ task, response });
  
  // Correction if needed
  if (!review.toLowerCase().includes('none')) {
    console.log('Issues found:', review);
    
    response = await PromptTemplate.fromTemplate(
      `Improve this response:
      Original: {response}
      Issues: {review}
      
      Improved version:`
    )
      .pipe(model)
      .pipe(new StringOutputParser())
      .invoke({ response, review });
  }
  
  return response;
}
```

---

## Error Handling & Fallbacks

### Try-Catch Chain

```typescript
async function robustChain(input: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  try {
    // Primary chain
    const result = await PromptTemplate.fromTemplate(
      'Complex analysis: {input}'
    )
      .pipe(model)
      .pipe(new StringOutputParser())
      .invoke({ input });
    
    return result;
    
  } catch (error) {
    console.error('Primary chain failed:', error);
    
    // Fallback chain
    return await PromptTemplate.fromTemplate(
      'Simple analysis: {input}'
    )
      .pipe(model)
      .pipe(new StringOutputParser())
      .invoke({ input });
  }
}
```

### Retry with Exponential Backoff

```typescript
async function chainWithRetry(
  chain: any,
  input: any,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await chain.invoke(input);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${maxRetries} in ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

---

## Chain Optimization

### Caching

```typescript
class CachedChain {
  private cache = new Map<string, string>();
  private chain: any;
  
  constructor(chain: any) {
    this.chain = chain;
  }
  
  async invoke(input: any): Promise<string> {
    const key = JSON.stringify(input);
    
    if (this.cache.has(key)) {
      console.log('Cache hit');
      return this.cache.get(key)!;
    }
    
    console.log('Cache miss');
    const result = await this.chain.invoke(input);
    this.cache.set(key, result);
    
    return result;
  }
  
  clearCache() {
    this.cache.clear();
  }
}
```

### Parallel Optimization

```typescript
async function optimizedPipeline(texts: string[]) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Process all in parallel
  const results = await Promise.all(
    texts.map(text =>
      PromptTemplate.fromTemplate('Process: {text}')
        .pipe(model)
        .pipe(new StringOutputParser())
        .invoke({ text })
    )
  );
  
  return results;
}
```

---

## Advanced Patterns

### Hierarchical Chain

```typescript
async function hierarchicalProcessing(document: string) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  
  // Level 1: Section summaries
  const sections = document.split('\n\n');
  const sectionSummaries = await Promise.all(
    sections.map(section =>
      PromptTemplate.fromTemplate('Summarize section: {section}')
        .pipe(model)
        .pipe(new StringOutputParser())
        .invoke({ section })
    )
  );
  
  // Level 2: Chapter summary
  const chapterSummary = await PromptTemplate.fromTemplate(
    'Combine section summaries:\n{summaries}'
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ summaries: sectionSummaries.join('\n') });
  
  // Level 3: Document summary
  const documentSummary = await PromptTemplate.fromTemplate(
    'Create executive summary from:\n{chapter}'
  )
    .pipe(model)
    .pipe(new StringOutputParser())
    .invoke({ chapter: chapterSummary });
  
  return {
    sectionSummaries,
    chapterSummary,
    documentSummary
  };
}
```

### Feedback Loop Chain

```typescript
async function iterativeImprovement(initialContent: string, iterations = 3) {
  const model = new ChatOpenAI({ modelName: 'gpt-4' });
  let content = initialContent;
  
  for (let i = 0; i < iterations; i++) {
    console.log(`Iteration ${i + 1}/${iterations}`);
    
    // Critique
    const critique = await PromptTemplate.fromTemplate(
      'Critique this content and suggest improvements:\n{content}'
    )
      .pipe(model)
      .pipe(new StringOutputParser())
      .invoke({ content });
    
    // Improve
    content = await PromptTemplate.fromTemplate(
      `Improve based on critique:
      Content: {content}
      Critique: {critique}`
    )
      .pipe(model)
      .pipe(new StringOutputParser())
      .invoke({ content, critique });
  }
  
  return content;
}
```

---

## Praktische Projekte

### Projekt 1: Content Creation Pipeline

```typescript
class ContentCreationPipeline {
  private model: ChatOpenAI;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  async create(topic: string, targetAudience: string) {
    console.log('Starting pipeline...');
    
    // Step 1: Research
    const research = await this.research(topic);
    console.log('✓ Research complete');
    
    // Step 2: Outline
    const outline = await this.createOutline(research, targetAudience);
    console.log('✓ Outline created');
    
    // Step 3: Write
    const draft = await this.writeDraft(outline);
    console.log('✓ Draft written');
    
    // Step 4: Edit
    const edited = await this.edit(draft);
    console.log('✓ Edited');
    
    // Step 5: SEO
    const withSEO = await this.addSEO(edited, topic);
    console.log('✓ SEO added');
    
    return {
      research,
      outline,
      draft,
      edited,
      final: withSEO
    };
  }
  
  private async research(topic: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      'Research and list 10 key facts about: {topic}'
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ topic });
  }
  
  private async createOutline(research: string, audience: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      `Create article outline for {audience}:
      Research: {research}`
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ research, audience });
  }
  
  private async writeDraft(outline: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      'Write full article from outline:\n{outline}'
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ outline });
  }
  
  private async edit(draft: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      'Edit for clarity and flow:\n{draft}'
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ draft });
  }
  
  private async addSEO(content: string, topic: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      `Add SEO elements (title, meta, keywords):
      Topic: {topic}
      Content: {content}`
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ content, topic });
  }
}

// Verwendung
const pipeline = new ContentCreationPipeline();
const result = await pipeline.create('AI in Healthcare', 'medical professionals');
```

### Projekt 2: Customer Support Bot

```typescript
class CustomerSupportBot {
  private model: ChatOpenAI;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  async handleQuery(query: string) {
    // Step 1: Classify
    const category = await this.classifyQuery(query);
    console.log('Category:', category);
    
    // Step 2: Retrieve context
    const context = await this.retrieveContext(category);
    
    // Step 3: Generate response
    const response = await this.generateResponse(query, context);
    
    // Step 4: Quality check
    const quality = await this.qualityCheck(query, response);
    
    if (quality.score < 7) {
      // Regenerate if quality is low
      return await this.generateResponse(query, context + '\n' + quality.feedback);
    }
    
    return response;
  }
  
  private async classifyQuery(query: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      `Classify as: billing, technical, account, shipping, or general
      Query: {query}
      Category:`
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ query });
  }
  
  private async retrieveContext(category: string): Promise<string> {
    // Simuliere Kontext-Retrieval
    const contexts = {
      billing: 'Billing policies...',
      technical: 'Technical documentation...',
      account: 'Account management info...',
      shipping: 'Shipping guidelines...',
      general: 'General information...'
    };
    
    return contexts[category.toLowerCase()] || contexts.general;
  }
  
  private async generateResponse(query: string, context: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      `Answer the query using this context:
      
      Context: {context}
      
      Query: {query}
      
      Provide helpful, professional response.`
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ query, context });
  }
  
  private async qualityCheck(query: string, response: string): Promise<{ score: number; feedback: string }> {
    const check = await PromptTemplate.fromTemplate(
      `Rate response quality (1-10) and provide feedback:
      
      Query: {query}
      Response: {response}
      
      Format: Score: X, Feedback: ...`
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ query, response });
    
    const scoreMatch = check.match(/Score:\s*(\d+)/);
    const score = scoreMatch ? parseInt(scoreMatch[1]) : 5;
    
    return { score, feedback: check };
  }
}
```

### Projekt 3: Code Review Pipeline

```typescript
class CodeReviewPipeline {
  private model: ChatOpenAI;
  
  constructor() {
    this.model = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  async review(code: string, language: string) {
    console.log('Starting code review...');
    
    // Parallel analysis
    const analyses = await RunnableParallel.from({
      bugs: this.analyzeBugs(code, language),
      security: this.analyzeSecurity(code, language),
      performance: this.analyzePerformance(code, language),
      style: this.analyzeStyle(code, language)
    }).invoke({});
    
    // Generate summary
    const summary = await this.generateSummary(analyses);
    
    // Suggest improvements
    const improvements = await this.suggestImprovements(code, summary);
    
    return {
      analyses,
      summary,
      improvements
    };
  }
  
  private analyzeBugs(code: string, language: string) {
    return PromptTemplate.fromTemplate(
      'Find potential bugs in this {language} code:\n{code}'
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ code, language });
  }
  
  private analyzeSecurity(code: string, language: string) {
    return PromptTemplate.fromTemplate(
      'Identify security issues in this {language} code:\n{code}'
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ code, language });
  }
  
  private analyzePerformance(code: string, language: string) {
    return PromptTemplate.fromTemplate(
      'Analyze performance of this {language} code:\n{code}'
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ code, language });
  }
  
  private analyzeStyle(code: string, language: string) {
    return PromptTemplate.fromTemplate(
      'Review code style and best practices for {language}:\n{code}'
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ code, language });
  }
  
  private async generateSummary(analyses: any): Promise<string> {
    return await PromptTemplate.fromTemplate(
      `Create code review summary:
      
      Bugs: {bugs}
      Security: {security}
      Performance: {performance}
      Style: {style}
      
      Provide overall assessment.`
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke(analyses);
  }
  
  private async suggestImprovements(code: string, summary: string): Promise<string> {
    return await PromptTemplate.fromTemplate(
      `Suggest specific code improvements:
      
      Original Code: {code}
      Review Summary: {summary}
      
      Provide improved version with explanations.`
    )
      .pipe(this.model)
      .pipe(new StringOutputParser())
      .invoke({ code, summary });
  }
}
```

---

## Zusammenfassung

Du hast nun gelernt:

✅ **Grundkonzepte**: Sequential Chains, LCEL Composition
✅ **Transformation Chains**: Data Extraction, Translation Pipelines
✅ **Router Chains**: Dynamic Routing, Multi-Expert Systems
✅ **Map-Reduce**: Parallel Processing, Document Analysis
✅ **Conditional Chains**: Validation, Self-Correction
✅ **Error Handling**: Try-Catch, Retry Logic, Fallbacks
✅ **Optimization**: Caching, Parallel Execution
✅ **Advanced Patterns**: Hierarchical Processing, Feedback Loops
✅ **Praktische Projekte**: Content Pipeline, Support Bot, Code Review

### Best Practices

- [ ] Kleine, fokussierte Chains statt monolithischer Prompts
- [ ] Validierung zwischen Chain-Schritten
- [ ] Error Handling und Fallback-Strategien
- [ ] Caching für wiederholte Operationen
- [ ] Logging für Debugging und Monitoring
- [ ] Parallelisierung wo möglich
- [ ] Type-Safety mit TypeScript/Zod

### Hilfreiche Ressourcen

- [LangChain Chains](https://js.langchain.com/docs/modules/chains)
- [LCEL Documentation](https://js.langchain.com/docs/expression_language)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)

Viel Erfolg mit Prompt Chaining! 🔗
