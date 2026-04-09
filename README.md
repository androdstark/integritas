# Integritas — Scoring de Riesgo de Funcionarios Peruanos

> **Índice de Exposición al Riesgo (IER)** — plataforma pública que cruza bases de datos del Estado peruano para generar un score de riesgo por funcionario público.

[![Estado](https://img.shields.io/badge/estado-en%20desarrollo-yellow)](#)
[![Stack](https://img.shields.io/badge/stack-Python%20%2B%20FastAPI%20%2B%20React-blue)](#)
[![Licencia](https://img.shields.io/badge/licencia-MIT-green)](#)

---

## ¿Qué es?

Integritas (producto: **Mírantir**) es un sistema cívico de transparencia que:

1. Recibe el DNI de un funcionario público peruano
2. Cruza automáticamente múltiples bases de datos del Estado
3. Calcula un **IER [0.0–1.0]** usando lógica difusa
4. Muestra el desglose auditado de cada variable que contribuyó al score

> ⚠️ El IER es un **indicador de riesgo**, no una acusación. Todas las fuentes son públicas y verificables.

---

## Fuentes de datos

| Fuente | Contenido |
|---|---|
| MEF — Transparencia Económica | Contratos, transferencias, planillas |
| OSCE / SEACE | Licitaciones públicas |
| SUNAT RUC | Vinculaciones empresariales |
| JNE | Declaraciones de bienes y rentas |
| Contraloría General | Auditorías, observaciones, sanciones |
| INFObras | Ejecución de obras públicas |
| Poder Judicial / INDECOPI | Procesos legales públicos |

---

## Stack

- **Backend:** Python 3.12 + FastAPI
- **DB:** PostgreSQL 16
- **ETL:** requests + BeautifulSoup + Playwright
- **Scoring:** scikit-fuzzy
- **Grafos:** NetworkX + D3.js
- **Frontend:** React + Tailwind CSS
- **IA narrativa:** Claude API (Anthropic)

---

## Setup rápido

```bash
git clone https://github.com/androdstark/integritas
cd integritas
docker-compose up -d
cd backend && pip install -r requirements.txt
uvicorn app.main:app --reload
```

---

## Marco legal

Proyecto basado únicamente en datos de acceso público conforme a:
- Ley N°27806 — Transparencia y Acceso a Información Pública
- Ley N°29733 — Protección de Datos Personales

---

## Estado

Fase 0 — bootstrap del proyecto. Ver [CLAUDE.md](./CLAUDE.md) para roadmap completo.

---

## Inspiración

Basado en el trabajo de Bruno César ([Cerebro Digital](https://github.com/brunoclz), Brasil 2026), adaptado al ecosistema de datos públicos del Perú.
