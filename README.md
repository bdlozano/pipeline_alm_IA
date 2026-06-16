# Self-Healing CI System with Generative AI

Un sistema automatizado que detecta fallos de CI/CD, diagnostica causas raíz y genera correcciones usando IA generativa.

## 🎯 Visión General

Este sistema implementa un flujo de autocuración para pipelines de CI/CD:

1. **Detección**: Webhook captura fallos de CI/CD
2. **Análisis**: Agente IA analiza logs y diagnostica causa raíz
3. **Generación**: Agente genera código de corrección
4. **Validación**: Tests automáticos validan la solución
5. **Revisión**: Humano aprueba antes de mergear
6. **Resolución**: Rama se mergea a main automáticamente

## 📋 Características Principales

- ✅ **Análisis Automático de Logs**: Extrae y analiza errores de CI/CD
- ✅ **Diagnóstico con IA**: Identifica causas raíz usando LLM
- ✅ **Generación de Código**: Crea patches y correcciones
- ✅ **Git Automation**: Crea ramas, commits y PRs automáticamente
- ✅ **Validación de Tests**: Verifica que tests pasen antes de PRs
- ✅ **Revisión Humana**: Requiere aprobación antes de merge
- ✅ **Audit Trail**: Registro completo de todas las acciones
- ✅ **Escalada Inteligente**: Detecta casos sin solución clara

## 🏗️ Arquitectura

```
CI/CD Pipeline
    ↓ (falla)
Webhook Detector
    ↓
GitHub Issue Creator
    ↓
AI Agent Service
├─ Log Analyzer
├─ LLM Diagnosis Engine
├─ Code Fix Generator
├─ Git Operations
└─ PR Creator
    ↓
CI Tests on PR
    ↓
Human Review
    ↓
Merge to Main
```

## 📂 Estructura de Directorios

```
.
├── README.md                          # Este archivo
├── ARCHITECTURE.md                    # Documentación técnica detallada
├── FLOW_DIAGRAM.md                    # Diagrama ASCII del flujo
├── requirements.txt                   # Dependencias Python
├── .env.example                       # Variables de entorno
├── .github/
│   └── workflows/
│       ├── test.yml                   # Tests unitarios
│       ├── lint.yml                   # Linting y format
│       └── deploy.yml                 # Deployment
├── src/
│   ├── __init__.py
│   ├── main.py                        # Entry point del agente
│   ├── config.py                      # Configuración global
│   ├── models.py                      # Modelos de datos
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── base_agent.py             # Clase base del agente
│   │   ├── issue_listener.py         # Escucha issues
│   │   ├── log_analyzer.py           # Analiza logs
│   │   ├── llm_engine.py             # Motor LLM
│   │   ├── code_generator.py         # Generador de código
│   │   └── pr_manager.py             # Gestor de PRs
│   ├── integrations/
│   │   ├── __init__.py
│   │   ├── github_api.py             # API de GitHub
│   │   ├── git_ops.py                # Operaciones Git
│   │   ├── llm_provider.py           # Proveedores LLM
│   │   └── webhook_handler.py        # Handler de webhooks
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── logger.py                 # Logging
│   │   ├── validators.py             # Validadores
│   │   └── helpers.py                # Utilidades
│   └── rules/
│       ├── __init__.py
│       └── safety_rules.py           # Reglas de seguridad
├── tests/
│   ├── __init__.py
│   ├── test_agent.py                 # Tests del agente
│   ├── test_log_analyzer.py          # Tests del analizador
│   ├── test_code_generator.py        # Tests del generador
│   └── fixtures/
│       └── sample_logs.py            # Logs de ejemplo
├── examples/
│   ├── webhook_payload.json          # Payload de ejemplo
│   ├── issue_body_example.md         # Ejemplo de issue
│   └── pr_description_example.md     # Ejemplo de PR
├── docs/
│   ├── WEBHOOK_SETUP.md              # Configurar webhooks
│   ├── LLM_SETUP.md                  # Configurar LLM
│   ├── SECURITY.md                   # Consideraciones de seguridad
│   └── TROUBLESHOOTING.md            # Guía de troubleshooting
└── docker/
    ├── Dockerfile                     # Imagen Docker
    └── docker-compose.yml             # Compose para desarrollo
```

## 🚀 Inicio Rápido

### Requisitos

- Python 3.9+
- Git
- GitHub Token (con permisos read/write)
- API Key de LLM (Claude, GPT-4, etc.)
- Docker (opcional)

### Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/bdlozano/pipeline_alm_IA.git
cd pipeline_alm_IA

# 2. Crear entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Configurar variables de entorno
cp .env.example .env
# Editar .env con tus credenciales

# 5. Ejecutar agente
python src/main.py
```

### Con Docker

```bash
docker-compose up -d
```

## ⚙️ Configuración

### Variables de Entorno

```bash
# GitHub
GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxx
GITHUB_OWNER=bdlozano
GITHUB_REPO=pipeline_alm_IA

# LLM Provider
LLM_PROVIDER=anthropic  # o openai, cohere
LLM_API_KEY=sk-xxxxxxxxxxxxxxxx
LLM_MODEL=claude-3-sonnet-20240229

# Agente
AGENT_CONFIDENCE_THRESHOLD=0.80
AGENT_AUTO_MERGE_THRESHOLD=0.90
AGENT_MAX_RETRIES=3
AGENT_TIMEOUT_MINUTES=60

# Logging
LOG_LEVEL=INFO
LOG_FILE=logs/agent.log

# Webhook
WEBHOOK_PORT=8000
WEBHOOK_SECRET=your-secret-key
```

Ver `.env.example` para documentación completa.

## 📖 Documentación Detallada

- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Arquitectura técnica completa
- **[FLOW_DIAGRAM.md](docs/FLOW_DIAGRAM.md)** - Diagrama de flujo ASCII
- **[WEBHOOK_SETUP.md](docs/WEBHOOK_SETUP.md)** - Configurar webhooks en CI/CD
- **[LLM_SETUP.md](docs/LLM_SETUP.md)** - Configurar providers LLM
- **[SECURITY.md](docs/SECURITY.md)** - Guía de seguridad
- **[TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)** - Guía de troubleshooting

## 🔄 Flujo de Ejemplo

### 1. CI Falla

```bash
# Pipeline de tests falla
pytest tests/ -v
# FAILED tests/test_calculator.py::test_add
```

### 2. Webhook Captura el Fallo

```json
{
  "pipeline_run_id": "build-12345",
  "status": "FAILED",
  "failed_job": "test-unit",
  "branch": "main",
  "logs": "AssertionError: expected 5 to equal 10"
}
```

### 3. Issue Creado

```
🔴 CI FAILURE: unit tests failed on main
Status: FAILED
Build: build-12345
Logs: [error details]
```

### 4. Agente Analiza

```
Analyzing logs...
Root cause: Off-by-one error in Calculator.add()
Confidence: 95%
```

### 5. Agente Genera Fix

```python
# Cambio: src/calculator.py
- return a + b + 1  # BUG
+ return a + b      # FIXED
```

### 6. PR Creado

```
🤖 AI Auto-Fix: Resolve unit test failure in Calculator.add()
Issue: #1234567890
Confidence: 95%
Status: Tests passing ✅
```

### 7. Humano Revisa y Aprueba

```
✅ Approved by @reviewer
```

### 8. Merge a Main

```
Merged #4567 into main
Issue #1234567890 closed
```

## 🛡️ Puntos de Intervención Humana

### Siempre Requerido

- [ ] Revisión de cambios antes de merge a main
- [ ] Validación de que la solución es correcta
- [ ] Verificación de que no hay side effects

### Condicionales

- [ ] Si confidence < 80%: Pre-aprobación requerida
- [ ] Si múltiples intentos fallan: Escalar a especialista
- [ ] Si cambios fuera de scope: Revisión manual

### Automáticos (Sin Intervención)

- ✅ Confidence > 90% AND tests pass: Auto-merge (configurable)
- ✅ Trivial fixes (typo, lint): Auto-merge

## 📊 Monitoreo y Observabilidad

### Métricas Disponibles

- Cantidad de issues auto-resueltos
- Tasa de éxito de auto-fixes
- Tiempo promedio de resolución
- Confianza promedio del agente
- Logs de todas las operaciones

### Dashboards

Ver `/docs/monitoring.md` para configurar dashboards.

## 🧪 Tests

```bash
# Ejecutar todos los tests
pytest tests/ -v

# Con coverage
pytest tests/ --cov=src --cov-report=html

# Tests específicos
pytest tests/test_agent.py -v
```

## 🐛 Troubleshooting

### El agente no se conecta a GitHub

```bash
# Verificar token
echo $GITHUB_TOKEN

# Verificar permisos
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user
```

### LLM no responde

```bash
# Verificar API key
echo $LLM_API_KEY

# Verificar conexión
python -c "import anthropic; print(anthropic.__version__)"
```

Ver `docs/TROUBLESHOOTING.md` para más casos.

## 🤝 Contribuir

1. Fork el repositorio
2. Crea una rama (`git checkout -b feature/my-feature`)
3. Commit cambios (`git commit -m 'Add feature'`)
4. Push a la rama (`git push origin feature/my-feature`)
5. Abre un Pull Request

## ⚖️ Licencia

MIT License - Ver LICENSE para detalles.

## 👤 Autor

**Diseño y Arquitectura**: Sistema Self-Healing CI con IA Generativa

---

## 📞 Soporte

- 📖 [Documentación Completa](./docs/)
- 🐛 [Reportar Issues](https://github.com/bdlozano/pipeline_alm_IA/issues)
- 💬 [Discussions](https://github.com/bdlozano/pipeline_alm_IA/discussions)

---

**Última actualización**: Junio 2026
