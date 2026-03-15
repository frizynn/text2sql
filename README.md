# Nivii

Consultas en lenguaje natural sobre datos de ventas de un restaurante (24,212 filas). Dos modelos locales corren en CPU, offline, en Docker: uno traduce la pregunta a SQL, el otro explica los resultados en español.

## Requisitos previos

- **Python ≥ 3.9**
- **Git** y **curl** (para clonar y descargar modelos)
- Para Docker: **Docker** y **Docker Compose**
- Para uso nativo (sin Docker): **Homebrew** (macOS) o **cmake + git** (Linux) — necesarios para instalar `llama-server`

## Instalacion

```bash
git clone https://github.com/frizynn/text2sql.git
cd text2sql
```

### Opcion 1: Docker (recomendado)

Descarga los modelos (~1.6 GB), copia el `.env` y levanta todo:

```bash
mkdir -p models
curl -L -o models/qwen2.5-coder-1.5b-instruct-q4_k_m.gguf \
  https://huggingface.co/Qwen/Qwen2.5-Coder-1.5B-Instruct-GGUF/resolve/main/qwen2.5-coder-1.5b-instruct-q4_k_m.gguf
curl -L -o models/Qwen3.5-2B-Q4_K_M.gguf \
  https://huggingface.co/unsloth/Qwen3.5-2B-GGUF/resolve/main/Qwen3.5-2B-Q4_K_M.gguf

cp .env.example .env
docker-compose up --build   # o: docker compose up --build
# Abrir http://localhost:8000
```

### Opcion 2: CLI nativa (sin Docker)

Instala el paquete y sus dependencias, luego ejecuta la CLI:

```bash
pip install .                    # instala el comando 'nivii' + rich
pip install -r api/requirements.txt  # instala fastapi, uvicorn, httpx
nivii
```

El wizard de setup chequea el entorno, instala `llama-server` si falta (via Homebrew en macOS o compilando desde source), descarga los modelos (~1.6 GB), levanta los servidores y te deja elegir entre Web UI o TUI.

Para una consulta directa (requiere que los servidores esten corriendo):

```bash
nivii ask "¿Cuales son los 5 productos mas vendidos?"
```

> **Tip**: Si preferis un entorno aislado, usa un virtualenv antes de instalar:
> ```bash
> python -m venv .venv
> source .venv/bin/activate  # en Windows: .venv\Scripts\activate
> pip install .
> pip install -r api/requirements.txt
> ```

## Arquitectura

| Servicio | Modelo | Rol |
|----------|--------|-----|
| `text2sql` | qwen2.5-coder-1.5b | Pregunta → SQL |
| `nlg` | Qwen3.5-2B | Resultados → respuesta en español |
| `api` | FastAPI + SQLite | Orquesta pipeline, sirve la UI |

Pipeline MCTS: genera SQL, verifica semanticamente, y si falla refina con busqueda de arbol (critique → refine → evaluate, hasta 5 rollouts).

## Estructura

```
api/             Backend FastAPI + pipeline MCTS
cli/             CLI con setup wizard
static/          Web UI
prompts/         Prompts y few-shot examples
grammars/        Gramatica GBNF para SQL
docker/          Dockerfiles
data.csv         Dataset de ventas (24,212 filas)
```
