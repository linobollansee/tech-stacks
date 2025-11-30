# Teamprojekte - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in Teamprojekte](#einführung-in-teamprojekte)
2. [Projektplanung & Setup](#projektplanung--setup)
3. [Team-Zusammenarbeit](#team-zusammenarbeit)
4. [Git Workflows für Teams](#git-workflows-für-teams)
5. [Code Review & Quality](#code-review--quality)
6. [Kommunikation & Dokumentation](#kommunikation--dokumentation)
7. [Projektmanagement](#projektmanagement)
8. [Testing & CI/CD im Team](#testing--cicd-im-team)
9. [Deployment & Operations](#deployment--operations)
10. [Konfliktlösung & Best Practices](#konfliktlösung--best-practices)
11. [Beispiel-Projekt: E-Commerce Platform](#beispiel-projekt-e-commerce-platform)

---

## Einführung in Teamprojekte

### Was macht ein erfolgreiches Teamprojekt aus?

**Klare Ziele** 🎯
- Gemeinsames Verständnis
- Messbare Objectives
- Definierte Meilensteine

**Gute Kommunikation** 💬
- Regelmäßiger Austausch
- Dokumentation
- Transparenz

**Technische Exzellenz** 🔧
- Code-Qualität
- Best Practices
- Kontinuierliche Verbesserung

### Team-Rollen

```typescript
interface TeamStructure {
  projectManager: {
    responsibilities: [
      'Projektplanung',
      'Ressourcen-Management',
      'Stakeholder-Kommunikation',
      'Risk Management'
    ];
  };
  
  techLead: {
    responsibilities: [
      'Technische Architektur',
      'Code Reviews',
      'Mentoring',
      'Technische Entscheidungen'
    ];
  };
  
  developers: {
    responsibilities: [
      'Feature-Implementierung',
      'Testing',
      'Code Reviews',
      'Dokumentation'
    ];
  };
  
  designer: {
    responsibilities: [
      'UI/UX Design',
      'Prototyping',
      'Design System',
      'User Research'
    ];
  };
  
  qa: {
    responsibilities: [
      'Test Planning',
      'Manual Testing',
      'Automation',
      'Bug Tracking'
    ];
  };
}
```

---

## Projektplanung & Setup

### Projekt-Kickoff

```typescript
class ProjectKickoff {
  async conductKickoff(): Promise<ProjectPlan> {
    console.log('🚀 Projekt-Kickoff Meeting\n');
    
    // 1. Vision & Ziele
    const vision = this.defineVision();
    console.log('Vision:', vision);
    
    // 2. Scope Definition
    const scope = this.defineScope();
    console.log('\nScope:', scope);
    
    // 3. Team Formation
    const team = this.formTeam();
    console.log('\nTeam:', team);
    
    // 4. Timeline
    const timeline = this.createTimeline();
    console.log('\nTimeline:', timeline);
    
    // 5. Success Criteria
    const criteria = this.defineSuccessCriteria();
    console.log('\nSuccess Criteria:', criteria);
    
    return {
      vision,
      scope,
      team,
      timeline,
      criteria
    };
  }
  
  private defineVision(): ProjectVision {
    return {
      statement: 'Build a modern e-commerce platform',
      objectives: [
        'User-friendly shopping experience',
        'Secure payment processing',
        'Scalable architecture',
        'Mobile-first design'
      ],
      targetAudience: 'Online shoppers',
      valueProposition: 'Fast, secure, and convenient online shopping'
    };
  }
  
  private defineScope(): ProjectScope {
    return {
      inScope: [
        'User authentication',
        'Product catalog',
        'Shopping cart',
        'Checkout & payment',
        'Order management',
        'Admin dashboard'
      ],
      outOfScope: [
        'Mobile apps (Phase 2)',
        'Advanced analytics (Phase 2)',
        'Multi-language support (Phase 2)'
      ],
      constraints: [
        'Budget: €50,000',
        'Timeline: 3 months',
        'Team size: 5 developers'
      ]
    };
  }
  
  private formTeam(): TeamMember[] {
    return [
      { name: 'Alice', role: 'Tech Lead', skills: ['React', 'Node.js', 'AWS'] },
      { name: 'Bob', role: 'Backend Dev', skills: ['Node.js', 'PostgreSQL', 'Docker'] },
      { name: 'Charlie', role: 'Frontend Dev', skills: ['React', 'TypeScript', 'CSS'] },
      { name: 'Diana', role: 'Full-Stack Dev', skills: ['React', 'Node.js', 'MongoDB'] },
      { name: 'Eve', role: 'QA Engineer', skills: ['Jest', 'Playwright', 'CI/CD'] }
    ];
  }
  
  private createTimeline(): ProjectMilestone[] {
    return [
      { name: 'Project Setup', date: new Date('2025-12-01'), status: 'planned' },
      { name: 'MVP Release', date: new Date('2026-01-15'), status: 'planned' },
      { name: 'Beta Testing', date: new Date('2026-02-01'), status: 'planned' },
      { name: 'Production Launch', date: new Date('2026-02-28'), status: 'planned' }
    ];
  }
  
  private defineSuccessCriteria(): string[] {
    return [
      'All core features implemented',
      'Test coverage > 80%',
      'Page load time < 2s',
      'Zero critical bugs in production',
      'Positive user feedback'
    ];
  }
}

interface ProjectVision {
  statement: string;
  objectives: string[];
  targetAudience: string;
  valueProposition: string;
}

interface ProjectScope {
  inScope: string[];
  outOfScope: string[];
  constraints: string[];
}

interface TeamMember {
  name: string;
  role: string;
  skills: string[];
}

interface ProjectMilestone {
  name: string;
  date: Date;
  status: 'planned' | 'in-progress' | 'completed' | 'delayed';
}

interface ProjectPlan {
  vision: ProjectVision;
  scope: ProjectScope;
  team: TeamMember[];
  timeline: ProjectMilestone[];
  criteria: string[];
}
```

### Repository Setup

```typescript
class RepositorySetup {
  async setupProject(projectName: string): Promise<void> {
    console.log(`🔧 Setting up project: ${projectName}\n`);
    
    // 1. Create repository structure
    this.createDirectoryStructure();
    
    // 2. Initialize Git
    this.initializeGit();
    
    // 3. Setup branch protection
    this.setupBranchProtection();
    
    // 4. Configure CI/CD
    this.configureCICD();
    
    // 5. Setup documentation
    this.setupDocumentation();
  }
  
  private createDirectoryStructure(): void {
    const structure = `
    project-root/
    ├── .github/
    │   ├── workflows/          # GitHub Actions
    │   ├── PULL_REQUEST_TEMPLATE.md
    │   └── ISSUE_TEMPLATE/
    ├── src/
    │   ├── components/         # React components
    │   ├── pages/              # Next.js pages
    │   ├── lib/                # Utilities
    │   ├── hooks/              # Custom hooks
    │   ├── types/              # TypeScript types
    │   └── styles/             # CSS/styled-components
    ├── tests/
    │   ├── unit/
    │   ├── integration/
    │   └── e2e/
    ├── docs/
    │   ├── architecture.md
    │   ├── api.md
    │   └── deployment.md
    ├── .eslintrc.js
    ├── .prettierrc
    ├── tsconfig.json
    ├── package.json
    ├── README.md
    └── CONTRIBUTING.md
    `;
    
    console.log('Directory structure:', structure);
  }
  
  private initializeGit(): void {
    const gitCommands = [
      'git init',
      'git add .',
      'git commit -m "Initial commit"',
      'git branch -M main',
      'git remote add origin <repository-url>',
      'git push -u origin main'
    ];
    
    console.log('\nGit initialization commands:');
    gitCommands.forEach(cmd => console.log(`  ${cmd}`));
  }
  
  private setupBranchProtection(): void {
    const rules = {
      main: {
        requirePullRequest: true,
        requiredReviews: 2,
        requireStatusChecks: true,
        requiredChecks: ['build', 'test', 'lint'],
        enforceAdmins: true,
        requireLinearHistory: true
      },
      develop: {
        requirePullRequest: true,
        requiredReviews: 1,
        requireStatusChecks: true
      }
    };
    
    console.log('\nBranch protection rules:', JSON.stringify(rules, null, 2));
  }
  
  private configureCICD(): void {
    const ciConfig = `
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build
    `;
    
    console.log('\nCI/CD Configuration:', ciConfig);
  }
  
  private setupDocumentation(): void {
    console.log('\nDocumentation templates created:');
    console.log('  ✓ README.md');
    console.log('  ✓ CONTRIBUTING.md');
    console.log('  ✓ CODE_OF_CONDUCT.md');
    console.log('  ✓ API.md');
    console.log('  ✓ ARCHITECTURE.md');
  }
}
```

---

## Team-Zusammenarbeit

### Stand-up Meetings

```typescript
class TeamStandup {
  private date: Date;
  private updates: Map<string, StandupUpdate> = new Map();
  
  constructor() {
    this.date = new Date();
  }
  
  addUpdate(member: string, update: StandupUpdate): void {
    this.updates.set(member, update);
  }
  
  generateReport(): string {
    let report = `📅 Daily Standup - ${this.date.toLocaleDateString()}\n\n`;
    
    this.updates.forEach((update, member) => {
      report += `👤 ${member}\n`;
      report += `  ✅ Completed: ${update.completed}\n`;
      report += `  🎯 Today: ${update.today}\n`;
      report += `  ⏱️  Hours: ${update.hoursWorked}\n`;
      
      if (update.blockers.length > 0) {
        report += `  🚧 Blockers:\n`;
        update.blockers.forEach(blocker => {
          report += `     - ${blocker}\n`;
        });
      }
      
      report += '\n';
    });
    
    return report;
  }
  
  getBlockers(): Blocker[] {
    const blockers: Blocker[] = [];
    
    this.updates.forEach((update, member) => {
      update.blockers.forEach(description => {
        blockers.push({
          member,
          description,
          reportedDate: this.date
        });
      });
    });
    
    return blockers;
  }
}

interface StandupUpdate {
  completed: string;
  today: string;
  hoursWorked: number;
  blockers: string[];
}

interface Blocker {
  member: string;
  description: string;
  reportedDate: Date;
}

// Beispiel
const standup = new TeamStandup();

standup.addUpdate('Alice', {
  completed: 'Implemented user authentication API',
  today: 'Work on JWT token refresh',
  hoursWorked: 8,
  blockers: []
});

standup.addUpdate('Bob', {
  completed: 'Setup PostgreSQL database',
  today: 'Create database migrations',
  hoursWorked: 6,
  blockers: ['Need database credentials for staging']
});

console.log(standup.generateReport());
```

### Pair Programming

```typescript
class PairProgramming {
  static readonly STYLES = {
    driverNavigator: {
      name: 'Driver-Navigator',
      description: 'One types (driver), one reviews (navigator)',
      switchInterval: '15-30 minutes',
      bestFor: 'Complex problems, learning'
    },
    pingPong: {
      name: 'Ping-Pong',
      description: 'One writes test, other implements, switch',
      switchInterval: 'Per test case',
      bestFor: 'TDD, equal skill level'
    },
    strongStyle: {
      name: 'Strong-Style',
      description: 'Navigator dictates, driver types',
      switchInterval: '5-10 minutes',
      bestFor: 'Teaching, knowledge transfer'
    }
  };
  
  static planSession(participants: string[], task: string, style: string): PairSession {
    return {
      participants,
      task,
      style,
      startTime: new Date(),
      switchTimes: this.calculateSwitchTimes(style),
      goals: [
        'Complete the task',
        'Share knowledge',
        'Maintain code quality'
      ]
    };
  }
  
  private static calculateSwitchTimes(style: string): Date[] {
    const switches: Date[] = [];
    const now = new Date();
    const interval = style === 'Strong-Style' ? 10 : 20;
    
    for (let i = 1; i <= 4; i++) {
      const switchTime = new Date(now.getTime() + i * interval * 60000);
      switches.push(switchTime);
    }
    
    return switches;
  }
}

interface PairSession {
  participants: string[];
  task: string;
  style: string;
  startTime: Date;
  switchTimes: Date[];
  goals: string[];
}
```

---

## Git Workflows für Teams

### Feature Branch Workflow

```typescript
class GitWorkflow {
  static featureBranchWorkflow(): GitWorkflowGuide {
    return {
      name: 'Feature Branch Workflow',
      steps: [
        {
          step: 1,
          description: 'Create feature branch from develop',
          commands: [
            'git checkout develop',
            'git pull origin develop',
            'git checkout -b feature/user-authentication'
          ]
        },
        {
          step: 2,
          description: 'Work on feature with frequent commits',
          commands: [
            'git add .',
            'git commit -m "feat: add login form component"',
            'git commit -m "feat: implement JWT authentication"',
            'git commit -m "test: add auth tests"'
          ]
        },
        {
          step: 3,
          description: 'Keep branch updated',
          commands: [
            'git checkout develop',
            'git pull origin develop',
            'git checkout feature/user-authentication',
            'git rebase develop',
            '# or: git merge develop'
          ]
        },
        {
          step: 4,
          description: 'Push and create Pull Request',
          commands: [
            'git push origin feature/user-authentication',
            '# Create PR on GitHub/GitLab'
          ]
        },
        {
          step: 5,
          description: 'After approval, merge to develop',
          commands: [
            '# Merge via PR interface',
            'git checkout develop',
            'git pull origin develop',
            'git branch -d feature/user-authentication'
          ]
        }
      ],
      branchingModel: {
        main: 'Production-ready code',
        develop: 'Integration branch',
        'feature/*': 'New features',
        'bugfix/*': 'Bug fixes',
        'hotfix/*': 'Emergency fixes',
        'release/*': 'Release preparation'
      }
    };
  }
  
  static commitConventions(): CommitConvention {
    return {
      format: '<type>(<scope>): <subject>',
      types: {
        feat: 'New feature',
        fix: 'Bug fix',
        docs: 'Documentation',
        style: 'Formatting, no code change',
        refactor: 'Code refactoring',
        test: 'Adding tests',
        chore: 'Maintenance tasks'
      },
      examples: [
        'feat(auth): add JWT token validation',
        'fix(cart): resolve checkout calculation error',
        'docs(api): update endpoint documentation',
        'test(user): add integration tests for user service'
      ]
    };
  }
}

interface GitWorkflowGuide {
  name: string;
  steps: WorkflowStep[];
  branchingModel: Record<string, string>;
}

interface WorkflowStep {
  step: number;
  description: string;
  commands: string[];
}

interface CommitConvention {
  format: string;
  types: Record<string, string>;
  examples: string[];
}
```

### Pull Request Template

```markdown
# Pull Request Template

## Description
<!-- Describe your changes in detail -->

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issue
Closes #(issue number)

## Changes Made
- 
- 
- 

## How Has This Been Tested?
- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tests added/updated
- [ ] All tests passing
- [ ] No merge conflicts

## Screenshots (if applicable)

## Additional Notes
```

---

## Code Review & Quality

### Code Review Guidelines

```typescript
class CodeReview {
  static guidelines = {
    purpose: [
      'Catch bugs early',
      'Maintain code quality',
      'Share knowledge',
      'Ensure consistency'
    ],
    
    checklist: [
      'Code is understandable',
      'No obvious bugs',
      'Tests are adequate',
      'Follows style guide',
      'No security issues',
      'Performance considered',
      'Documentation updated'
    ],
    
    bestPractices: [
      'Be respectful and constructive',
      'Focus on code, not person',
      'Ask questions, don\'t demand',
      'Suggest alternatives',
      'Praise good work',
      'Review promptly'
    ]
  };
  
  static reviewPR(pr: PullRequest): ReviewFeedback {
    const feedback: ReviewFeedback = {
      approved: false,
      comments: [],
      requestedChanges: []
    };
    
    // Check for common issues
    if (pr.changedFiles > 20) {
      feedback.comments.push({
        type: 'suggestion',
        message: 'Consider splitting this PR into smaller chunks'
      });
    }
    
    if (!pr.hasTests) {
      feedback.requestedChanges.push({
        type: 'required',
        message: 'Please add tests for the new functionality'
      });
    }
    
    if (!pr.hasDescription) {
      feedback.requestedChanges.push({
        type: 'required',
        message: 'Please add a description of what this PR does'
      });
    }
    
    // Approve if no required changes
    feedback.approved = feedback.requestedChanges.length === 0;
    
    return feedback;
  }
}

interface PullRequest {
  id: number;
  title: string;
  description?: string;
  author: string;
  changedFiles: number;
  additions: number;
  deletions: number;
  hasTests: boolean;
  hasDescription: boolean;
}

interface ReviewFeedback {
  approved: boolean;
  comments: ReviewComment[];
  requestedChanges: ReviewComment[];
}

interface ReviewComment {
  type: 'suggestion' | 'required' | 'praise';
  message: string;
  file?: string;
  line?: number;
}
```

### Code Quality Tools

```typescript
// ESLint Configuration
const eslintConfig = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'prettier'
  ],
  rules: {
    'no-console': 'warn',
    'no-unused-vars': 'error',
    'prefer-const': 'error',
    '@typescript-eslint/explicit-function-return-type': 'warn',
    'react/prop-types': 'off'
  }
};

// Prettier Configuration
const prettierConfig = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 100,
  tabWidth: 2,
  arrowParens: 'avoid'
};

// Husky Pre-commit Hook
const huskyPreCommit = `
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run lint
npm run test
npm run type-check
`;

// Package.json scripts
const packageScripts = {
  lint: 'eslint src --ext .ts,.tsx',
  'lint:fix': 'eslint src --ext .ts,.tsx --fix',
  format: 'prettier --write "src/**/*.{ts,tsx,css}"',
  'type-check': 'tsc --noEmit',
  test: 'jest',
  'test:watch': 'jest --watch',
  'test:coverage': 'jest --coverage'
};
```

---

## Kommunikation & Dokumentation

### Dokumentations-Standards

```typescript
class Documentation {
  static readonly TEMPLATES = {
    readme: `
# Project Name

## Description
Brief project description

## Prerequisites
- Node.js >= 18
- PostgreSQL >= 14
- Docker (optional)

## Installation
\`\`\`bash
npm install
cp .env.example .env
npm run db:migrate
\`\`\`

## Usage
\`\`\`bash
npm run dev
\`\`\`

## Testing
\`\`\`bash
npm run test
\`\`\`

## Deployment
Instructions for deployment

## Contributing
See CONTRIBUTING.md

## License
MIT
`,
    
    api: `
# API Documentation

## Authentication
All API requests require authentication via JWT token.

## Endpoints

### GET /api/users
Get all users

**Parameters:**
- \`page\` (optional): Page number
- \`limit\` (optional): Items per page

**Response:**
\`\`\`json
{
  "users": [...],
  "total": 100,
  "page": 1
}
\`\`\`

### POST /api/users
Create new user

**Body:**
\`\`\`json
{
  "name": "John Doe",
  "email": "john@example.com"
}
\`\`\`
`,
    
    architecture: `
# Architecture

## Overview
High-level architecture description

## Technology Stack
- Frontend: React + TypeScript
- Backend: Node.js + Express
- Database: PostgreSQL
- Cache: Redis
- Queue: Bull

## System Design
[Architecture diagram]

## Data Flow
1. User request
2. API Gateway
3. Service Layer
4. Database

## Security
- JWT Authentication
- Rate Limiting
- Input Validation
- SQL Injection Prevention
`
  };
  
  static generateJSDoc(functionName: string, params: Parameter[], returnType: string): string {
    let doc = `/**\n`;
    doc += ` * ${functionName}\n`;
    doc += ` * \n`;
    
    params.forEach(param => {
      doc += ` * @param {${param.type}} ${param.name} - ${param.description}\n`;
    });
    
    doc += ` * @returns {${returnType}}\n`;
    doc += ` * @example\n`;
    doc += ` * ${functionName}(${params.map(p => p.example).join(', ')})\n`;
    doc += ` */\n`;
    
    return doc;
  }
}

interface Parameter {
  name: string;
  type: string;
  description: string;
  example: string;
}
```

### Meeting Notes Template

```typescript
interface MeetingNotes {
  date: Date;
  attendees: string[];
  agenda: string[];
  discussion: DiscussionPoint[];
  decisions: Decision[];
  actionItems: ActionItem[];
  nextMeeting?: Date;
}

interface DiscussionPoint {
  topic: string;
  summary: string;
  duration: number;
}

interface Decision {
  topic: string;
  decision: string;
  rationale: string;
  votedBy?: string[];
}

interface ActionItem {
  description: string;
  assignee: string;
  dueDate: Date;
  status: 'todo' | 'in-progress' | 'done';
}

class MeetingManager {
  createMeetingNotes(data: Partial<MeetingNotes>): string {
    let notes = `# Meeting Notes - ${data.date?.toLocaleDateString()}\n\n`;
    
    notes += `## Attendees\n`;
    data.attendees?.forEach(person => {
      notes += `- ${person}\n`;
    });
    
    notes += `\n## Agenda\n`;
    data.agenda?.forEach((item, idx) => {
      notes += `${idx + 1}. ${item}\n`;
    });
    
    notes += `\n## Discussion\n`;
    data.discussion?.forEach(point => {
      notes += `### ${point.topic}\n`;
      notes += `${point.summary}\n\n`;
    });
    
    notes += `\n## Decisions\n`;
    data.decisions?.forEach(decision => {
      notes += `### ${decision.topic}\n`;
      notes += `**Decision:** ${decision.decision}\n`;
      notes += `**Rationale:** ${decision.rationale}\n\n`;
    });
    
    notes += `\n## Action Items\n`;
    data.actionItems?.forEach(item => {
      notes += `- [ ] ${item.description} (@${item.assignee}, due: ${item.dueDate.toLocaleDateString()})\n`;
    });
    
    if (data.nextMeeting) {
      notes += `\n## Next Meeting\n`;
      notes += `${data.nextMeeting.toLocaleDateString()}\n`;
    }
    
    return notes;
  }
}
```

---

## Projektmanagement

### Task Tracking

```typescript
class TaskBoard {
  private tasks: Map<string, Task> = new Map();
  
  createTask(task: Omit<Task, 'id' | 'createdAt'>): string {
    const id = `TASK-${Date.now()}`;
    const newTask: Task = {
      id,
      ...task,
      createdAt: new Date(),
      status: 'todo'
    };
    
    this.tasks.set(id, newTask);
    return id;
  }
  
  updateTaskStatus(id: string, status: TaskStatus): boolean {
    const task = this.tasks.get(id);
    if (!task) return false;
    
    task.status = status;
    task.updatedAt = new Date();
    
    if (status === 'done') {
      task.completedAt = new Date();
    }
    
    return true;
  }
  
  assignTask(id: string, assignee: string): boolean {
    const task = this.tasks.get(id);
    if (!task) return false;
    
    task.assignee = assignee;
    return true;
  }
  
  getTasksByStatus(status: TaskStatus): Task[] {
    return Array.from(this.tasks.values())
      .filter(task => task.status === status);
  }
  
  getTasksByAssignee(assignee: string): Task[] {
    return Array.from(this.tasks.values())
      .filter(task => task.assignee === assignee);
  }
  
  getMetrics(): TaskMetrics {
    const allTasks = Array.from(this.tasks.values());
    
    return {
      total: allTasks.length,
      todo: allTasks.filter(t => t.status === 'todo').length,
      inProgress: allTasks.filter(t => t.status === 'in-progress').length,
      inReview: allTasks.filter(t => t.status === 'in-review').length,
      done: allTasks.filter(t => t.status === 'done').length,
      blocked: allTasks.filter(t => t.status === 'blocked').length,
      averageCompletionTime: this.calculateAverageCompletionTime(allTasks)
    };
  }
  
  private calculateAverageCompletionTime(tasks: Task[]): number {
    const completed = tasks.filter(t => t.completedAt && t.createdAt);
    
    if (completed.length === 0) return 0;
    
    const totalTime = completed.reduce((sum, task) => {
      const time = task.completedAt!.getTime() - task.createdAt.getTime();
      return sum + time;
    }, 0);
    
    return totalTime / completed.length / (1000 * 60 * 60 * 24); // days
  }
}

type TaskStatus = 'todo' | 'in-progress' | 'in-review' | 'blocked' | 'done';

interface Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
  priority: 'low' | 'medium' | 'high' | 'critical';
  assignee?: string;
  estimatedHours: number;
  actualHours?: number;
  tags: string[];
  createdAt: Date;
  updatedAt?: Date;
  completedAt?: Date;
  blockedReason?: string;
}

interface TaskMetrics {
  total: number;
  todo: number;
  inProgress: number;
  inReview: number;
  done: number;
  blocked: number;
  averageCompletionTime: number;
}
```

---

## Testing & CI/CD im Team

### Test Strategy

```typescript
class TestStrategy {
  static readonly PYRAMID = {
    unit: {
      percentage: 70,
      description: 'Fast, isolated tests',
      tools: ['Jest', 'Vitest'],
      coverage: '> 80%'
    },
    integration: {
      percentage: 20,
      description: 'Test component interactions',
      tools: ['Jest', 'Testing Library'],
      coverage: '> 60%'
    },
    e2e: {
      percentage: 10,
      description: 'Full user flows',
      tools: ['Playwright', 'Cypress'],
      coverage: 'Critical paths'
    }
  };
  
  static createTestPlan(features: string[]): TestPlan {
    return {
      unitTests: features.map(f => `Unit tests for ${f}`),
      integrationTests: features.map(f => `Integration tests for ${f}`),
      e2eTests: [
        'User registration flow',
        'Login and authentication',
        'Purchase flow',
        'Admin operations'
      ],
      performanceTests: [
        'Load testing with 1000 concurrent users',
        'API response time < 200ms'
      ],
      securityTests: [
        'SQL injection prevention',
        'XSS protection',
        'CSRF protection'
      ]
    };
  }
}

interface TestPlan {
  unitTests: string[];
  integrationTests: string[];
  e2eTests: string[];
  performanceTests: string[];
  securityTests: string[];
}
```

### CI/CD Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:coverage
      - uses: codecov/codecov-action@v3

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build

  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "Deploy to staging"

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    needs: build
    environment: production
    steps:
      - run: echo "Deploy to production"
```

---

## Deployment & Operations

### Deployment Checklist

```typescript
class DeploymentChecklist {
  static readonly PRE_DEPLOYMENT = [
    'All tests passing',
    'Code review completed',
    'Documentation updated',
    'Database migrations ready',
    'Environment variables configured',
    'Backup created',
    'Rollback plan documented'
  ];
  
  static readonly DEPLOYMENT = [
    'Run database migrations',
    'Deploy new version',
    'Run smoke tests',
    'Monitor error logs',
    'Check performance metrics',
    'Verify core functionality'
  ];
  
  static readonly POST_DEPLOYMENT = [
    'Monitor for 24 hours',
    'Check analytics',
    'User feedback collection',
    'Document any issues',
    'Update changelog',
    'Notify stakeholders'
  ];
  
  static createDeploymentPlan(version: string): DeploymentPlan {
    return {
      version,
      date: new Date(),
      preDeployment: this.PRE_DEPLOYMENT,
      deployment: this.DEPLOYMENT,
      postDeployment: this.POST_DEPLOYMENT,
      rollbackPlan: this.createRollbackPlan(version),
      risks: this.assessRisks()
    };
  }
  
  private static createRollbackPlan(version: string): string[] {
    return [
      'Stop new deployment',
      `Revert to previous version`,
      'Rollback database migrations',
      'Clear cache',
      'Verify rollback success',
      'Notify team'
    ];
  }
  
  private static assessRisks(): Risk[] {
    return [
      {
        description: 'Database migration failure',
        probability: 'low',
        impact: 'high',
        mitigation: 'Test migrations on staging'
      },
      {
        description: 'Increased error rate',
        probability: 'medium',
        impact: 'medium',
        mitigation: 'Gradual rollout, monitoring'
      }
    ];
  }
}

interface DeploymentPlan {
  version: string;
  date: Date;
  preDeployment: string[];
  deployment: string[];
  postDeployment: string[];
  rollbackPlan: string[];
  risks: Risk[];
}

interface Risk {
  description: string;
  probability: 'low' | 'medium' | 'high';
  impact: 'low' | 'medium' | 'high';
  mitigation: string;
}
```

---

## Konfliktlösung & Best Practices

### Konflikt-Management

```typescript
class ConflictResolution {
  static readonly STRATEGIES = {
    technical: {
      name: 'Technical Disagreements',
      approach: [
        'Present both solutions',
        'List pros/cons',
        'Create proof of concept',
        'Team vote or Tech Lead decision'
      ]
    },
    process: {
      name: 'Process Conflicts',
      approach: [
        'Identify root cause',
        'Discuss in retrospective',
        'Agree on experiment',
        'Review after sprint'
      ]
    },
    interpersonal: {
      name: 'Interpersonal Issues',
      approach: [
        'One-on-one discussion',
        'Focus on behavior, not person',
        'Find common ground',
        'Involve mediator if needed'
      ]
    }
  };
  
  static resolveCodeConflict(file: string, conflictType: string): Resolution {
    return {
      file,
      conflictType,
      steps: [
        'Pull latest changes from main branch',
        'Review conflicting changes',
        'Discuss with team if complex',
        'Resolve conflicts locally',
        'Run tests',
        'Push resolved version'
      ],
      prevention: [
        'Regular rebasing/merging',
        'Smaller, focused PRs',
        'Clear ownership of files',
        'Good communication'
      ]
    };
  }
}

interface Resolution {
  file: string;
  conflictType: string;
  steps: string[];
  prevention: string[];
}
```

### Team Best Practices

```typescript
const teamBestPractices = {
  communication: [
    'Use team chat for quick questions',
    'Document decisions in wiki',
    'Regular sync meetings',
    'Clear and concise messages',
    'Respect time zones'
  ],
  
  codeQuality: [
    'Follow style guide consistently',
    'Write self-documenting code',
    'Add comments for complex logic',
    'Keep functions small and focused',
    'Regular refactoring'
  ],
  
  collaboration: [
    'Review PRs within 24 hours',
    'Constructive feedback only',
    'Help teammates when blocked',
    'Share knowledge regularly',
    'Celebrate team wins'
  ],
  
  productivity: [
    'Focus time: no meetings',
    'Clear task priorities',
    'Minimize context switching',
    'Take regular breaks',
    'Sustainable pace'
  ]
};
```

---

## Beispiel-Projekt: E-Commerce Platform

### Project Overview

```typescript
const ecommercePlatform = {
  name: 'ShopHub',
  description: 'Modern e-commerce platform',
  
  features: {
    core: [
      'User authentication & authorization',
      'Product catalog with search',
      'Shopping cart management',
      'Checkout & payment processing',
      'Order management',
      'Admin dashboard'
    ],
    additional: [
      'Product reviews & ratings',
      'Wishlist functionality',
      'Email notifications',
      'Inventory management',
      'Analytics dashboard'
    ]
  },
  
  techStack: {
    frontend: ['Next.js 14', 'React', 'TypeScript', 'Tailwind CSS'],
    backend: ['Node.js', 'Express', 'PostgreSQL', 'Prisma'],
    auth: ['NextAuth.js', 'JWT'],
    payment: ['Stripe'],
    deployment: ['Vercel', 'AWS RDS'],
    monitoring: ['Sentry', 'Google Analytics']
  },
  
  team: [
    { name: 'Alice', role: 'Tech Lead & Full-Stack' },
    { name: 'Bob', role: 'Backend Developer' },
    { name: 'Charlie', role: 'Frontend Developer' },
    { name: 'Diana', role: 'Full-Stack Developer' },
    { name: 'Eve', role: 'QA Engineer' }
  ],
  
  timeline: {
    week1_2: 'Project setup & architecture',
    week3_4: 'Authentication & user management',
    week5_6: 'Product catalog & search',
    week7_8: 'Cart & checkout',
    week9_10: 'Admin dashboard',
    week11_12: 'Testing & deployment'
  }
};
```

### Sprint Planning Example

```typescript
class EcommerceSprintPlanning {
  planSprint1(): SprintPlan {
    return {
      sprintNumber: 1,
      goal: 'Setup project infrastructure and implement authentication',
      duration: '2 weeks',
      
      userStories: [
        {
          id: 'US-1',
          title: 'Project Setup',
          description: 'Initialize repository, setup CI/CD, configure tools',
          points: 5,
          tasks: [
            'Create GitHub repository',
            'Setup Next.js project',
            'Configure ESLint & Prettier',
            'Setup CI/CD pipeline',
            'Create project documentation'
          ],
          assignee: 'Alice'
        },
        {
          id: 'US-2',
          title: 'User Registration',
          description: 'Users can create accounts',
          points: 8,
          tasks: [
            'Design database schema',
            'Create user model',
            'Implement registration API',
            'Build registration form',
            'Add form validation',
            'Write tests'
          ],
          assignee: 'Bob'
        },
        {
          id: 'US-3',
          title: 'User Login',
          description: 'Users can log in and access protected routes',
          points: 8,
          tasks: [
            'Setup NextAuth.js',
            'Implement login API',
            'Build login form',
            'Add JWT token handling',
            'Protect routes',
            'Write tests'
          ],
          assignee: 'Charlie'
        }
      ],
      
      totalPoints: 21,
      dailyStandups: '9:00 AM daily',
      reviewDate: new Date('2025-12-13'),
      retroDate: new Date('2025-12-13')
    };
  }
}

interface SprintPlan {
  sprintNumber: number;
  goal: string;
  duration: string;
  userStories: SprintUserStory[];
  totalPoints: number;
  dailyStandups: string;
  reviewDate: Date;
  retroDate: Date;
}

interface SprintUserStory {
  id: string;
  title: string;
  description: string;
  points: number;
  tasks: string[];
  assignee: string;
}
```

---

## Zusammenfassung

Du hast gelernt:

✅ **Projektplanung**: Kickoff, Scope, Timeline
✅ **Repository Setup**: Struktur, Git, CI/CD
✅ **Team-Zusammenarbeit**: Standups, Pair Programming
✅ **Git Workflows**: Feature Branches, Commit Conventions
✅ **Code Review**: Guidelines, Quality Tools
✅ **Kommunikation**: Dokumentation, Meeting Notes
✅ **Projektmanagement**: Task Tracking, Metrics
✅ **Testing**: Test Strategy, CI/CD Pipeline
✅ **Deployment**: Checklist, Rollback Plans
✅ **Konfliktlösung**: Strategien, Best Practices
✅ **Praxis-Beispiel**: E-Commerce Platform

### Erfolgsfaktoren für Teamprojekte

- [ ] Klare Kommunikation
- [ ] Regelmäßiges Feedback
- [ ] Code Quality Standards
- [ ] Gute Dokumentation
- [ ] Effektive Tools
- [ ] Respektvoller Umgang
- [ ] Kontinuierliches Lernen
- [ ] Team Spirit

### Hilfreiche Ressourcen

- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [The Twelve-Factor App](https://12factor.net/)
- [Team Topologies](https://teamtopologies.com/)

Viel Erfolg mit deinen Teamprojekten! 🎉
