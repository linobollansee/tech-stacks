# Agents & Memory - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in Agents](#einführung-in-agents)
2. [Agent-Typen](#agent-typen)
3. [Tools & Tool Calling](#tools--tool-calling)
4. [Agent Execution](#agent-execution)
5. [Memory-Systeme](#memory-systeme)
6. [Conversation Memory](#conversation-memory)
7. [Entity Memory](#entity-memory)
8. [Vector Memory](#vector-memory)
9. [Advanced Agent Patterns](#advanced-agent-patterns)
10. [Multi-Agent Systems](#multi-agent-systems)
11. [Praktische Projekte](#praktische-projekte)

---

## Einführung in Agents

### Was sind Agents?

**Agents** sind LLM-basierte Systeme, die autonom Entscheidungen treffen, Tools verwenden und Aufgaben lösen können.

### Agent-Komponenten

```
Agent
  ├─ LLM           → Reasoning Engine
  ├─ Tools         → Externe Funktionen
  ├─ Memory        → Kontext speichern
  └─ Executor      → Action Loop
```

### ReAct Pattern

```
Thought: Ich muss die Temperatur herausfinden
Action: weather_tool
Action Input: "Berlin"
Observation: 22°C, Sunny

Thought: Ich habe die Antwort
Final Answer: Es sind 22°C in Berlin
```

### Einfacher Agent

```typescript
import { ChatOpenAI } from '@langchain/openai';
import { initializeAgentExecutorWithOptions } from 'langchain/agents';
import { Calculator } from '@langchain/community/tools/calculator';

const model = new ChatOpenAI({
  modelName: 'gpt-4',
  temperature: 0
});

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

---

## Agent-Typen

### Zero-Shot React Agent

```typescript
import { initializeAgentExecutorWithOptions } from 'langchain/agents';

// Entscheidet ohne Beispiele
const agent = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'zero-shot-react-description',
  maxIterations: 5,
  verbose: true
});

const result = await agent.invoke({
  input: 'Search for the population of Berlin and multiply it by 2'
});
```

### Structured Chat Agent

```typescript
import { ChatOpenAI } from '@langchain/openai';
import { initializeAgentExecutorWithOptions } from 'langchain/agents';

// Besser für strukturierte Inputs
const agent = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'structured-chat-zero-shot-react-description',
  verbose: true
});

const result = await agent.invoke({
  input: 'Find weather in Berlin and calculate if 22°C is warmer than 20°C'
});
```

### OpenAI Functions Agent

```typescript
import { initializeAgentExecutorWithOptions } from 'langchain/agents';

// Nutzt OpenAI Function Calling
const agent = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'openai-functions',
  verbose: true
});

// Präzisere Tool-Selection
const result = await agent.invoke({
  input: 'Get user info for ID 123 and send them an email'
});
```

### Conversational Agent

```typescript
import { BufferMemory } from 'langchain/memory';

const memory = new BufferMemory({
  returnMessages: true,
  memoryKey: 'chat_history'
});

const agent = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'chat-conversational-react-description',
  memory: memory,
  verbose: true
});

// Erinnert sich an Kontext
await agent.invoke({ input: 'My name is Max' });
await agent.invoke({ input: 'What is my name?' });
```

---

## Tools & Tool Calling

### Custom Tool

```typescript
import { DynamicTool } from '@langchain/core/tools';

const weatherTool = new DynamicTool({
  name: 'get-weather',
  description: 'Get current weather for a city. Input should be a city name.',
  func: async (city: string) => {
    // API Call
    const response = await fetch(`https://api.weather.com/${city}`);
    const data = await response.json();
    return `Temperature: ${data.temp}°C, Conditions: ${data.conditions}`;
  }
});
```

### Structured Tool mit Zod

```typescript
import { StructuredTool } from '@langchain/core/tools';
import { z } from 'zod';

class UserSearchTool extends StructuredTool {
  name = 'search-users';
  description = 'Search for users in the database';
  
  schema = z.object({
    query: z.string().describe('Search query'),
    age: z.number().optional().describe('Filter by age'),
    role: z.enum(['admin', 'user', 'guest']).optional()
  });
  
  async _call(input: z.infer<typeof this.schema>): Promise<string> {
    const users = await searchDatabase(input);
    return JSON.stringify(users);
  }
}

const tool = new UserSearchTool();
```

### Tool mit Error Handling

```typescript
class SafeTool extends DynamicTool {
  constructor() {
    super({
      name: 'safe-api-call',
      description: 'Make API call with error handling',
      func: async (endpoint: string) => {
        try {
          const response = await fetch(endpoint);
          
          if (!response.ok) {
            return `Error: ${response.status} ${response.statusText}`;
          }
          
          const data = await response.json();
          return JSON.stringify(data);
          
        } catch (error) {
          return `Exception: ${error.message}`;
        }
      }
    });
  }
}
```

### Multi-Function Tool

```typescript
class DatabaseTool extends StructuredTool {
  name = 'database-operations';
  description = 'Perform database operations';
  
  schema = z.object({
    operation: z.enum(['read', 'write', 'delete']),
    table: z.string(),
    data: z.record(z.any()).optional()
  });
  
  async _call({ operation, table, data }: z.infer<typeof this.schema>) {
    switch (operation) {
      case 'read':
        return await this.read(table);
      case 'write':
        return await this.write(table, data);
      case 'delete':
        return await this.delete(table, data);
    }
  }
  
  private async read(table: string) {
    // Read implementation
    return `Read from ${table}`;
  }
  
  private async write(table: string, data: any) {
    // Write implementation
    return `Written to ${table}`;
  }
  
  private async delete(table: string, data: any) {
    // Delete implementation
    return `Deleted from ${table}`;
  }
}
```

---

## Agent Execution

### Custom Agent Loop

```typescript
class CustomAgent {
  private model: ChatOpenAI;
  private tools: Map<string, DynamicTool>;
  
  constructor(model: ChatOpenAI, tools: DynamicTool[]) {
    this.model = model;
    this.tools = new Map(tools.map(t => [t.name, t]));
  }
  
  async execute(input: string, maxIterations = 5) {
    let thought = input;
    let iteration = 0;
    
    while (iteration < maxIterations) {
      iteration++;
      console.log(`\n--- Iteration ${iteration} ---`);
      
      // Get action from LLM
      const action = await this.getAction(thought);
      
      if (action.type === 'final') {
        return action.output;
      }
      
      // Execute tool
      const observation = await this.executeTool(action);
      
      // Update thought
      thought = `Previous thought: ${thought}\n` +
                `Action: ${action.tool}\n` +
                `Observation: ${observation}`;
    }
    
    return 'Max iterations reached';
  }
  
  private async getAction(thought: string) {
    const toolDescriptions = Array.from(this.tools.values())
      .map(t => `${t.name}: ${t.description}`)
      .join('\n');
    
    const prompt = `You have access to these tools:
${toolDescriptions}

Current situation: ${thought}

Decide what to do next. Respond in format:
Thought: [your reasoning]
Action: [tool name or FINAL_ANSWER]
Action Input: [input for tool]`;

    const response = await this.model.invoke(prompt);
    
    // Parse response
    return this.parseAction(response.content as string);
  }
  
  private parseAction(response: string) {
    const actionMatch = response.match(/Action: (.+)/);
    const inputMatch = response.match(/Action Input: (.+)/);
    
    if (actionMatch?.[1] === 'FINAL_ANSWER') {
      return { type: 'final', output: inputMatch?.[1] || response };
    }
    
    return {
      type: 'tool',
      tool: actionMatch?.[1]?.trim() || '',
      input: inputMatch?.[1]?.trim() || ''
    };
  }
  
  private async executeTool(action: any): Promise<string> {
    const tool = this.tools.get(action.tool);
    
    if (!tool) {
      return `Tool ${action.tool} not found`;
    }
    
    try {
      return await tool.invoke(action.input);
    } catch (error) {
      return `Error executing tool: ${error.message}`;
    }
  }
}
```

### Agent with Streaming

```typescript
async function streamingAgent(input: string) {
  const executor = await initializeAgentExecutorWithOptions(tools, model, {
    agentType: 'openai-functions',
    returnIntermediateSteps: true
  });
  
  const stream = await executor.stream({ input });
  
  for await (const chunk of stream) {
    if (chunk.intermediateSteps) {
      console.log('Step:', chunk.intermediateSteps);
    }
    if (chunk.output) {
      console.log('Output:', chunk.output);
    }
  }
}
```

---

## Memory-Systeme

### Buffer Memory

```typescript
import { BufferMemory } from 'langchain/memory';
import { ConversationChain } from 'langchain/chains';

const memory = new BufferMemory({
  returnMessages: true,
  memoryKey: 'chat_history',
  inputKey: 'input',
  outputKey: 'output'
});

const chain = new ConversationChain({
  llm: model,
  memory: memory
});

await chain.invoke({ input: 'Hi, I am Max' });
await chain.invoke({ input: 'What is my name?' });

// Memory anzeigen
const context = await memory.loadMemoryVariables({});
console.log(context);
```

### Buffer Window Memory

```typescript
import { BufferWindowMemory } from 'langchain/memory';

// Nur letzte k Nachrichten
const memory = new BufferWindowMemory({
  k: 5,  // Letzte 5 Interaktionen
  returnMessages: true,
  memoryKey: 'chat_history'
});

const agent = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'chat-conversational-react-description',
  memory: memory
});
```

### Summary Memory

```typescript
import { ConversationSummaryMemory } from 'langchain/memory';

const memory = new ConversationSummaryMemory({
  llm: model,
  maxTokenLimit: 2000,
  returnMessages: true
});

// Fasst alte Konversationen zusammen
const chain = new ConversationChain({
  llm: model,
  memory: memory
});

// Lange Konversation
for (let i = 0; i < 10; i++) {
  await chain.invoke({ input: `Message ${i}` });
}

// Summary wird automatisch erstellt
const context = await memory.loadMemoryVariables({});
console.log(context.history);
```

---

## Conversation Memory

### Token Buffer Memory

```typescript
import { ConversationTokenBufferMemory } from 'langchain/memory';

const memory = new ConversationTokenBufferMemory({
  llm: model,
  maxTokenLimit: 500,  // Max 500 Tokens
  returnMessages: true
});

const agent = await initializeAgentExecutorWithOptions(tools, model, {
  agentType: 'chat-conversational-react-description',
  memory: memory
});
```

### Custom Memory

```typescript
import { BaseChatMemory } from 'langchain/memory';
import { AIMessage, HumanMessage } from '@langchain/core/messages';

class CustomMemory extends BaseChatMemory {
  private messages: Array<HumanMessage | AIMessage> = [];
  private importance: Map<number, number> = new Map();
  
  async saveContext(input: any, output: any) {
    this.messages.push(new HumanMessage(input.input));
    this.messages.push(new AIMessage(output.output));
    
    // Calculate importance
    const index = this.messages.length - 1;
    const importance = await this.calculateImportance(output.output);
    this.importance.set(index, importance);
  }
  
  async loadMemoryVariables() {
    // Return most important messages
    const sorted = Array.from(this.importance.entries())
      .sort((a, b) => b[1] - a[1])
      .slice(0, 10);
    
    const importantMessages = sorted.map(([index]) => this.messages[index]);
    
    return {
      chat_history: importantMessages
    };
  }
  
  private async calculateImportance(text: string): Promise<number> {
    // Simple importance score
    return text.length;
  }
}
```

---

## Entity Memory

### Entity Extraction & Storage

```typescript
import { EntityMemory } from 'langchain/memory';

const memory = new EntityMemory({
  llm: model,
  returnMessages: true
});

const chain = new ConversationChain({
  llm: model,
  memory: memory
});

await chain.invoke({
  input: 'Alice works at Google and lives in Berlin'
});

await chain.invoke({
  input: 'Where does Alice work?'
});

// Entities anzeigen
const entities = await memory.loadMemoryVariables({ input: 'Alice' });
console.log(entities);
```

### Knowledge Graph Memory

```typescript
class KnowledgeGraphMemory {
  private entities: Map<string, any> = new Map();
  private relationships: Array<{from: string; to: string; type: string}> = [];
  
  async addEntity(name: string, properties: any) {
    this.entities.set(name, properties);
  }
  
  async addRelationship(from: string, to: string, type: string) {
    this.relationships.push({ from, to, type });
  }
  
  async query(entityName: string) {
    const entity = this.entities.get(entityName);
    const relations = this.relationships.filter(
      r => r.from === entityName || r.to === entityName
    );
    
    return { entity, relations };
  }
  
  async extractFromText(text: string) {
    // Use LLM to extract entities and relationships
    const prompt = `Extract entities and relationships from:
${text}

Format:
ENTITY: name | property1: value1, property2: value2
RELATION: entity1 -> relationship -> entity2`;

    const response = await model.invoke(prompt);
    
    // Parse and store
    this.parseAndStore(response.content as string);
  }
  
  private parseAndStore(text: string) {
    // Implementation
  }
}
```

---

## Vector Memory

### Semantic Memory

```typescript
import { MemoryVectorStore } from 'langchain/vectorstores/memory';
import { OpenAIEmbeddings } from '@langchain/openai';

class SemanticMemory {
  private vectorStore: MemoryVectorStore;
  private embeddings: OpenAIEmbeddings;
  
  constructor() {
    this.embeddings = new OpenAIEmbeddings();
  }
  
  async initialize() {
    this.vectorStore = new MemoryVectorStore(this.embeddings);
  }
  
  async remember(text: string, metadata: any = {}) {
    await this.vectorStore.addDocuments([
      { pageContent: text, metadata }
    ]);
  }
  
  async recall(query: string, k = 5) {
    const results = await this.vectorStore.similaritySearch(query, k);
    return results.map(doc => ({
      content: doc.pageContent,
      metadata: doc.metadata
    }));
  }
}

// Verwendung
const memory = new SemanticMemory();
await memory.initialize();

await memory.remember('Alice is a software engineer', { entity: 'Alice' });
await memory.remember('Bob works in marketing', { entity: 'Bob' });

const results = await memory.recall('Who is a developer?');
console.log(results);
```

### Long-Term Memory

```typescript
class LongTermMemory {
  private shortTerm: BufferWindowMemory;
  private longTerm: MemoryVectorStore;
  private embeddings: OpenAIEmbeddings;
  
  constructor() {
    this.embeddings = new OpenAIEmbeddings();
    this.shortTerm = new BufferWindowMemory({ k: 10 });
  }
  
  async initialize() {
    this.longTerm = new MemoryVectorStore(this.embeddings);
  }
  
  async saveContext(input: string, output: string) {
    // Save to short-term
    await this.shortTerm.saveContext({ input }, { output });
    
    // Check if important enough for long-term
    const importance = await this.assessImportance(input, output);
    
    if (importance > 0.7) {
      await this.longTerm.addDocuments([
        { 
          pageContent: `Q: ${input}\nA: ${output}`,
          metadata: { timestamp: Date.now(), importance }
        }
      ]);
    }
  }
  
  async loadMemoryVariables(query: string) {
    // Combine short-term and relevant long-term
    const shortTermContext = await this.shortTerm.loadMemoryVariables({});
    const longTermContext = await this.longTerm.similaritySearch(query, 3);
    
    return {
      short_term: shortTermContext,
      long_term: longTermContext
    };
  }
  
  private async assessImportance(input: string, output: string): Promise<number> {
    // Simple heuristic
    const combinedLength = input.length + output.length;
    return Math.min(combinedLength / 500, 1.0);
  }
}
```

---

## Advanced Agent Patterns

### Plan-Execute Agent

```typescript
class PlanExecuteAgent {
  private model: ChatOpenAI;
  private tools: DynamicTool[];
  
  constructor(model: ChatOpenAI, tools: DynamicTool[]) {
    this.model = model;
    this.tools = tools;
  }
  
  async execute(goal: string) {
    // Step 1: Create plan
    const plan = await this.createPlan(goal);
    console.log('Plan:', plan);
    
    // Step 2: Execute plan
    const results = [];
    for (const step of plan.steps) {
      console.log(`Executing: ${step}`);
      const result = await this.executeStep(step);
      results.push(result);
    }
    
    // Step 3: Synthesize results
    return await this.synthesize(goal, results);
  }
  
  private async createPlan(goal: string) {
    const prompt = `Create a step-by-step plan to achieve:
Goal: ${goal}

Available tools: ${this.tools.map(t => t.name).join(', ')}

Respond with numbered steps.`;

    const response = await this.model.invoke(prompt);
    
    // Parse steps
    const steps = (response.content as string)
      .split('\n')
      .filter(line => /^\d+\./.test(line))
      .map(line => line.replace(/^\d+\.\s*/, ''));
    
    return { steps };
  }
  
  private async executeStep(step: string) {
    // Use agent to execute individual step
    const executor = await initializeAgentExecutorWithOptions(
      this.tools,
      this.model,
      { agentType: 'openai-functions' }
    );
    
    return await executor.invoke({ input: step });
  }
  
  private async synthesize(goal: string, results: any[]) {
    const prompt = `Synthesize these results into a final answer:
    
Goal: ${goal}

Results:
${results.map((r, i) => `${i + 1}. ${r.output}`).join('\n')}

Final Answer:`;

    const response = await this.model.invoke(prompt);
    return response.content;
  }
}
```

### ReAct Agent with Reflection

```typescript
class ReflectiveAgent {
  private model: ChatOpenAI;
  private tools: DynamicTool[];
  
  async execute(input: string) {
    let attempt = 1;
    const maxAttempts = 3;
    
    while (attempt <= maxAttempts) {
      console.log(`\nAttempt ${attempt}/${maxAttempts}`);
      
      // Execute
      const result = await this.executeWithReflection(input);
      
      // Reflect
      const reflection = await this.reflect(input, result);
      
      if (reflection.satisfied) {
        return result;
      }
      
      console.log('Reflection:', reflection.feedback);
      input = `${input}\nPrevious attempt: ${result}\nFeedback: ${reflection.feedback}`;
      attempt++;
    }
    
    return 'Could not satisfy requirements after max attempts';
  }
  
  private async executeWithReflection(input: string) {
    const executor = await initializeAgentExecutorWithOptions(
      this.tools,
      this.model,
      { agentType: 'openai-functions' }
    );
    
    const result = await executor.invoke({ input });
    return result.output;
  }
  
  private async reflect(original: string, result: string) {
    const prompt = `Reflect on this result:
    
Original request: ${original}
Result: ${result}

Is this result satisfactory? Respond in format:
Satisfied: yes/no
Feedback: [your feedback]`;

    const response = await this.model.invoke(prompt);
    const content = response.content as string;
    
    return {
      satisfied: content.toLowerCase().includes('satisfied: yes'),
      feedback: content.match(/Feedback: (.+)/)?.[1] || ''
    };
  }
}
```

---

## Multi-Agent Systems

### Agent Collaboration

```typescript
class AgentTeam {
  private agents: Map<string, any> = new Map();
  private coordinator: ChatOpenAI;
  
  constructor() {
    this.coordinator = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  addAgent(name: string, agent: any, specialty: string) {
    this.agents.set(name, { agent, specialty });
  }
  
  async solve(problem: string) {
    // Step 1: Coordinator decides which agents to use
    const plan = await this.coordinatorPlan(problem);
    
    // Step 2: Execute with selected agents
    const results = [];
    for (const agentName of plan.agents) {
      const agentInfo = this.agents.get(agentName);
      if (agentInfo) {
        console.log(`Consulting ${agentName}...`);
        const result = await agentInfo.agent.invoke({ input: problem });
        results.push({ agent: agentName, result: result.output });
      }
    }
    
    // Step 3: Synthesize
    return await this.synthesizeResults(problem, results);
  }
  
  private async coordinatorPlan(problem: string) {
    const agentList = Array.from(this.agents.entries())
      .map(([name, info]) => `${name}: ${info.specialty}`)
      .join('\n');
    
    const prompt = `You are coordinating a team of agents.
    
Available agents:
${agentList}

Problem: ${problem}

Which agents should work on this? List their names.`;

    const response = await this.coordinator.invoke(prompt);
    
    // Parse agent names
    const agents = Array.from(this.agents.keys()).filter(name =>
      (response.content as string).toLowerCase().includes(name.toLowerCase())
    );
    
    return { agents };
  }
  
  private async synthesizeResults(problem: string, results: any[]) {
    const resultsText = results
      .map(r => `${r.agent}: ${r.result}`)
      .join('\n\n');
    
    const prompt = `Synthesize these expert opinions:

Problem: ${problem}

Opinions:
${resultsText}

Final answer:`;

    const response = await this.coordinator.invoke(prompt);
    return response.content;
  }
}

// Verwendung
const team = new AgentTeam();

const researchAgent = await initializeAgentExecutorWithOptions(
  [searchTool],
  model,
  { agentType: 'openai-functions' }
);

const calculationAgent = await initializeAgentExecutorWithOptions(
  [new Calculator()],
  model,
  { agentType: 'openai-functions' }
);

team.addAgent('Researcher', researchAgent, 'Finding information');
team.addAgent('Calculator', calculationAgent, 'Mathematical calculations');

const result = await team.solve('Find the population of Berlin and calculate if it doubled in 20 years');
```

### Hierarchical Agents

```typescript
class HierarchicalAgentSystem {
  private supervisor: ChatOpenAI;
  private workers: Map<string, any> = new Map();
  
  constructor() {
    this.supervisor = new ChatOpenAI({ modelName: 'gpt-4' });
  }
  
  addWorker(name: string, tools: DynamicTool[], specialty: string) {
    this.workers.set(name, { tools, specialty });
  }
  
  async execute(task: string) {
    // Supervisor breaks down task
    const subtasks = await this.breakDownTask(task);
    
    // Assign subtasks to workers
    const results = [];
    for (const subtask of subtasks) {
      const worker = await this.assignWorker(subtask);
      console.log(`Assigning to ${worker}: ${subtask}`);
      
      const result = await this.executeSubtask(worker, subtask);
      results.push({ subtask, result });
    }
    
    // Supervisor combines results
    return await this.combineResults(task, results);
  }
  
  private async breakDownTask(task: string) {
    const prompt = `Break this task into subtasks:
Task: ${task}

List subtasks:`;

    const response = await this.supervisor.invoke(prompt);
    
    return (response.content as string)
      .split('\n')
      .filter(line => line.trim().length > 0);
  }
  
  private async assignWorker(subtask: string): Promise<string> {
    const workerList = Array.from(this.workers.entries())
      .map(([name, info]) => `${name}: ${info.specialty}`)
      .join('\n');
    
    const prompt = `Assign this subtask to a worker:

Workers:
${workerList}

Subtask: ${subtask}

Best worker:`;

    const response = await this.supervisor.invoke(prompt);
    
    // Find matching worker
    for (const name of this.workers.keys()) {
      if ((response.content as string).toLowerCase().includes(name.toLowerCase())) {
        return name;
      }
    }
    
    return Array.from(this.workers.keys())[0];
  }
  
  private async executeSubtask(workerName: string, subtask: string) {
    const worker = this.workers.get(workerName);
    
    const executor = await initializeAgentExecutorWithOptions(
      worker.tools,
      this.supervisor,
      { agentType: 'openai-functions' }
    );
    
    const result = await executor.invoke({ input: subtask });
    return result.output;
  }
  
  private async combineResults(task: string, results: any[]) {
    const resultsText = results
      .map(r => `Subtask: ${r.subtask}\nResult: ${r.result}`)
      .join('\n\n');
    
    const prompt = `Combine these results:

Original task: ${task}

${resultsText}

Final answer:`;

    const response = await this.supervisor.invoke(prompt);
    return response.content;
  }
}
```

---

## Praktische Projekte

### Projekt 1: Personal Assistant Agent

```typescript
class PersonalAssistant {
  private agent: any;
  private memory: LongTermMemory;
  
  async initialize() {
    this.memory = new LongTermMemory();
    await this.memory.initialize();
    
    // Setup tools
    const tools = [
      new DynamicTool({
        name: 'calendar',
        description: 'Manage calendar events',
        func: async (input) => this.manageCalendar(input)
      }),
      new DynamicTool({
        name: 'email',
        description: 'Send emails',
        func: async (input) => this.sendEmail(input)
      }),
      new DynamicTool({
        name: 'reminders',
        description: 'Set reminders',
        func: async (input) => this.setReminder(input)
      }),
      new DynamicTool({
        name: 'search',
        description: 'Search the web',
        func: async (query) => this.search(query)
      })
    ];
    
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    
    this.agent = await initializeAgentExecutorWithOptions(tools, model, {
      agentType: 'openai-functions',
      verbose: true
    });
  }
  
  async chat(message: string) {
    // Load relevant context
    const context = await this.memory.loadMemoryVariables(message);
    
    // Execute with context
    const result = await this.agent.invoke({
      input: `Context: ${JSON.stringify(context)}\n\nUser: ${message}`
    });
    
    // Save to memory
    await this.memory.saveContext(message, result.output);
    
    return result.output;
  }
  
  private async manageCalendar(input: string) {
    // Calendar API integration
    return 'Event added to calendar';
  }
  
  private async sendEmail(input: string) {
    // Email API integration
    return 'Email sent';
  }
  
  private async setReminder(input: string) {
    // Reminder system integration
    return 'Reminder set';
  }
  
  private async search(query: string) {
    // Search API integration
    return 'Search results...';
  }
}

// Verwendung
const assistant = new PersonalAssistant();
await assistant.initialize();

console.log(await assistant.chat('Schedule a meeting with Alice tomorrow at 2pm'));
console.log(await assistant.chat('Send her an email about it'));
console.log(await assistant.chat('What meetings do I have tomorrow?'));
```

### Projekt 2: Research Agent

```typescript
class ResearchAgent {
  private agent: any;
  private findings: Array<{ query: string; result: string }> = [];
  
  async initialize() {
    const tools = [
      new DynamicTool({
        name: 'search',
        description: 'Search for information',
        func: async (query) => this.search(query)
      }),
      new DynamicTool({
        name: 'analyze',
        description: 'Analyze text content',
        func: async (text) => this.analyze(text)
      }),
      new DynamicTool({
        name: 'summarize',
        description: 'Summarize findings',
        func: async () => this.summarize()
      })
    ];
    
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    
    this.agent = await initializeAgentExecutorWithOptions(tools, model, {
      agentType: 'openai-functions',
      verbose: true
    });
  }
  
  async research(topic: string, depth = 3) {
    const prompt = `Research this topic thoroughly: ${topic}
    
1. Find key information
2. Analyze findings
3. Summarize results

Depth: ${depth} levels of research`;

    const result = await this.agent.invoke({ input: prompt });
    return result.output;
  }
  
  private async search(query: string) {
    // Search implementation
    const results = `Results for: ${query}`;
    this.findings.push({ query, result: results });
    return results;
  }
  
  private async analyze(text: string) {
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    const analysis = await model.invoke(`Analyze this: ${text}`);
    return analysis.content;
  }
  
  private summarize() {
    return JSON.stringify(this.findings, null, 2);
  }
}
```

### Projekt 3: Code Assistant Agent

```typescript
class CodeAssistantAgent {
  private agent: any;
  private codebase: Map<string, string> = new Map();
  
  async initialize() {
    const tools = [
      new StructuredTool({
        name: 'read-file',
        description: 'Read code file',
        schema: z.object({ path: z.string() }),
        func: async ({ path }) => this.readFile(path)
      }),
      new StructuredTool({
        name: 'write-file',
        description: 'Write code file',
        schema: z.object({
          path: z.string(),
          content: z.string()
        }),
        func: async ({ path, content }) => this.writeFile(path, content)
      }),
      new DynamicTool({
        name: 'analyze-code',
        description: 'Analyze code quality',
        func: async (code) => this.analyzeCode(code)
      }),
      new DynamicTool({
        name: 'suggest-improvements',
        description: 'Suggest code improvements',
        func: async (code) => this.suggestImprovements(code)
      }),
      new Calculator()
    ];
    
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    
    this.agent = await initializeAgentExecutorWithOptions(tools, model, {
      agentType: 'openai-functions',
      verbose: true
    });
  }
  
  async assist(request: string) {
    return await this.agent.invoke({ input: request });
  }
  
  private async readFile(path: string) {
    return this.codebase.get(path) || 'File not found';
  }
  
  private async writeFile(path: string, content: string) {
    this.codebase.set(path, content);
    return 'File written';
  }
  
  private async analyzeCode(code: string) {
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    const analysis = await model.invoke(
      `Analyze this code for bugs and issues:\n${code}`
    );
    return analysis.content;
  }
  
  private async suggestImprovements(code: string) {
    const model = new ChatOpenAI({ modelName: 'gpt-4' });
    const suggestions = await model.invoke(
      `Suggest improvements:\n${code}`
    );
    return suggestions.content;
  }
}

// Verwendung
const assistant = new CodeAssistantAgent();
await assistant.initialize();

await assistant.assist('Read the file main.ts and analyze it for issues');
await assistant.assist('Suggest improvements and write them to main-improved.ts');
```

---

## Zusammenfassung

Du hast gelernt:

✅ **Agent-Basics**: ReAct Pattern, Agent-Typen
✅ **Tools**: Custom Tools, Structured Tools, Error Handling
✅ **Agent Execution**: Custom Loops, Streaming
✅ **Memory-Systeme**: Buffer, Window, Summary, Token-based
✅ **Conversation Memory**: Token Management, Custom Memory
✅ **Entity Memory**: Extraction, Knowledge Graphs
✅ **Vector Memory**: Semantic Memory, Long-Term Memory
✅ **Advanced Patterns**: Plan-Execute, Reflective Agents
✅ **Multi-Agent**: Collaboration, Hierarchical Systems
✅ **Projekte**: Personal Assistant, Research Agent, Code Assistant

### Best Practices

- [ ] Klare Tool-Beschreibungen für bessere Selection
- [ ] Memory-System an Use Case anpassen
- [ ] Error Handling in Tools implementieren
- [ ] Max Iterations begrenzen
- [ ] Wichtige Informationen in Long-Term Memory
- [ ] Reflection für kritische Tasks
- [ ] Multi-Agent für komplexe Probleme

Viel Erfolg mit Agents & Memory! 🤖
