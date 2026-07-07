# Plataforma de Automatización 

## Estado del proyecto

🚧 En diseño

---

## Antecedentes

Servidor A

Fue mi primer entorno de trabajo. Durante su construcción aprendí sobre Docker, redes, dominios, contenedores y despliegue de servicios. Sin embargo, la arquitectura fue evolucionando a medida que resolvía problemas, lo que ocasionó cambios frecuentes en la configuración, puertos y organización de los servicios.

Servidor B

Fue creado con un propósito específico: alojar servicios de inteligencia artificial. Cumplió correctamente su función, pero nuevamente muchas decisiones de infraestructura se tomaron durante la implementación y no como parte de un diseño previo.

Servidor C

A partir de la experiencia obtenida con los servidores anteriores, este proyecto nace con un enfoque diferente: diseñar primero la arquitectura y construir después.

El Servidor C será una plataforma de automatización independiente, diseñada desde el inicio para comunicarse con el Servidor B, permitiendo integrar flujos automatizados con servicios de inteligencia artificial sin afectar los entornos ya existentes.

## Problema identificado

Durante el desarrollo de los servidores anteriores identifiqué varios inconvenientes:

- La arquitectura fue evolucionando mientras aprendía nuevas tecnologías.
- Muchas decisiones se tomaron de forma reactiva y no como parte de un diseño inicial.
- La configuración de puertos, redes y servicios tuvo que modificarse varias veces.
- La documentación del proyecto terminó distribuida en numerosas conversaciones con herramientas de inteligencia artificial, dificultando reconstruir el proceso completo.
- El desarrollo se realizó resolviendo problemas puntuales, pero sin una visión arquitectónica integral desde el inicio.

Como resultado, comprendí que antes de desarrollar nuevas automatizaciones es necesario diseñar primero la infraestructura que las soportará.

...

## Objetivo general

Diseñar e implementar una plataforma de automatización modular, segura y escalable que permita desplegar, administrar y ejecutar servicios de forma organizada, facilitando la integración con un servidor especializado en inteligencia artificial mediante una arquitectura desacoplada.
...

## Objetivos específicos

- Implementar una plataforma centralizada para la administración del servidor y sus servicios.
- Desplegar un sistema de orquestación de flujos de trabajo.
- Integrar un entorno independiente para la ejecución de procesos en Python.
- Implementar una base de datos para el almacenamiento persistente de información.
- Diseñar una arquitectura basada en contenedores que facilite el despliegue, mantenimiento y crecimiento del sistema.
- Establecer un mecanismo de comunicación seguro entre el Servidor C y el Servidor B.
...

## Arquitectura propuesta

**Docker:** Proporciona el aislamiento entre servicios, permitiendo que cada aplicación se ejecute de forma independiente y facilitando el despliegue, la actualización y la portabilidad de la infraestructura.

**Cosmos:** Será la plataforma de administración del Servidor C. Su función será proporcionar una interfaz visual para administrar los contenedores y, además, centralizar aspectos de infraestructura como la publicación segura de servicios, el acceso remoto y la gestión de aplicaciones.

No se incorpora únicamente por comodidad, sino porque se convierte en la puerta de entrada para administrar toda la plataforma de forma organizada.

**n8n:** Será el motor de automatización. Recibirá solicitudes provenientes de otros sistemas, coordinará los flujos de trabajo y decidirá cuándo interactuar con el Python Worker, PostgreSQL o el Servidor B.

**Python Worker:** Será el encargado de ejecutar la lógica de negocio y el procesamiento que requiera mayor capacidad computacional. n8n actuará como orquestador, mientras que Python realizará el trabajo especializado.

**JupyterLab* será el entorno de desarrollo y experimentación para Python. Su función será permitir la creación, prueba y validación de scripts, análisis de datos y prototipos antes de integrarlos al Python Worker, donde posteriormente serán ejecutados como parte de los flujos automatizados. De esta manera, el desarrollo y las pruebas permanecen separados del entorno de ejecución en producción, facilitando el mantenimiento y reduciendo el riesgo de afectar los procesos automatizados.

**PostgreSQL** Será la capa de persistencia de la plataforma.
Almacenará de forma estructurada la información generada por las automatizaciones y permitirá conservar el estado de los procesos más allá del ciclo de vida de los contenedores.

**Redes Docker:** Permitirán que los servicios se comuniquen de forma controlada, evitando exponer componentes internos innecesariamente y mejorando la seguridad de la arquitectura.

**Comunicación con el Servidor B**

El Servidor C será el encargado de orquestar las automatizaciones.
Cuando un flujo requiera capacidades de inteligencia artificial, establecerá comunicación con el Servidor B mediante interfaces definidas para enviar solicitudes y recibir respuestas, manteniendo desacopladas las responsabilidades de ambas plataformas.

## Componentes

- Cosmos
- Docker
- n8n
- Python Worker
- PostgreSQL
- JupyterLab