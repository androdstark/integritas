# CLAUDE.md — integritas

Archivo de contexto para Claude Code. Lee este archivo antes de escribir cualquier línea de código.

---

## Qué es este proyecto

**Integritas** (nombre técnico) / **Mírantir** (nombre de producto) es una plataforma pública de transparencia que genera un **Índice de Exposición al Riesgo (IER)** por funcionario público peruano.

No acusa ni condena. Cruza datos abiertos del Estado y expone señales de riesgo de forma auditable. El output es siempre un score 0.0–1.0 con desglose de variables.

Referencia directa: [Cerebro Digital](https://github.com/brunoclz) de Bruno César (Brasil, 2026) — cruzó 70+ bases de datos usando el CPF de políticos. Perú tiene su equivalente: DNI (personas) + RUC (empresas).

---

## Estado actual

- Concepto definido, arquitectura diseñada
- Sin código inicial — este repo es el punto de partida
- Stack elegido, fuentes de datos mapeadas
- Pendiente: implementación del ETL, motor de scoring, API y frontend

---

## Stack técnico

| Capa | Tecnología |
|---|---|
| Backend API | Python 3.12 + FastAPI |
| Base de datos | PostgreSQL 16 |
| ETL / scraping | Python + requests + BeautifulSoup + Playwright |
| Motor de scoring | scikit-fuzzy (lógica difusa) |
| Grafos de relaciones | NetworkX + Pyvis |
| IA narrativa | Anthropic Claude API (claude-sonnet-4-6) |
| Frontend | React + Tailwind CSS |
| Hosting | Render o Hetzner VPS |
| Control de versiones | Git + GitHub |

---

## Fuentes de datos públicas

| Fuente | Contenido | URL |
|---|---|---|
| Portal de Transparencia Estándar | Directorio funcionarios, planillas, declaraciones juradas | transparencia.gob.pe |
| Portal de Transparencia Económica (MEF) | Proveedores del Estado, contratos 1999–hoy, transferencias | mef.gob.pe |
| OSCE / SEACE | Licitaciones y contrataciones públicas | seace.osce.gob.pe |
| SUNAT RUC | Empresas, representantes legales, vinculaciones | sunat.gob.pe |
| JNE | Declaraciones de bienes y rentas de candidatos y electos | jne.gob.pe |
| Contraloría General | Informes de auditoría, observaciones, sanciones | contraloria.gob.pe |
| INFObras (MEF) | Avance y costo de obras públicas | infobras.vivienda.gob.pe |
| RENIEC | Verificación de identidad | reniec.gob.pe |
| Poder Judicial / INDECOPI | Procesos legales públicos | pj.gob.pe |

**Identificadores clave:**
- `DNI` — personas naturales (equivalente al CPF brasileño)
- `RUC` — personas jurídicas (equivalente al CNPJ brasileño)

---

## Motor de scoring — Índice de Exposición al Riesgo (IER)

El IER es un valor continuo [0.0 – 1.0] calculado por lógica difusa. **Nunca se usa la palabra "corrupto".**

### Variables de entrada

| Variable | Peso | Señal de riesgo |
|---|---|---|
| Contratos con empresas de familiares | Alto | Empresa del cónyuge/hijo en contratos de su entidad |
| Discrepancia patrimonial | Alto | Declaración de bienes no justifica ingresos conocidos |
| Historial Contraloría | Alto | Auditorías previas con hallazgos graves |
| Vínculos empresariales directos | Medio | Funcionario como representante legal de empresa contratista |
| Concentración de contratos | Medio | Una empresa gana repetidamente licitaciones de la misma entidad |
| Precios vs. mercado | Medio | Precio adjudicado muy por encima del valor de mercado |
| Funcionarios fantasma | Bajo-Medio | Persona en planilla sin presencia verificable |

### Proceso de cálculo

1. Entrada: DNI del funcionario
2. ETL: consulta en paralelo las fuentes listadas arriba
3. Normalización: cada variable → valor difuso [0.0–1.0]
4. Agregación: ponderación por peso + defuzzificación
5. Salida: IER + desglose de contribuciones por variable

---

## Arquitectura del proyecto

```
integritas/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI entrypoint
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── funcionarios.py   # GET /funcionarios/{dni}
│   │   │       └── scoring.py        # GET /scoring/{dni}
│   │   ├── etl/
│   │   │   ├── mef.py           # Portal Transparencia MEF
│   │   │   ├── osce.py          # SEACE / OSCE
│   │   │   ├── sunat.py         # SUNAT RUC
│   │   │   ├── jne.py           # JNE declaraciones
│   │   │   ├── contraloria.py   # Contraloría General
│   │   │   └── infobras.py      # INFObras
│   │   ├── scoring/
│   │   │   ├── engine.py        # Motor de lógica difusa
│   │   │   ├── variables.py     # Definición de variables y pesos
│   │   │   └── fuzzy_rules.py   # Reglas difusas
│   │   ├── models/
│   │   │   ├── funcionario.py   # Modelo DB funcionario
│   │   │   └── score.py         # Modelo DB resultado IER
│   │   ├── schemas/
│   │   │   ├── funcionario.py   # Pydantic schemas
│   │   │   └── scoring.py       # Pydantic schemas output
│   │   └── db/
│   │       ├── session.py       # SQLAlchemy session
│   │       └── models.py        # ORM models
│   ├── migrations/              # Alembic
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── App.tsx
│   ├── package.json
│   └── Dockerfile
├── infra/
│   ├── docker-compose.yml
│   └── nginx.conf
├── docs/
│   ├── api-sources.md           # Estado real de cada API (acceso vs scraping)
│   └── legal.md                 # Marco legal aplicable
├── .gitignore                   # Nunca subir .env ni __pycache__
├── CLAUDE.md                    # Este archivo
└── README.md
```

---

## Fases de desarrollo

### Fase 0 — Setup (ahora)
- [x] Repo creado ✅
- [x] CLAUDE.md ✅
- [ ] Crear `.gitignore` ← **hacer primero antes de cualquier código**
- [ ] Docker-compose con PostgreSQL local
- [ ] Mapear qué fuentes tienen API real vs requieren scraping
- [ ] Definir esquema de base de datos inicial

### Fase 1 — ETL básico (MVP)
- [ ] Conector MEF: consulta por DNI → contratos públicos
- [ ] Conector OSCE/SEACE: licitaciones relacionadas
- [ ] Conector JNE: declaraciones de bienes
- [ ] Almacenamiento en PostgreSQL
- [ ] CLI de prueba: `python -m integritas.cli query --dni 12345678`

### Fase 2 — Motor de scoring
- [ ] Implementar variables de entrada (tabla de arriba)
- [ ] Motor scikit-fuzzy con reglas definidas
- [ ] API FastAPI: `GET /scoring/{dni}` → retorna IER + desglose
- [ ] Tests unitarios del motor

### Fase 3 — Frontend
- [ ] Búsqueda por DNI o nombre
- [ ] Perfil de funcionario con IER y gráfico de contribuciones
- [ ] Grafo de relaciones (NetworkX → D3.js/Pyvis)
- [ ] Explicación pública de la metodología

### Fase 4 — Producción
- [ ] Deploy en Hetzner VPS (mismo servidor que Zhinova)
- [ ] Dominio y SSL
- [ ] Actualización batch de datos (semanal)
- [ ] Consulta legal antes de lanzamiento público

---

## Decisiones pendientes (resolver antes de codear cada módulo)

1. **APIs vs scraping**: ¿Qué fuentes tienen acceso programático real? Verificar antes de implementar cada conector.
2. **Scoring inicial**: Empezar solo con reglas (sin ML). ML supervisado cuando haya datos etiquetados suficientes.
3. **Naturaleza del proyecto**: Fase 1 = civic tech abierto/open source. Fase 2+ evaluar modelo B2G/B2B.
4. **Frecuencia de actualización**: Batch semanal por defecto. On-demand para consultas individuales (con rate limit).
5. **Unidad de análisis**: Persona natural (por DNI) como unidad primaria. Cargo como contexto secundario.

---

## Consideraciones legales (críticas)

- Usar **exclusivamente datos de fuentes abiertas** del Estado Peruano
- Output siempre como **score de riesgo**, nunca como acusación o sentencia
- Mostrar siempre el **desglose de variables** que contribuyeron al IER (auditable)
- Marco legal aplicable:
  - Ley N°27806 — Transparencia y Acceso a Información Pública
  - Ley N°29733 — Protección de Datos Personales
- Consultar con abogado especializado antes de lanzamiento público
- No usar el término "corrupto" en ninguna parte del producto

---

## Reglas de código

1. Think before acting. Read existing files before writing code.
2. Be concise in output but thorough in reasoning.
3. Prefer editing over rewriting whole files.
4. Do not re-read files you already read unless the file may have changed.
5. Test your code before declaring done.
6. No sycophantic openers or closing fluff.
7. Keep solutions simple and direct.
8. My direct instructions always override these rules.
9. Responde en español salvo que yo escriba en inglés.
10. Si hay ambigüedad, da la respuesta más probable y señala la duda al final.

---

## Entorno de desarrollo local

- **PC principal**: Ubuntu 25, carpeta `~/projects/integritas/`
- **Servidor**: Hetzner VPS (mismo que Zhinova)
- **Flujo de trabajo**:
  1. Claude Code desarrolla en `~/projects/integritas/` (PC local)
  2. `git push origin main` desde el PC
  3. `git pull origin main` en el servidor Hetzner para deploy

### .gitignore obligatorio — crear antes de cualquier código

```bash
cat > .gitignore << 'EOF'
# Python
.env
*.env
.env.*
__pycache__/
*.pyc
*.pyo
.venv/
venv/
*.egg-info/
.pytest_cache/

# Node / Frontend
node_modules/
dist/
build/
.next/

# Logs y temporales
*.log
.DS_Store
*.sqlite3

# IDE
.vscode/
.idea/
EOF
```

### Variables de entorno (nunca en git)

Crear `backend/.env` local — **este archivo nunca se sube**:

```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/integritas
ANTHROPIC_API_KEY=sk-ant-...
DEBUG=true
```

---

## Comandos útiles

```bash
# Setup local
docker-compose up -d           # Levanta PostgreSQL + Redis
cd backend && pip install -r requirements.txt
uvicorn app.main:app --reload   # API en http://localhost:8000

# Migraciones
alembic upgrade head

# Tests
pytest tests/ -v

# Query de prueba (cuando esté implementado)
curl http://localhost:8000/api/v1/scoring/12345678
```
