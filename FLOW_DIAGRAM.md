# Self-Healing CI - Diagrama de Flujo

## 🔄 Flujo Principal ASCII

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     SISTEMA DE SELF-HEALING CI CON IA                   │
└─────────────────────────────────────────────────────────────────────────┘

    ┌──────────────┐
    │  CI/CD FALLA │
    └──────┬───────┘
           │
           ▼
    ┌──────────────────────────────────────────────┐
    │  WEBHOOK DETECTOR                            │
    │  - Captura evento de fallo en CI             │
    │  - Extrae: logs, código, configuración       │
    │  - Valida payload                            │
    └──────────────────┬───────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────┐
    │  CREAR ISSUE EN GITHUB                       │
    │  - Título: "🔴 CI Failure: [Test Name]"      │
    │  - Body: logs + stack trace + detalles       │
    │  - Labels: auto-healing, priority            │
    │  - Metadata: timestamps, build info          │
    └──────────────────┬───────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────┐
    │  🤖 AGENTE IA - FASE 1: ANÁLISIS             │
    │  (Escucha webhook/polling en issue)          │
    │  ├─ Descarga logs de CI                      │
    │  ├─ Analiza stack trace                      │
    │  ├─ Examina archivos afectados               │
    │  ├─ Busca commits recientes                  │
    │  └─ Diagnostica causa raíz                   │
    │                                              │
    │  📊 Output:                                  │
    │  {                                           │
    │    "root_cause": "Off-by-one error",        │
    │    "affected_files": ["src/calc.py"],       │
    │    "confidence": 0.95                       │
    │  }                                           │
    └──────────────────┬───────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────┐
    │  🤖 AGENTE IA - FASE 2: GENERACIÓN           │
    │  ├─ Genera código de corrección              │
    │  ├─ Valida sintaxis                          │
    │  ├─ Verifica que no hay side effects         │
    │  └─ Prepara cambios para commit              │
    │                                              │
    │  📊 Output:                                  │
    │  {                                           │
    │    "changes": [...],                        │
    │    "commit_message": "Fix: ...",            │
    │    "validation": "OK"                       │
    │  }                                           │
    └──────────────────┬───────────────────────────┘
                       │
           ┌───────────┴──────────────┐
           │                          │
           ▼                          ▼
    ┌──────────────┐          ┌──────────────────────────┐
    │ ¿CONFIANZA?  │          │ CREAR RAMA Y PUSHEAR     │
    │ > threshold  │          │ git checkout -b fix/...  │
    │ (default 80%)│          │ git commit               │
    └──────┬───────┘          │ git push                 │
           │                  └──────┬───────────────────┘
        NO │                         │
           │                         ▼
           │            ┌──────────────────────────┐
           │            │ 🔀 CREAR PULL REQUEST    │
           │            │                          │
           │            │ Datos del PR:           │
           │            │ - Título automático     │
           │            │ - Descripción detallada │
           │            │ - Vincula issue         │
           │            │ - Asigna reviewers      │
           │            │ - Labels: ai-fix        │
           │            │ - No draft (conf > 80%) │
           │            └──────┬───────────────────┘
           │                   │
           │                   ▼
           │    ┌──────────────────────────────┐
           │    │ 🧪 EJECUTAR TESTS EN PR      │
           │    │                              │
           │    │ GitHub Actions:             │
           │    │ ├─ Unit tests               │
           │    │ ├─ Integration tests        │
           │    │ ├─ Lint & format checks     │
           │    │ ├─ Security scans           │
           │    │ └─ Coverage validation      │
           │    └──────┬───────────────────────┘
           │           │
           │ ┌─────────┼─────────┬──────────────┐
           │ │         │         │              │
        ✅ │ │     ⚠️ FAIL   ⏹️ TIMEOUT  ❌ ERROR
        PASS│ │         │         │              │
           │ │         ▼         ▼              ▼
           │ │    ┌────────┐ ┌──────────┐ ┌──────────┐
           │ │    │REANALYZE│ │NOTIFICAR │ │ESCALAR   │
           │ │    │+ RETRY  │ │HUMANO    │ │REVISOR   │
           │ │    └────────┘ └──────────┘ └──────────┘
           │ │
           └─┼─────────────────────────────────┐
             │                                 │
             ▼                                 ▼
    ┌──────────────┐              ┌──────────────────────┐
    │ ESPERANDO    │              │  REVIEW DRAFT        │
    │ REVISIÓN     │              │  Confianza < 80%     │
    │ HUMANA       │              │  Requiere revisión   │
    └──────┬───────┘              │  antes de merge      │
           │                      └──────┬───────────────┘
           │                             │
    ┌──────┴─────────┬──────────────┬───┴─────────┐
    │                │              │             │
 APPROVE         REQUEST         REJECT        NO ACTION
 CAMBIOS         CAMBIOS         CAMBIOS       (24h timeout)
    │                │              │             │
    ▼                ▼              ▼             ▼
┌────────┐   ┌──────────────┐  ┌────────┐  ┌──────────┐
│ MERGE  │   │ AGENTE REVISA│  │CIERRA  │  │ ESCALADA │
│ a MAIN │   │ + ITERA      │  │ISSUE   │  │ A HUMANO │
│        │   │ (Back to 1)  │  │RECHAZADO│ │          │
└────┬───┘   └──────────────┘  └────────┘  └──────────┘
     │
     ▼
┌──────────────────────┐
│ ✅ RESUELTO          │
│ - Issue closed       │
│ - PR merged          │
│ - Commit en main     │
│ - Audit log creado   │
└──────────────────────┘
```

## 🔀 Decisiones del Agente

```
┌─────────────────────────────────────────────────────────┐
│        ÁRBOL DE DECISIONES DEL AGENTE                   │
└─────────────────────────────────────────────────────────┘

Inicio: Issue creado
    │
    ├─→ ¿Es CI failure?
    │   ├─ NO → Ignorar
    │   └─ SÍ ↓
    │
    ├─→ ¿Contiene logs suficientes?
    │   ├─ NO → Comentar + Esperar
    │   └─ SÍ ↓
    │
    ├─→ Analizar causa raíz
    │   └─ ↓
    │
    ├─→ ¿Confianza > 60%?
    │   ├─ NO → Comentar + Marcar como "needs-human-review"
    │   └─ SÍ ↓
    │
    ├─→ Generar código de corrección
    │   └─ ↓
    │
    ├─→ ¿Cambios son seguros?
    │   ├─ NO (many files, critical paths) → Comentar + Esperar
    │   └─ SÍ ↓
    │
    ├─→ ¿Confianza > 80%?
    │   ├─ NO → Crear PR en DRAFT
    │   └─ SÍ ↓
    │
    ├─→ Crear rama + push
    │   └─ ↓
    │
    ├─→ Crear PR (sin draft)
    │   └─ ↓
    │
    ├─→ Esperar tests (hasta 60 min)
    │   │
    │   ├─ PASS ↓
    │   ├─ FAIL → Reanalyze + Retry
    │   └─ TIMEOUT → Notificar humano
    │
    └─→ Esperar revisión humana (24h timeout)
        │
        ├─ APPROVE → Auto-merge si conf > 90%
        ├─ REQUEST CHANGES → Agente itera
        └─ REJECT → Cerrar issue, crear audit log
```

## 📊 Diagrama de Handoffs

```
┌─────────────────────────────────────────────────────────┐
│              TRANSFERENCIAS ENTRE SISTEMAS              │
└─────────────────────────────────────────────────────────┘

    CI/CD Pipeline
         │
         │ (webhook: build failed)
         ▼
    ┌─────────────────────────┐
    │  Webhook Handler        │  ← Datos: logs, error, commit
    └──────────┬──────────────┘
               │
               │ (create issue API)
               ▼
    ┌─────────────────────────┐
    │  GitHub API             │
    │  (Issue Creator)        │  → Issue #1234 created
    └──────────┬──────────────┘
               │
               │ (webhook: issue created)
               ▼
    ┌─────────────────────────┐
    │  🤖 Agente IA           │  ← Lee issue, descarga contexto
    │  (Listener Service)     │
    └──────────┬──────────────┘
               │
               ├─→ (GET /repos/.../contents)
               │   ▼
               │   GitHub API → Código fuente
               │
               ├─→ (POST /completions)
               │   ▼
               │   LLM Provider → Análisis + Fix
               │
               └─→ (git clone/push)
                   ▼
                   Git Server → Rama con cambios
                   
               ▼
    ┌─────────────────────────┐
    │  GitHub API             │
    │  (PR Creator)           │  → PR #4567 created
    └──────────┬──────────────┘
               │
               │ (webhook: pull_request created)
               ▼
    ┌─────────────────────────┐
    │  CI/CD Pipeline         │  → Run tests on PR
    └──────────┬──────────────┘
               │
               │ (webhook: check_run completed)
               ▼
    ┌─────────────────────────┐
    │  🤖 Agente IA           │  ← Verifica test results
    │  (Result Listener)      │
    └──────────┬──────────────┘
               │
               ├─ Tests PASS → Esperar revisión humana
               └─ Tests FAIL → Reintenta análisis
               
               ▼
    ┌─────────────────────────┐
    │  GitHub Web UI          │
    │  (Human Reviewer)       │  ← Notificación + PR link
    └──────────┬──────────────┘
               │
               │ (PR review action: approved)
               ▼
    ┌─────────────────────────┐
    │  GitHub API             │
    │  (Merge Service)        │  → PR merged, issue closed
    └─────────────────────────┘
```

## 🎯 Estados del Sistema

```
┌─────────────────────────────────────────────────────────┐
│           MÁQUINA DE ESTADOS DEL AGENTE                 │
└─────────────────────────────────────────────────────────┘

    [IDLE]
      │
      │ (issue: ci-failure created)
      ▼
    [ANALYZING]
      │
      ├─ Fallo en diagnóstico? → [WAITING_HUMAN]
      └─ Diagnóstico OK? → [GENERATING]
      
    [GENERATING]
      │
      ├─ Generación falló? → [WAITING_HUMAN]
      ├─ Confianza baja? → [CREATING_DRAFT_PR]
      └─ Generación OK? → [CREATING_BRANCH]
      
    [CREATING_BRANCH]
      │
      ├─ Error de git? → [WAITING_HUMAN]
      └─ Rama creada? → [PUSHING_CODE]
      
    [PUSHING_CODE]
      │
      ├─ Error en push? → [WAITING_HUMAN]
      └─ Push OK? → [CREATING_PR]
      
    [CREATING_PR]
      │
      ├─ Error de API? → [WAITING_HUMAN]
      └─ PR creado? → [WAITING_TESTS]
      
    [CREATING_DRAFT_PR]
      │
      └─ PR en DRAFT → [WAITING_HUMAN]
      
    [WAITING_TESTS]
      │
      ├─ Tests PASS → [WAITING_REVIEW]
      ├─ Tests FAIL → [ANALYZING] (re-attempt)
      └─ Timeout → [WAITING_HUMAN]
      
    [WAITING_REVIEW]
      │
      ├─ Aprobado? → [MERGING]
      ├─ Cambios solicitados? → [GENERATING]
      ├─ Rechazado? → [CLOSED]
      └─ Timeout (24h)? → [ESCALATING]
      
    [MERGING]
      │
      ├─ Merge OK? → [COMPLETED]
      └─ Merge falló? → [WAITING_HUMAN]
      
    [COMPLETED] / [CLOSED] / [WAITING_HUMAN] / [ESCALATING]
      │
      └─→ [IDLE]
```

## 📋 Secuencia de Eventos Típica

```
Time  | Component      | Action
------+----------------+-------------------------------------------
00:00 | CI Pipeline    | ❌ Tests fallan
00:01 | Webhook        | 📤 POST a GitHub
00:02 | GitHub API     | ✅ Issue #123 creado
00:03 | Issue Listener | 🔔 Notificación recibida
00:04 | Log Analyzer   | 📖 Descargando logs
00:05 | Log Analyzer   | 🔍 Analizando error
00:06 | LLM Engine     | 🤖 Llamando a IA
00:08 | LLM Engine     | ✅ Diagnóstico: Off-by-one
00:09 | Code Generator | 💡 Generando fix
00:10 | Code Generator | ✅ Fix validado
00:11 | Git Ops        | 🔀 Creando rama
00:12 | Git Ops        | 📤 Pusheando código
00:13 | PR Manager     | 🔀 Creando PR #456
00:14 | GitHub API     | ✅ PR #456 creado
00:15 | CI Pipeline    | 🧪 Ejecutando tests en PR
00:45 | CI Pipeline    | ✅ Tests pasaron
00:46 | Result Listener| ✅ PR tests OK
00:47 | Notifier       | 📧 Enviando notificación
01:00 | Developer      | 👀 Revisando PR
01:05 | Developer      | ✅ PR aprobado
01:06 | Auto Merge     | 🔀 Mergeando PR
01:07 | GitHub API     | ✅ PR merged
01:08 | Issue Manager  | 🔒 Issue #123 cerrado
01:09 | Audit Log      | 📝 Registro completado
```

## 🔐 Puntos de Validación y Seguridad

```
┌─────────────────────────────────────────────────────────┐
│          VALIDACIONES Y CHECKPOINTS                      │
└─────────────────────────────────────────────────────────┘

Issue Analysis Phase:
  ✓ ¿Payload tiene firma válida?
  ✓ ¿Issue tiene labels 'ci-failure'?
  ✓ ¿Logs contienen información útil?
  ✓ ¿Repository está permitido?
  
Code Generation Phase:
  ✓ ¿LLM confidence > threshold?
  ✓ ¿Cambios en scope permitido?
  ✓ ✓ No modificar archivos críticos?
  ✓ ¿Cambios menores? (< 500 líneas)
  ✓ ¿Código válido (sintaxis)?
  ✓ ¿No hay secrets en el código?
  
Git Operations Phase:
  ✓ ¿Token de git válido?
  ✓ ¿Rama no existe ya?
  ✓ ¿Permiso de push?
  ✓ ¿Commit message válido?
  
PR Creation Phase:
  ✓ ¿PR base branch es main/master?
  ✓ ¿Descripción completa?
  ✓ ¿Reviewer asignado?
  ✓ ¿Labels correctos?
  
Merge Phase:
  ✓ ¿CI pasó?
  ✓ ¿PR aprobado?
  ✓ ¿No hay conflictos?
  ✓ ¿Revisión humana completada?
```

---

**Diagrama actualizado**: Junio 2026
