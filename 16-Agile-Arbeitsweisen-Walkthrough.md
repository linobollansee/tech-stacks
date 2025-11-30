# Agile Arbeitsweisen - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in Agile](#einführung-in-agile)
2. [Agile Manifesto & Prinzipien](#agile-manifesto--prinzipien)
3. [Scrum Framework](#scrum-framework)
4. [Kanban Methode](#kanban-methode)
5. [User Stories & Backlog](#user-stories--backlog)
6. [Sprint Planning & Execution](#sprint-planning--execution)
7. [Daily Standups & Meetings](#daily-standups--meetings)
8. [Retrospektiven & Continuous Improvement](#retrospektiven--continuous-improvement)
9. [Agile Tools & Workflows](#agile-tools--workflows)
10. [Best Practices](#best-practices)
11. [Praktische Szenarien](#praktische-szenarien)

---

## Einführung in Agile

### Was ist Agile?

**Agile** ist eine iterative und inkrementelle Methode zur Software-Entwicklung, die Flexibilität, Zusammenarbeit und schnelles Feedback betont.

### Warum Agile?

**Flexibilität** 🔄
- Schnelle Anpassung an Änderungen
- Iterative Entwicklung
- Kontinuierliches Feedback

**Zusammenarbeit** 🤝
- Cross-funktionale Teams
- Direkte Kommunikation
- Stakeholder-Einbindung

**Qualität** ✨
- Kontinuierliche Verbesserung
- Regelmäßiges Testing
- Technical Excellence

### Agile vs. Waterfall

```
Waterfall:
  Requirements → Design → Implementation → Testing → Deployment
  ✗ Starre Planung
  ✗ Spätes Feedback
  ✗ Hohe Risiken

Agile:
  Sprint 1 → Sprint 2 → Sprint 3 → ...
  ✓ Iterativ & Inkrementell
  ✓ Frühes & häufiges Feedback
  ✓ Flexibel & anpassungsfähig
```

---

## Agile Manifesto & Prinzipien

### Die 4 Werte

```
1. Individuals and interactions > Processes and tools
   Menschen und Interaktionen wichtiger als Prozesse und Werkzeuge

2. Working software > Comprehensive documentation
   Funktionierende Software wichtiger als umfassende Dokumentation

3. Customer collaboration > Contract negotiation
   Zusammenarbeit mit dem Kunden wichtiger als Vertragsverhandlung

4. Responding to change > Following a plan
   Reagieren auf Veränderung wichtiger als Befolgen eines Plans
```

### Die 12 Prinzipien

```typescript
const agilePrinciples = [
  '1. Kundenzufriedenheit durch frühe und kontinuierliche Lieferung',
  '2. Willkommen heißen von sich ändernden Anforderungen',
  '3. Häufige Lieferung funktionierender Software',
  '4. Tägliche Zusammenarbeit zwischen Business und Entwicklern',
  '5. Motivierte Individuen mit Unterstützung und Vertrauen',
  '6. Face-to-Face Kommunikation',
  '7. Funktionierende Software als Hauptmaß des Fortschritts',
  '8. Nachhaltige Entwicklung',
  '9. Kontinuierliche Aufmerksamkeit auf technische Exzellenz',
  '10. Einfachheit - Kunst, Arbeit zu maximieren, die nicht getan wird',
  '11. Selbstorganisierende Teams',
  '12. Regelmäßige Reflexion und Anpassung'
];
```

---

## Scrum Framework

### Scrum Rollen

```typescript
interface ScrumTeam {
  productOwner: {
    role: 'Product Owner';
    responsibilities: [
      'Product Backlog Management',
      'Stakeholder Communication',
      'Vision & Strategy',
      'Priorisierung'
    ];
  };
  
  scrumMaster: {
    role: 'Scrum Master';
    responsibilities: [
      'Facilitating Scrum Events',
      'Removing Impediments',
      'Coaching the Team',
      'Protecting the Team'
    ];
  };
  
  developers: {
    role: 'Development Team';
    responsibilities: [
      'Sprint Planning',
      'Daily Standup',
      'Development & Testing',
      'Sprint Review & Retrospective'
    ];
    size: '3-9 Members';
    crossFunctional: true;
    selfOrganizing: true;
  };
}
```

### Scrum Events

```typescript
class ScrumEvents {
  // Sprint: 1-4 Wochen Container
  sprint = {
    duration: '1-4 weeks',
    goal: 'Potentially shippable increment',
    events: ['Planning', 'Daily Standup', 'Review', 'Retrospective']
  };
  
  // Sprint Planning: Was und Wie?
  sprintPlanning = {
    duration: '8 hours (4-week sprint)',
    participants: ['Product Owner', 'Scrum Master', 'Dev Team'],
    outcome: 'Sprint Goal + Sprint Backlog',
    parts: {
      part1: 'What can be done?',
      part2: 'How will it be done?'
    }
  };
  
  // Daily Standup: 15 Minuten Synchronisation
  dailyStandup = {
    duration: '15 minutes',
    time: 'Same time & place daily',
    questions: [
      'What did I do yesterday?',
      'What will I do today?',
      'Any impediments?'
    ]
  };
  
  // Sprint Review: Demo & Feedback
  sprintReview = {
    duration: '4 hours (4-week sprint)',
    participants: ['Team + Stakeholders'],
    purpose: 'Inspect increment & adapt backlog',
    outcome: 'Updated Product Backlog'
  };
  
  // Retrospective: Verbesserung
  sprintRetrospective = {
    duration: '3 hours (4-week sprint)',
    participants: ['Scrum Team'],
    purpose: 'Inspect process & create improvement plan',
    format: 'What went well? What can improve? Actions?'
  };
}
```

### Scrum Artifacts

```typescript
interface ScrumArtifacts {
  productBacklog: {
    description: 'Ordered list of everything needed';
    owner: 'Product Owner';
    dynamic: true;
    items: UserStory[];
  };
  
  sprintBacklog: {
    description: 'Items selected for Sprint + plan';
    owner: 'Development Team';
    commitment: 'Sprint Goal';
    visible: true;
  };
  
  increment: {
    description: 'Sum of completed PBIs + previous increments';
    state: 'Done';
    releasable: true;
  };
}

interface UserStory {
  id: string;
  title: string;
  description: string;
  acceptanceCriteria: string[];
  priority: number;
  estimation: number;
  status: 'To Do' | 'In Progress' | 'Done';
}
```

---

## Kanban Methode

### Kanban Prinzipien

```typescript
const kanbanPrinciples = {
  visualize: 'Make work visible',
  limitWIP: 'Limit Work in Progress',
  manageFlow: 'Manage and improve flow',
  explicitPolicies: 'Make process policies explicit',
  feedbackLoops: 'Implement feedback loops',
  improve: 'Improve collaboratively, evolve experimentally'
};
```

### Kanban Board

```typescript
interface KanbanBoard {
  columns: Column[];
  cards: Card[];
  wipLimits: Map<string, number>;
}

interface Column {
  name: string;
  wipLimit?: number;
  cards: Card[];
}

interface Card {
  id: string;
  title: string;
  description: string;
  assignee?: string;
  priority: 'High' | 'Medium' | 'Low';
  blocked?: boolean;
  tags: string[];
}

class KanbanBoardManager {
  private board: KanbanBoard;
  
  constructor() {
    this.board = {
      columns: [
        { name: 'Backlog', cards: [] },
        { name: 'Ready', wipLimit: 5, cards: [] },
        { name: 'In Progress', wipLimit: 3, cards: [] },
        { name: 'Review', wipLimit: 2, cards: [] },
        { name: 'Done', cards: [] }
      ],
      cards: [],
      wipLimits: new Map([
        ['Ready', 5],
        ['In Progress', 3],
        ['Review', 2]
      ])
    };
  }
  
  canMoveCard(columnName: string): boolean {
    const column = this.board.columns.find(c => c.name === columnName);
    if (!column) return false;
    
    const wipLimit = this.board.wipLimits.get(columnName);
    if (!wipLimit) return true;
    
    return column.cards.length < wipLimit;
  }
  
  moveCard(cardId: string, toColumn: string): boolean {
    if (!this.canMoveCard(toColumn)) {
      console.log(`WIP limit reached for ${toColumn}`);
      return false;
    }
    
    // Remove from current column
    for (const column of this.board.columns) {
      const index = column.cards.findIndex(c => c.id === cardId);
      if (index !== -1) {
        const card = column.cards.splice(index, 1)[0];
        
        // Add to new column
        const targetColumn = this.board.columns.find(c => c.name === toColumn);
        if (targetColumn) {
          targetColumn.cards.push(card);
          return true;
        }
      }
    }
    
    return false;
  }
  
  getMetrics() {
    return {
      leadTime: this.calculateLeadTime(),
      cycleTime: this.calculateCycleTime(),
      throughput: this.calculateThroughput(),
      wipViolations: this.checkWIPViolations()
    };
  }
  
  private calculateLeadTime(): number {
    // Time from creation to completion
    return 0; // Implementation
  }
  
  private calculateCycleTime(): number {
    // Time from start to completion
    return 0; // Implementation
  }
  
  private calculateThroughput(): number {
    // Cards completed per time period
    return 0; // Implementation
  }
  
  private checkWIPViolations(): string[] {
    const violations: string[] = [];
    
    for (const column of this.board.columns) {
      const limit = this.board.wipLimits.get(column.name);
      if (limit && column.cards.length > limit) {
        violations.push(`${column.name}: ${column.cards.length}/${limit}`);
      }
    }
    
    return violations;
  }
}
```

---

## User Stories & Backlog

### User Story Format

```typescript
class UserStory {
  id: string;
  title: string;
  
  // As a [role], I want [feature], so that [benefit]
  asA: string;
  iWant: string;
  soThat: string;
  
  acceptanceCriteria: AcceptanceCriterion[];
  estimation: number;
  priority: number;
  
  // INVEST Criteria
  independent: boolean;
  negotiable: boolean;
  valuable: boolean;
  estimable: boolean;
  small: boolean;
  testable: boolean;
  
  constructor(data: Partial<UserStory>) {
    Object.assign(this, data);
  }
  
  getDescription(): string {
    return `Als ${this.asA} möchte ich ${this.iWant}, damit ${this.soThat}.`;
  }
  
  isReady(): boolean {
    return this.independent &&
           this.negotiable &&
           this.valuable &&
           this.estimable &&
           this.small &&
           this.testable &&
           this.acceptanceCriteria.length > 0;
  }
}

interface AcceptanceCriterion {
  given: string;  // Given [initial context]
  when: string;   // When [event occurs]
  then: string;   // Then [ensure outcome]
}

// Beispiel
const story = new UserStory({
  id: 'US-123',
  title: 'User Login',
  asA: 'registrierter Benutzer',
  iWant: 'mich einloggen können',
  soThat: 'ich auf meine persönlichen Daten zugreifen kann',
  acceptanceCriteria: [
    {
      given: 'Ich bin auf der Login-Seite',
      when: 'Ich gültige Credentials eingebe',
      then: 'werde ich zum Dashboard weitergeleitet'
    },
    {
      given: 'Ich bin auf der Login-Seite',
      when: 'Ich ungültige Credentials eingebe',
      then: 'sehe ich eine Fehlermeldung'
    }
  ],
  estimation: 5,
  priority: 1
});

console.log(story.getDescription());
```

### Product Backlog Management

```typescript
class ProductBacklog {
  private items: UserStory[] = [];
  
  addStory(story: UserStory): void {
    this.items.push(story);
    this.prioritize();
  }
  
  removeStory(id: string): void {
    this.items = this.items.filter(item => item.id !== id);
  }
  
  updateStory(id: string, updates: Partial<UserStory>): void {
    const story = this.items.find(item => item.id === id);
    if (story) {
      Object.assign(story, updates);
      this.prioritize();
    }
  }
  
  private prioritize(): void {
    // MoSCoW Method: Must, Should, Could, Won't
    // WSJF: Weighted Shortest Job First
    this.items.sort((a, b) => {
      // Higher priority first
      if (a.priority !== b.priority) {
        return b.priority - a.priority;
      }
      // Smaller estimation if same priority
      return a.estimation - b.estimation;
    });
  }
  
  getTopStories(count: number): UserStory[] {
    return this.items.slice(0, count);
  }
  
  getReadyStories(): UserStory[] {
    return this.items.filter(story => story.isReady());
  }
  
  getVelocityEstimate(completedSprints: Sprint[]): number {
    if (completedSprints.length === 0) return 0;
    
    const totalPoints = completedSprints.reduce(
      (sum, sprint) => sum + sprint.completedPoints,
      0
    );
    
    return totalPoints / completedSprints.length;
  }
}

interface Sprint {
  number: number;
  startDate: Date;
  endDate: Date;
  committedPoints: number;
  completedPoints: number;
  stories: UserStory[];
}
```

---

## Sprint Planning & Execution

### Sprint Planning

```typescript
class SprintPlanning {
  private backlog: ProductBacklog;
  private team: TeamMember[];
  private velocity: number;
  
  constructor(backlog: ProductBacklog, team: TeamMember[], velocity: number) {
    this.backlog = backlog;
    this.team = team;
    this.velocity = velocity;
  }
  
  planSprint(sprintGoal: string): Sprint {
    console.log('🎯 Sprint Goal:', sprintGoal);
    
    // Part 1: What can be done?
    const selectedStories = this.selectStories();
    
    // Part 2: How will it be done?
    const tasks = this.breakdownIntoTasks(selectedStories);
    
    return {
      number: this.getNextSprintNumber(),
      startDate: new Date(),
      endDate: this.getSprintEndDate(),
      committedPoints: this.calculateTotalPoints(selectedStories),
      completedPoints: 0,
      stories: selectedStories,
      tasks,
      goal: sprintGoal
    };
  }
  
  private selectStories(): UserStory[] {
    const readyStories = this.backlog.getReadyStories();
    const selected: UserStory[] = [];
    let totalPoints = 0;
    
    for (const story of readyStories) {
      if (totalPoints + story.estimation <= this.velocity) {
        selected.push(story);
        totalPoints += story.estimation;
      } else {
        break;
      }
    }
    
    console.log(`✅ Selected ${selected.length} stories (${totalPoints} points)`);
    return selected;
  }
  
  private breakdownIntoTasks(stories: UserStory[]): Task[] {
    const tasks: Task[] = [];
    
    for (const story of stories) {
      // Break story into technical tasks
      const storyTasks = this.createTasksForStory(story);
      tasks.push(...storyTasks);
    }
    
    return tasks;
  }
  
  private createTasksForStory(story: UserStory): Task[] {
    return [
      {
        id: `${story.id}-T1`,
        title: 'Design & API Definition',
        storyId: story.id,
        estimatedHours: 4,
        status: 'To Do'
      },
      {
        id: `${story.id}-T2`,
        title: 'Backend Implementation',
        storyId: story.id,
        estimatedHours: 8,
        status: 'To Do'
      },
      {
        id: `${story.id}-T3`,
        title: 'Frontend Implementation',
        storyId: story.id,
        estimatedHours: 6,
        status: 'To Do'
      },
      {
        id: `${story.id}-T4`,
        title: 'Testing',
        storyId: story.id,
        estimatedHours: 4,
        status: 'To Do'
      },
      {
        id: `${story.id}-T5`,
        title: 'Code Review & Documentation',
        storyId: story.id,
        estimatedHours: 2,
        status: 'To Do'
      }
    ];
  }
  
  private getNextSprintNumber(): number {
    return 1; // Implementation
  }
  
  private getSprintEndDate(): Date {
    const endDate = new Date();
    endDate.setDate(endDate.getDate() + 14); // 2-week sprint
    return endDate;
  }
  
  private calculateTotalPoints(stories: UserStory[]): number {
    return stories.reduce((sum, story) => sum + story.estimation, 0);
  }
}

interface Task {
  id: string;
  title: string;
  storyId: string;
  estimatedHours: number;
  actualHours?: number;
  assignee?: string;
  status: 'To Do' | 'In Progress' | 'Done';
  blockers?: string[];
}

interface TeamMember {
  id: string;
  name: string;
  role: string;
  capacity: number; // hours per sprint
}
```

---

## Daily Standups & Meetings

### Daily Standup Structure

```typescript
class DailyStandup {
  private date: Date;
  private updates: StandupUpdate[] = [];
  private impediments: Impediment[] = [];
  
  constructor() {
    this.date = new Date();
  }
  
  addUpdate(member: string, yesterday: string, today: string, blockers?: string[]): void {
    this.updates.push({
      member,
      yesterday,
      today,
      blockers: blockers || []
    });
    
    if (blockers && blockers.length > 0) {
      blockers.forEach(blocker => {
        this.impediments.push({
          reportedBy: member,
          description: blocker,
          reportedDate: this.date,
          resolved: false
        });
      });
    }
  }
  
  getSummary(): string {
    let summary = `📅 Daily Standup - ${this.date.toLocaleDateString()}\n\n`;
    
    this.updates.forEach(update => {
      summary += `👤 ${update.member}\n`;
      summary += `  ✅ Yesterday: ${update.yesterday}\n`;
      summary += `  🎯 Today: ${update.today}\n`;
      if (update.blockers.length > 0) {
        summary += `  ⚠️  Blockers: ${update.blockers.join(', ')}\n`;
      }
      summary += '\n';
    });
    
    if (this.impediments.length > 0) {
      summary += '🚧 Impediments to resolve:\n';
      this.impediments.forEach((imp, idx) => {
        summary += `  ${idx + 1}. ${imp.description} (${imp.reportedBy})\n`;
      });
    }
    
    return summary;
  }
  
  getImpediments(): Impediment[] {
    return this.impediments.filter(imp => !imp.resolved);
  }
}

interface StandupUpdate {
  member: string;
  yesterday: string;
  today: string;
  blockers: string[];
}

interface Impediment {
  reportedBy: string;
  description: string;
  reportedDate: Date;
  resolved: boolean;
  resolvedDate?: Date;
  resolvedBy?: string;
}

// Beispiel
const standup = new DailyStandup();

standup.addUpdate(
  'Alice',
  'Completed user authentication',
  'Working on password reset flow',
  []
);

standup.addUpdate(
  'Bob',
  'Started dashboard implementation',
  'Continue dashboard, need API documentation',
  ['Missing API documentation']
);

standup.addUpdate(
  'Charlie',
  'Fixed bug in payment module',
  'Writing tests for payment module',
  []
);

console.log(standup.getSummary());
```

---

## Retrospektiven & Continuous Improvement

### Retrospektive Format

```typescript
class SprintRetrospective {
  private sprint: Sprint;
  private wentWell: string[] = [];
  private needsImprovement: string[] = [];
  private actionItems: ActionItem[] = [];
  
  constructor(sprint: Sprint) {
    this.sprint = sprint;
  }
  
  addWentWell(item: string): void {
    this.wentWell.push(item);
  }
  
  addNeedsImprovement(item: string): void {
    this.needsImprovement.push(item);
  }
  
  addActionItem(item: ActionItem): void {
    this.actionItems.push(item);
  }
  
  generateReport(): string {
    let report = `🔄 Sprint ${this.sprint.number} Retrospective\n\n`;
    
    report += '✅ What went well:\n';
    this.wentWell.forEach((item, idx) => {
      report += `  ${idx + 1}. ${item}\n`;
    });
    
    report += '\n🔧 What needs improvement:\n';
    this.needsImprovement.forEach((item, idx) => {
      report += `  ${idx + 1}. ${item}\n`;
    });
    
    report += '\n🎯 Action Items:\n';
    this.actionItems.forEach((item, idx) => {
      report += `  ${idx + 1}. ${item.description}\n`;
      report += `     Owner: ${item.owner}\n`;
      report += `     Due: ${item.dueDate.toLocaleDateString()}\n`;
    });
    
    return report;
  }
  
  // Retrospective Techniques
  static startStopContinue() {
    return {
      start: [] as string[],    // What should we start doing?
      stop: [] as string[],     // What should we stop doing?
      continue: [] as string[]  // What should we continue doing?
    };
  }
  
  static madSadGlad() {
    return {
      mad: [] as string[],    // What made us angry/frustrated?
      sad: [] as string[],    // What made us disappointed?
      glad: [] as string[]    // What made us happy?
    };
  }
  
  static fourLs() {
    return {
      liked: [] as string[],     // What did we like?
      learned: [] as string[],   // What did we learn?
      lacked: [] as string[],    // What did we lack?
      longedFor: [] as string[]  // What did we long for?
    };
  }
}

interface ActionItem {
  description: string;
  owner: string;
  dueDate: Date;
  status: 'Open' | 'In Progress' | 'Done';
}

// Beispiel
const retro = new SprintRetrospective({
  number: 5,
  startDate: new Date('2025-11-01'),
  endDate: new Date('2025-11-14'),
  committedPoints: 34,
  completedPoints: 32
});

retro.addWentWell('Great team collaboration');
retro.addWentWell('CI/CD pipeline improvements');
retro.addWentWell('Better code review process');

retro.addNeedsImprovement('Too many meetings');
retro.addNeedsImprovement('Unclear requirements');
retro.addNeedsImprovement('Technical debt growing');

retro.addActionItem({
  description: 'Reduce meeting time by 20%',
  owner: 'Scrum Master',
  dueDate: new Date('2025-11-21'),
  status: 'Open'
});

retro.addActionItem({
  description: 'Schedule backlog refinement sessions',
  owner: 'Product Owner',
  dueDate: new Date('2025-11-18'),
  status: 'Open'
});

console.log(retro.generateReport());
```

---

## Agile Tools & Workflows

### Jira Workflow Simulation

```typescript
interface JiraIssue {
  key: string;
  type: 'Story' | 'Task' | 'Bug' | 'Epic';
  summary: string;
  description: string;
  status: string;
  assignee?: string;
  reporter: string;
  priority: 'Highest' | 'High' | 'Medium' | 'Low' | 'Lowest';
  storyPoints?: number;
  sprint?: string;
  labels: string[];
  comments: Comment[];
}

class AgileProjectManagement {
  private issues: Map<string, JiraIssue> = new Map();
  private workflow = ['To Do', 'In Progress', 'Code Review', 'Testing', 'Done'];
  
  createIssue(issue: JiraIssue): string {
    this.issues.set(issue.key, {
      ...issue,
      status: 'To Do',
      comments: []
    });
    return issue.key;
  }
  
  transitionIssue(key: string, newStatus: string): boolean {
    const issue = this.issues.get(key);
    if (!issue) return false;
    
    const currentIndex = this.workflow.indexOf(issue.status);
    const newIndex = this.workflow.indexOf(newStatus);
    
    // Can only move forward or backward by one step
    if (Math.abs(newIndex - currentIndex) !== 1) {
      console.log('Invalid transition');
      return false;
    }
    
    issue.status = newStatus;
    this.addComment(key, `Status changed to ${newStatus}`, 'System');
    return true;
  }
  
  addComment(key: string, text: string, author: string): void {
    const issue = this.issues.get(key);
    if (issue) {
      issue.comments.push({
        author,
        text,
        timestamp: new Date()
      });
    }
  }
  
  getSprintBurndown(sprintName: string): BurndownData {
    const sprintIssues = Array.from(this.issues.values())
      .filter(issue => issue.sprint === sprintName);
    
    const totalPoints = sprintIssues.reduce(
      (sum, issue) => sum + (issue.storyPoints || 0),
      0
    );
    
    const completedPoints = sprintIssues
      .filter(issue => issue.status === 'Done')
      .reduce((sum, issue) => sum + (issue.storyPoints || 0), 0);
    
    return {
      total: totalPoints,
      completed: completedPoints,
      remaining: totalPoints - completedPoints
    };
  }
}

interface Comment {
  author: string;
  text: string;
  timestamp: Date;
}

interface BurndownData {
  total: number;
  completed: number;
  remaining: number;
}
```

---

## Best Practices

### Definition of Done (DoD)

```typescript
class DefinitionOfDone {
  private criteria = [
    'Code geschrieben und committed',
    'Unit Tests geschrieben und passing',
    'Integration Tests passing',
    'Code Review durchgeführt',
    'Dokumentation aktualisiert',
    'Acceptance Criteria erfüllt',
    'Kein Critical/Blocker Bug',
    'Performance getestet',
    'Security Review durchgeführt',
    'Deployed to Staging',
    'Product Owner Acceptance'
  ];
  
  checkStory(story: UserStory, checklist: Map<string, boolean>): boolean {
    let allDone = true;
    
    console.log(`\n📋 Definition of Done für ${story.id}:\n`);
    
    for (const criterion of this.criteria) {
      const done = checklist.get(criterion) || false;
      const symbol = done ? '✅' : '❌';
      console.log(`${symbol} ${criterion}`);
      
      if (!done) allDone = false;
    }
    
    return allDone;
  }
}
```

### Estimation Techniques

```typescript
class EstimationTechniques {
  // Planning Poker
  static planningPoker(story: UserStory, team: TeamMember[]): number {
    const fibonacciSequence = [1, 2, 3, 5, 8, 13, 21];
    const estimates: number[] = [];
    
    // Each team member estimates
    team.forEach(member => {
      const estimate = this.getRandomFibonacci(fibonacciSequence);
      estimates.push(estimate);
      console.log(`${member.name}: ${estimate} points`);
    });
    
    // Discuss outliers
    const min = Math.min(...estimates);
    const max = Math.max(...estimates);
    
    if (max - min > 5) {
      console.log('Large variance - discussion needed');
    }
    
    // Consensus or average
    return Math.round(estimates.reduce((a, b) => a + b) / estimates.length);
  }
  
  // T-Shirt Sizing
  static tShirtSizing(story: UserStory): string {
    const sizes = ['XS', 'S', 'M', 'L', 'XL'];
    
    // Complexity factors
    const complexity = this.assessComplexity(story);
    
    if (complexity <= 2) return 'XS';
    if (complexity <= 5) return 'S';
    if (complexity <= 10) return 'M';
    if (complexity <= 20) return 'L';
    return 'XL';
  }
  
  private static getRandomFibonacci(sequence: number[]): number {
    return sequence[Math.floor(Math.random() * sequence.length)];
  }
  
  private static assessComplexity(story: UserStory): number {
    let complexity = 5; // Base complexity
    
    if (story.acceptanceCriteria.length > 5) complexity += 5;
    if (!story.independent) complexity += 3;
    if (!story.testable) complexity += 2;
    
    return complexity;
  }
}
```

---

## Praktische Szenarien

### Szenario 1: Neues Feature entwickeln

```typescript
async function developNewFeature() {
  console.log('🚀 Entwicklung: User Profile Feature\n');
  
  // 1. Product Backlog
  const backlog = new ProductBacklog();
  const userStory = new UserStory({
    id: 'US-456',
    title: 'User Profile Page',
    asA: 'registrierter Benutzer',
    iWant: 'mein Profil ansehen und bearbeiten',
    soThat: 'ich meine Daten aktuell halten kann',
    acceptanceCriteria: [
      {
        given: 'Ich bin eingeloggt',
        when: 'Ich auf mein Profil gehe',
        then: 'sehe ich meine aktuellen Daten'
      },
      {
        given: 'Ich bin auf meiner Profilseite',
        when: 'Ich Änderungen vornehme und speichere',
        then: 'werden meine Daten aktualisiert'
      }
    ],
    estimation: 8,
    priority: 2,
    independent: true,
    negotiable: true,
    valuable: true,
    estimable: true,
    small: true,
    testable: true
  });
  
  backlog.addStory(userStory);
  
  // 2. Sprint Planning
  const planning = new SprintPlanning(backlog, [], 40);
  const sprint = planning.planSprint('Implement User Profile Management');
  
  console.log(`Sprint ${sprint.number} geplant mit ${sprint.committedPoints} Points\n`);
  
  // 3. Daily Development
  for (let day = 1; day <= 10; day++) {
    console.log(`\n📅 Tag ${day}`);
    
    const standup = new DailyStandup();
    standup.addUpdate(
      'Developer 1',
      'Backend API endpoints',
      'Frontend implementation',
      []
    );
    
    if (day === 5) {
      standup.addUpdate(
        'Developer 2',
        'UI components',
        'Integration testing',
        ['Missing test data']
      );
    }
  }
  
  // 4. Sprint Review
  console.log('\n🎉 Sprint Review: Feature Demo');
  console.log('✅ User kann Profil ansehen');
  console.log('✅ User kann Profil bearbeiten');
  console.log('✅ Validierung funktioniert');
  
  // 5. Retrospective
  const retro = new SprintRetrospective(sprint);
  retro.addWentWell('Gute Zusammenarbeit im Team');
  retro.addNeedsImprovement('Mehr Pair Programming');
  retro.addActionItem({
    description: 'Pair Programming Sessions einplanen',
    owner: 'Team',
    dueDate: new Date(),
    status: 'Open'
  });
  
  console.log('\n' + retro.generateReport());
}
```

### Szenario 2: Bug Fixing Sprint

```typescript
async function bugFixingSprint() {
  const bugs = [
    {
      key: 'BUG-101',
      summary: 'Login fails with special characters',
      priority: 'Highest' as const,
      storyPoints: 3
    },
    {
      key: 'BUG-102',
      summary: 'Profile image not displaying',
      priority: 'High' as const,
      storyPoints: 2
    },
    {
      key: 'BUG-103',
      summary: 'Email notifications delayed',
      priority: 'Medium' as const,
      storyPoints: 5
    }
  ];
  
  console.log('🐛 Bug Fixing Sprint\n');
  
  const pm = new AgileProjectManagement();
  
  bugs.forEach(bug => {
    const key = pm.createIssue({
      ...bug,
      type: 'Bug',
      description: `Fix: ${bug.summary}`,
      status: 'To Do',
      reporter: 'QA Team',
      labels: ['bug', 'production']
    });
    
    console.log(`Created ${key}: ${bug.summary}`);
  });
  
  // Workflow
  console.log('\n📊 Bug Workflow:');
  console.log('To Do → In Progress → Code Review → Testing → Done');
}
```

---

## Zusammenfassung

Du hast gelernt:

✅ **Agile Basics**: Manifesto, Prinzipien, vs Waterfall
✅ **Scrum**: Rollen, Events, Artifacts
✅ **Kanban**: Prinzipien, Board, WIP Limits
✅ **User Stories**: INVEST, Acceptance Criteria
✅ **Sprint Planning**: Selection, Breakdown, Estimation
✅ **Daily Standups**: 3 Questions, Impediments
✅ **Retrospektiven**: Formate, Action Items
✅ **Tools**: Jira Workflow, Metrics
✅ **Best Practices**: DoD, Estimation Techniques
✅ **Praktische Szenarien**: Feature Development, Bug Fixing

### Agile Mindset

- [ ] Embrace Change
- [ ] Continuous Improvement
- [ ] Team Collaboration
- [ ] Customer Focus
- [ ] Transparency
- [ ] Inspect & Adapt
- [ ] Sustainable Pace
- [ ] Technical Excellence

### Hilfreiche Ressourcen

- [Scrum Guide](https://scrumguides.org/)
- [Agile Manifesto](https://agilemanifesto.org/)
- [Kanban Guide](https://kanban.university/)
- [Atlassian Agile](https://www.atlassian.com/agile)

Viel Erfolg mit Agile! 🚀
