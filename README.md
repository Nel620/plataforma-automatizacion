# Plataforma de Automatización

## Estado del proyecto

✅ Implementado (v1) — en operación

---

## Antecedentes

### Servidor A

Fue mi primer entorno de trabajo. Durante su construcción aprendí sobre Docker, redes, dominios, contenedores y despliegue de servicios. Sin embargo, la arquitectura fue evolucionando a medida que resolvía problemas, lo que ocasionó cambios frecuentes en la configuración, puertos y organización de los servicios.

### Servidor B

Fue creado con un propósito específico: alojar servicios de inteligencia artificial (Ollama). Cumplió correctamente su función, pero nuevamente muchas decisiones de infraestructura se tomaron durante la implementación y no como parte de un diseño previo.

Durante la operación, el servidor original quedó inestable al cargar modelos de lenguaje sin un control adecuado de recursos, lo que obligó a reconstruirlo desde cero. En la reconstrucción se aplicaron dos correcciones clave: 

- Se ubicó en la **misma región de DigitalOcean que el Servidor C (SFO2)**, para que ambos pudieran compartir la red privada (VPC) del proveedor y comunicarse sin exponer servicios a internet público.
- Se adoptó un criterio más conservador para la selección de modelos de lenguaje, priorizando estabilidad sobre capacidad, dado el límite de recursos del servidor (8 GB RAM).

### Servidor C

A partir de la experiencia obtenida con los servidores anteriores, este proyecto nace con un enfoque diferente: diseñar primero la arquitectura y construir después.

El Servidor C es una plataforma de automatización independiente, diseñada desde el inicio para comunicarse con el Servidor B, permitiendo integrar flujos automatizados con servicios de inteligencia artificial sin afectar los entornos ya existentes.

---

## Problema identificado

Durante el desarrollo de los servidores anteriores identifiqué varios inconvenientes:

- La arquitectura fue evolucionando mientras aprendía nuevas tecnologías.
- Muchas decisiones se tomaron de forma reactiva y no como parte de un diseño inicial.
- La configuración de puertos, redes y servicios tuvo que modificarse varias veces.
- La documentación del proyecto terminó distribuida en numerosas conversaciones con herramientas de inteligencia artificial, dificultando reconstruir el proceso completo.
- El desarrollo se realizó resolviendo problemas puntuales, pero sin una visión arquitectónica integral desde el inicio.

Como resultado, comprendí que antes de desarrollar nuevas automatizaciones es necesario diseñar primero la infraestructura que las soportará.

---

## Objetivo general

Diseñar e implementar una plataforma de automatización modular, segura y escalable que permita desplegar, administrar y ejecutar servicios de forma organizada, facilitando la integración con un servidor especializado en inteligencia artificial mediante una arquitectura desacoplada.

## Objetivos específicos

- Implementar una plataforma centralizada para la administración del servidor y sus servicios.
- Desplegar un sistema de orquestación de flujos de trabajo.
- Integrar un entorno independiente para la ejecución de procesos en Python.
- Implementar una base de datos para el almacenamiento persistente de información.
- Diseñar una arquitectura basada en contenedores que facilite el despliegue, mantenimiento y crecimiento del sistema.
- Establecer un mecanismo de comunicación seguro entre el Servidor C y el Servidor B.

---

## Decisión de plataforma de administración: de Cosmos a Coolify

La elección inicial fue **Cosmos**, por ser una solución "todo en uno" con una interfaz visual atractiva y un enfoque más cercano a un entorno *self-hosted* de uso doméstico/personal — es decir, pensado para quien busca simplicidad y control total sobre su propio servidor, similar a plataformas como Unraid o CasaOS en su filosofía.

Durante la implementación surgió un problema recurrente con la emisión de certificados SSL vía Let's Encrypt (error de *rate limit* por múltiples registros de cuenta ACME), asociado a inestabilidad en la forma en que Cosmos gestionaba la persistencia de su configuración.

Se evaluó migrar a **Coolify**, una plataforma orientada a un perfil más cercano al de PaaS (*Platform as a Service*, al estilo Heroku self-hosted), con mayor madurez en:

- Gestión automática y estable de certificados SSL.
- Despliegue de servicios mediante plantillas (n8n, PostgreSQL, pgAdmin) y Docker Compose personalizado.
- Panel de control más consistente para administrar dominios, variables de entorno y redes internas de Docker.

El cambio implicó una curva de aprendizaje adicional (nomenclatura distinta, gestión de redes explícita mediante la opción "Connect to Predefined Network", necesidad de identificar nombres de contenedores generados automáticamente), pero resultó en una plataforma más estable para el objetivo del proyecto.

---

## Arquitectura final implementada

```
DigitalOcean
    ├── Servidor C (SFO2) — Plataforma de automatización
    │     ├── Coolify           → administración de contenedores, dominios y SSL
    │     ├── n8n                → orquestador de flujos de trabajo
    │     ├── PostgreSQL         → persistencia de datos
    │     ├── pgAdmin            → administración visual de PostgreSQL
    │     ├── Python Worker      → procesamiento especializado (API + tareas programadas)
    │     └── JupyterLab         → entorno de desarrollo y experimentación en Python
    │
    └── Servidor B (SFO2) — Servicios de inteligencia artificial
          └── Ollama             → modelos de lenguaje local (LLM)

    Comunicación Servidor C ↔ Servidor B: red privada (VPC), misma región,
    sin exposición de puertos a internet público.
```

### Componentes

| Componente | Función | Ubicación |
|---|---|---|
| **Coolify** | Plataforma de administración del Servidor C: contenedores, dominios, SSL | Servidor C |
| **n8n** | Motor de automatización. Recibe solicitudes, coordina flujos y decide cuándo interactuar con el Python Worker, PostgreSQL o el Servidor B | Servidor C |
| **Python Worker** | Ejecuta lógica de negocio y procesamiento especializado. Expone una API (FastAPI) para recibir tareas desde n8n y ejecuta tareas programadas (cron) | Servidor C |
| **JupyterLab** | Entorno de desarrollo y experimentación en Python, separado del entorno de producción | Servidor C |
| **PostgreSQL** | Capa de persistencia de la plataforma | Servidor C |
| **pgAdmin** | Interfaz visual para administrar y consultar PostgreSQL | Servidor C |
| **Ollama** | Servidor de modelos de lenguaje (LLM) local, consumido por n8n para tareas de IA | Servidor B |

### Comunicación con el Servidor B

El Servidor C orquesta las automatizaciones. Cuando un flujo requiere capacidades de inteligencia artificial, n8n establece comunicación con Ollama en el Servidor B mediante peticiones HTTP a través de la **red privada de DigitalOcean (VPC)**, ya que ambos servidores residen en la misma región (SFO2). El acceso está restringido mediante firewall (`ufw`) únicamente a la IP privada del Servidor C, sin exponer el servicio a internet público.

### Redes Docker

Los servicios dentro del Servidor C se comunican mediante la red interna gestionada por Coolify, evitando exponer componentes internos innecesariamente (por ejemplo, PostgreSQL no está expuesto públicamente; solo es alcanzable por los demás contenedores conectados a la misma red Docker).

---

## Dominios utilizados

| Servicio | Dominio |
|---|---|
| n8n | `n8n.entornoandres.me` |
| JupyterLab | `jupyter.entornoandres.me` |
| Python Worker | `pythonworker.entornoandres.me` |
| pgAdmin | `pgadmin.entornoandres.me` |

---

## Lecciones aprendidas

- Definir la región de los servidores desde el inicio evita limitaciones posteriores de red privada entre servicios.
- El dimensionamiento de recursos (RAM/CPU) debe hacerse en función de la carga real esperada, especialmente al trabajar con modelos de lenguaje, cuyo consumo de memoria puede saturar servidores pequeños.
- Mantener actualizados los registros DNS y eliminar los que ya no corresponden a servidores activos evita conflictos de resolución difíciles de diagnosticar.
- Documentar las decisiones de arquitectura a medida que se toman —y no al final— facilita mantener coherencia entre lo planeado y lo implementado.

---

## Mejoras futuras (opcionales, no bloqueantes)

La plataforma está completa y en operación. Estas son mejoras que se pueden evaluar más adelante, sin que su ausencia afecte el funcionamiento actual:

- Backups automáticos programados para PostgreSQL.
- Autenticación adicional para los servicios expuestos públicamente.
- Modelos de lenguaje adicionales en el Servidor B según necesidades futuras.

---

## Archivos de despliegue (Docker Compose)

Archivos usados para desplegar cada servicio en Coolify. Sirven de respaldo para redesplegar la plataforma en un servidor nuevo sin reconstruir la configuración desde cero.

> Los valores de contraseñas, tokens y hosts internos son placeholders — deben reemplazarse por los propios al reutilizar estos archivos.

### JupyterLab

```yaml
services:
  jupyterlab:
    image: jupyter/scipy-notebook:latest
    ports:
      - "8888"
    environment:
      - JUPYTER_TOKEN=CAMBIA_ESTO_POR_UNA_CLAVE_SEGURA
    volumes:
      - jupyter-data:/home/jovyan/work

volumes:
  jupyter-data:
```

### Python Worker

```yaml
services:
  python-worker:
    image: python:3.11-slim
    working_dir: /app
    volumes:
      - /opt/python-worker:/app
    command: sh -c "pip install --no-cache-dir -r requirements.txt && uvicorn main:app --host 0.0.0.0 --port 8000"
    ports:
      - "8000"
    environment:
      - DB_HOST=NOMBRE_CONTENEDOR_POSTGRES
      - DB_PORT=5432
      - DB_NAME=postgres
      - DB_USER=postgres
      - DB_PASSWORD=CONTRASEÑA_POSTGRES
```

Código de la aplicación (montado desde `/opt/python-worker` en el Servidor C):

**requirements.txt**
```
fastapi
uvicorn
apscheduler
psycopg2-binary
```

**main.py** — expone `GET /health`, `POST /run-task` (consumido por n8n) y una tarea programada con APScheduler.

### pgAdmin

```yaml
services:
  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      - PGADMIN_DEFAULT_EMAIL=tu@email.com
      - PGADMIN_DEFAULT_PASSWORD=CAMBIA_ESTO_POR_UNA_CLAVE_SEGURA
    ports:
      - "80"
    volumes:
      - pgadmin-data:/var/lib/pgadmin

volumes:
  pgadmin-data:
```

### n8n y PostgreSQL

Se desplegaron desde las plantillas nativas de Coolify (catálogo "One-Click"), no desde un compose escrito a mano. Para exportar su configuración exacta: en el recurso correspondiente, botón **Edit Compose File** (n8n) o pestaña **Details** (PostgreSQL) dentro de Coolify.

### Red interna

Cada recurso necesita la opción **"Connect To Predefined Network"** activada en su configuración general de Coolify, para conectarse a la red `coolify` donde vive PostgreSQL y poder resolverse por nombre de contenedor.

---

## Conexión con el Servidor B (Ollama) — configuración de referencia

```bash
sudo systemctl edit ollama
```
```ini
[Service]
Environment="OLLAMA_HOST=IP_PRIVADA_SERVIDOR_B:11434"
```
```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama

sudo ufw allow from IP_PRIVADA_SERVIDOR_C to any port 11434
sudo ufw enable
```

Endpoint consumido desde n8n:
```
POST http://IP_PRIVADA_SERVIDOR_B:11434/api/generate
```
```json
{
  "model": "llama3",
  "prompt": "...",
  "stream": false
}
```
