# Self-Healing CI - Arquitectura Técnica

## 📐 Visión General de la Arquitectura

```
┌─────────────────────────────────────────────────────────┐
│          SELF-HEALING CI ARCHITECTURE                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  CI/CD Pipeline  ─────webhook────→  Event Handler      │
│                                         │                │
│                                         ▼                │
│                                   Issue Creator         │
│                                  (GitHub API)           │
│                                         │                │
│                                         ▼                │
│  ┌─────────────────────────────────────────────────┐   │
│  │           AGENT SERVICE (Async Queue)           │   │
│  │                                                 │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │  Issue Listener (Webhook/Polling)       │  │   │
│  │  │  ├─ Repository Analyzer                 │  │   │
│  │  │  ├─ Log Parser & Analyzer               │  │   │
│  │  │  ├─ LLM Diagnosis Engine                │  │   │
│  │  │  ├─ Code Fix Generator                  │  │   │
│  │  │  ├─ Git Operations (Clone, Push)        │  │   │
│  │  │  ├─ PR Creator                          │  │   │
│  │  │  └─ Webhook Handler (Issue comments)    │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  │                                                 │   │
│  │  Queue: Redis/RabbitMQ                          │   │
│  │  Store: PostgreSQL (audit trail)                │   │
│  │  Cache: Redis (repo data, LLM responses)        │   │
│  │  Config: Environment + per-repo settings        │   │
│  └─────────────────────────────────────────────────┘   │
│         ▲              ▲              ▲                 │
│         │              │              │                 │
│    ┌────┴──┐      ┌────┴──┐      ┌────┴──┐            │
│    │GitHub │      │  LLM  │      │  Git  │            │
│    │  API  │      │Provider  │      │Repo  │            │
│    └───────┘      └───────┘      └───────┘            │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │      HUMAN REVIEW & APPROVAL WORKFLOW           │   │
│  │      (GitHub Web UI + Email Notifications)      │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## 🏗️ Componentes Principales

### 1. **Webhook Handler** (`src/integrations/webhook_handler.py`)

**Responsabilidad**: Capturar eventos de CI/CD y crear issues

```python
Interface:
  - POST /webhooks/ci-failure
  - Valida firma del webhook
  - Extrae información de error
  - Crea issue en GitHub
  
Entrada:
  {
    "build_id": "build-12345",
    "status": "FAILED",
    "branch": "main",
    "logs": "AssertionError: ...",
    "timestamp": "2026-06-16T14:30:00Z"
  }

Salida:
  {
    "issue_id": "1234567890",
    "issue_url": "https://github.com/...",
    "created": true
  }
```

### 2. **Issue Listener** (`src/agents/issue_listener.py`)

**Responsabilidad**: Escuchar y procesar issues creados

```python
Interface:
  - Webhooks GitHub (issue.opened)
  - Polling sobre issues recientes
  - Dispatch a agente para análisis
  
Flujo:
  1. Recibe evento de issue creado
  2. Valida que sea ci-failure
  3. Extrae metadatos
  4. Enqueue para análisis
  5. Retorna 200 OK
```

### 3. **Log Analyzer** (`src/agents/log_analyzer.py`)

**Responsabilidad**: Analizar y estructurar logs de CI

```python
Class LogAnalyzer:
  def parse_logs(raw_logs: str) -> ParsedLogs
  def extract_error_context(logs) -> ErrorContext
  def identify_test_file(error) -> str
  def get_stack_trace(logs) -> List[str]

ParsedLogs:
  - error_message: str
  - stack_trace: List[str]
  - failed_test: str
  - error_type: str (AssertionError, TypeError, etc)
  - context_lines: List[str]
  - timestamp: datetime
```

### 4. **LLM Diagnosis Engine** (`src/agents/llm_engine.py`)

**Responsabilidad**: Diagnosticar causa raíz usando IA

```python
Class LLMEngine:
  def analyze_error(
    logs: ParsedLogs,
    code_context: str,
    recent_changes: List[Commit]
  ) -> Diagnosis
  
Diagnosis:
  - root_cause: str
  - affected_files: List[str]
  - error_pattern: str
  - confidence: float (0-1)
  - explanation: str
  - proposed_fix_type: str (code_change, config, dependency, etc)

Ejemplo:
  {
    "root_cause": "Off-by-one error in Calculator.add()",
    "affected_files": ["src/calculator.py"],
    "error_pattern": "AssertionError: expected 5 to equal 10",
    "confidence": 0.95,
    "explanation": "La función add() suma 1 de más",
    "proposed_fix_type": "code_change"
  }
```

### 5. **Code Generator** (`src/agents/code_generator.py`)

**Responsabilidad**: Generar código de corrección

```python
Class CodeGenerator:
  def generate_fix(
    diagnosis: Diagnosis,
    affected_files: Dict[str, str]
  ) -> Fix
  
Fix:
  - changes: List[FileChange]
  - commit_message: str
  - validation_status: str (OK, WARNING, ERROR)
  - test_commands: List[str]
  - estimated_confidence: float

FileChange:
  - path: str
  - original: str
  - fixed: str
  - line_range: Tuple[int, int]
  - justification: str
  - diff: str
```

### 6. **Git Operations** (`src/integrations/git_ops.py`)

**Responsabilidad**: Operaciones Git (clone, commit, push)

```python
Class GitOps:
  def clone_repo(repo_url, branch) -> LocalRepo
  def create_branch(base_branch, new_branch_name) -> Branch
  def apply_changes(changes: List[FileChange]) -> bool
  def commit(message: str, author: Author) -> Commit
  def push(branch: str, force: bool = False) -> bool
  
Configuración requerida:
  - GitHub Token con permisos push
  - SSH Key (alternativa)
  - User name y email para commits
```

### 7. **PR Manager** (`src/agents/pr_manager.py`)

**Responsabilidad**: Crear y gestionar Pull Requests

```python
Class PRManager:
  def create_pr(
    title: str,
    description: str,
    head_branch: str,
    base_branch: str,
    reviewers: List[str]
  ) -> PullRequest
  
  def update_pr(pr_number: int, body: str) -> bool
  def request_reviews(pr_number: int, reviewers: List[str]) -> bool
  def mark_draft(pr_number: int) -> bool

PullRequest:
  - number: int
  - title: str
  - description: str
  - url: str
  - draft: bool
  - created_at: datetime
```

### 8. **GitHub API Wrapper** (`src/integrations/github_api.py`)

**Responsabilidad**: Interfaz con GitHub REST API

```python
Class GitHubAPI:
  # Issues
  def get_issue(issue_id) -> Issue
  def create_issue(title, body, labels) -> Issue
  def add_comment(issue_id, body) -> Comment
  def update_issue(issue_id, state, labels) -> Issue
  
  # PRs
  def get_pr(pr_number) -> PullRequest
  def create_pr(title, head, base, body) -> PullRequest
  def merge_pr(pr_number, commit_message) -> bool
  def get_pr_checks(pr_number) -> List[Check]
  
  # Repository
  def get_file(path, ref) -> str
  def list_commits(since, until) -> List[Commit]
  def get_repo_info() -> Repository
```

## 📊 Flujo de Datos

### Fase 1: Captura de Fallo

```
CI Pipeline (falla)
    ↓
Webhook Payload
    {
      "build_id": "12345",
      "status": "FAILED",
      "logs": "[error details]",
      "branch": "main"
    }
    ↓
Webhook Handler
    ↓
Create GitHub Issue
    ↓
Issue #1234 created with labels: [ci-failure, auto-healing]
```

### Fase 2: Análisis

```
GitHub Issue Created Event
    ↓
Issue Listener
    ↓
Enqueue to Redis Queue
    ↓
Agent Worker dequeue
    ↓
LogAnalyzer.parse_logs()
    {
      "error_type": "AssertionError",
      "error_message": "expected 5 to equal 10",
      "failed_test": "test_calculator.py::test_add",
      "stack_trace": [...]
    }
    ↓
GitHubAPI.get_file("src/calculator.py")
    ↓
GitHubAPI.list_commits(limit=10)
    ↓
LLMEngine.analyze_error()
    {
      "root_cause": "Off-by-one error",
      "confidence": 0.95,
      "affected_files": ["src/calculator.py"]
    }
    ↓
Database: Audit log de análisis
```

### Fase 3: Generación

```
Diagnosis Result
    ↓
CodeGenerator.generate_fix()
    {
      "changes": [
        {
          "file": "src/calculator.py",
          "original": "return a + b + 1",
          "fixed": "return a + b"
        }
      ],
      "commit_message": "Fix: off-by-one error"
    }
    ↓
Validación de sintaxis y scope
    ↓
¿Confidence > threshold?
    ├─ NO → Mark issue as "needs-human-review"
    └─ SÍ → Continuar
    
    ↓
Database: Store fix proposal
```

### Fase 4: Git Operations

```
Fix Generation Approved
    ↓
GitOps.clone_repo()
    ↓
git checkout -b fix/ai-heal-1234
    ↓
Apply file changes
    ↓
git commit -m "Fix: off-by-one error"
    ↓
git push origin fix/ai-heal-1234
    ↓
Database: Log operación
```

### Fase 5: PR Creation

```
Branch pushed successfully
    ↓
PRManager.create_pr(
  title: "🤖 AI Auto-Fix: Resolve unit test failure",
  description: "[detailed description]",
  head: "fix/ai-heal-1234",
  base: "main"
)
    ↓
GitHub PR #456 created
    ↓
Request reviews from: [@tech-lead, @qa-team]
    ↓
Add labels: [ai-generated, needs-review]
    ↓
Notify developers
```

### Fase 6: CI Validation

```
PR created (GitHub webhook)
    ↓
CI/CD Pipeline triggered
    ↓
Run tests on PR branch
    ↓
Test Results returned to GitHub
    ↓
GitHub Webhook: check_run completed
    ↓
Agent listener receives
    ├─ PASS ✅ → Wait for human review
    ├─ FAIL ❌ → Re-analyze + retry
    └─ TIMEOUT ⏱️ → Notify human
```

### Fase 7: Human Review

```
PR ready for review
    ↓
Notification sent to reviewers
    ↓
Reviewer opens PR on GitHub UI
    ↓
Reviewer reviews changes + tests
    ↓
Reviewer action:
    ├─ APPROVE ✅
    ├─ REQUEST CHANGES 🔄
    └─ REJECT ❌
    
    ↓
GitHub Webhook: pull_request_review
    ↓
Agent listener processes
```

### Fase 8: Merge & Resolution

```
PR approved
    ↓
¿Auto-merge eligibility?
    ├─ YES (conf > 90%) → Auto-merge
    └─ NO → Wait for manual merge
    
    ↓
GitHub merge_commit
    ↓
PR merged to main
    ↓
Close GitHub Issue #1234
    ↓
Update labels: [ai-fixed, resolved]
    ↓
Database: Complete audit log
    ↓
✅ Resolution Complete
```

## 🔄 Patrones de Integración

### Pattern 1: Webhook Listener

```python
# Configurar en CI/CD pipeline
POST https://self-healing-agent.example.com/webhooks/ci-failure
Headers: {
  "X-Signature": "sha256=...",
  "Content-Type": "application/json"
}
Body: {
  "build_id": "12345",
  "status": "FAILED",
  "logs": "..."
}

# Handler
@app.post("/webhooks/ci-failure")
async def handle_ci_failure(payload: dict):
    signature = request.headers.get("X-Signature")
    if not verify_signature(signature, payload):
        return 401
    
    issue = await create_issue(payload)
    return {"issue_id": issue.id}
```

### Pattern 2: Async Job Queue

```python
# Enqueue
redis_queue.enqueue(
    'analyze_issue',
    issue_id=1234567890,
    repo='bdlozano/pipeline_alm_IA'
)

# Worker
@worker.job
def analyze_issue(issue_id, repo):
    issue = github_api.get_issue(issue_id)
    logs = extract_logs(issue.body)
    
    diagnosis = llm_engine.analyze(logs)
    if diagnosis.confidence > THRESHOLD:
        fix = code_generator.generate(diagnosis)
        enqueue('apply_fix', fix=fix)
```

### Pattern 3: Webhook Polling Fallback

```python
# Si webhooks falla, poll periódicamente
async def poll_for_issues():
    issues = github_api.list_issues(
        labels=['ci-failure'],
        state='open',
        updated_since=timedelta(minutes=5)
    )
    for issue in issues:
        if not is_processed(issue):
            enqueue('analyze_issue', issue_id=issue.id)

# Ejecutar cada 5 minutos
scheduler.add_job(poll_for_issues, 'interval', minutes=5)
```

## 📝 Modelos de Datos

```python
# Models en src/models.py

class Issue:
    id: int
    number: int
    title: str
    body: str
    labels: List[str]
    created_at: datetime
    updated_at: datetime
    state: str  # open, closed
    assignees: List[str]

class ParsedLogs:
    error_message: str
    stack_trace: List[str]
    error_type: str
    context_lines: List[str]
    failed_test: str
    failed_file: str

class Diagnosis:
    root_cause: str
    affected_files: List[str]
    error_pattern: str
    confidence: float
    explanation: str
    proposed_fix_type: str
    severity: str  # critical, high, medium, low

class Fix:
    id: str  # UUID
    changes: List[FileChange]
    commit_message: str
    validation_status: str
    test_commands: List[str]
    estimated_confidence: float

class FileChange:
    path: str
    original: str
    fixed: str
    line_range: Tuple[int, int]
    justification: str
    diff: str

class PullRequest:
    number: int
    title: str
    description: str
    url: str
    head_branch: str
    base_branch: str
    draft: bool
    reviewers: List[str]
    created_at: datetime
    merged_at: Optional[datetime]

class AuditLog:
    id: str
    timestamp: datetime
    action: str  # issue_created, analysis_started, fix_applied, etc
    issue_id: int
    pr_number: Optional[int]
    status: str  # success, failure, pending
    details: Dict
```

## 🔐 Reglas de Seguridad

Ver `src/rules/safety_rules.py`:

```python
class SafetyRules:
    
    # Cambios permitidos
    ALLOWED_FILE_PATTERNS = [
        'src/**/*.py',
        'tests/**/*.py',
        'config/**/*.yml',
        'requirements.txt'
    ]
    
    # Cambios prohibidos
    FORBIDDEN_FILE_PATTERNS = [
        '.github/workflows/**',
        'docker/**',
        'terraform/**',
        '*.key',
        '*.pem'
    ]
    
    # Límites
    MAX_FILES_CHANGED = 5
    MAX_LINES_CHANGED = 500
    MAX_RETRIES = 3
    MIN_CONFIDENCE = 0.60
    
    @staticmethod
    def validate_changes(changes: List[FileChange]) -> bool:
        for change in changes:
            if not SafetyRules.is_file_allowed(change.path):
                return False
            if len(change.diff.split('\n')) > SafetyRules.MAX_LINES_CHANGED:
                return False
        return True
    
    @staticmethod
    def is_file_allowed(path: str) -> bool:
        # Check against patterns
        pass
```

## 📚 Stack Tecnológico

| Componente | Tecnología | Propósito |
|-----------|-----------|----------|
| **Runtime** | Python 3.9+ | Lenguaje principal |
| **Framework Web** | FastAPI | Webhooks y API |
| **Queue** | Redis/RabbitMQ | Jobs async |
| **Database** | PostgreSQL | Audit logs, persistencia |
| **Cache** | Redis | Caching de datos |
| **LLM** | Anthropic Claude / OpenAI GPT-4 | IA para diagnóstico |
| **Git** | GitPython | Operaciones Git |
| **GitHub API** | PyGithub | Interacción con GitHub |
| **Testing** | pytest | Tests unitarios |
| **Logging** | Python logging | Observabilidad |
| **Container** | Docker | Deployment |
| **Orchestration** | Docker Compose / K8s | Orquestación |

## 🚀 Deployment

### Local Development
```bash
docker-compose up -d
# Services: agent, redis, postgres, webhook-server
```

### Production
```bash
# Deploy con: AWS ECS, Google Cloud Run, Azure Container Instances, K8s
docker build -t self-healing-ci:latest .
docker push registry.example.com/self-healing-ci:latest
```

---

**Arquitectura actualizada**: Junio 2026
