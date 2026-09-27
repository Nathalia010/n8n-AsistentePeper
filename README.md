# Peper Assistant

Asistente personal con inteligencia artificial conectado a **WhatsApp** y construido con **n8n**. Peper interpreta instrucciones en lenguaje natural y automatiza tareas personales como registro de gastos, gestión de eventos y recordatorios programados.

> **Demo en YouTube:** [PEGAR ENLACE AQUÍ]

## Funcionalidades

- 💰 **Finanzas:** registra gastos en COP, los guarda en n8n Data Tables y puede replicarlos en Google Sheets.
- 📅 **Eventos:** crea, consulta y elimina eventos en Google Calendar.
- 🔔 **Recordatorios:** guarda tareas pendientes, espera hasta la fecha/hora indicada, envía el aviso por WhatsApp y actualiza el estado a completado.
- 💬 **Interfaz conversacional:** el usuario interactúa desde WhatsApp usando lenguaje natural.
- 🧠 **IA:** Gemini interpreta intención, fechas relativas y selecciona la herramienta adecuada.

## Arquitectura

```text
WhatsApp Business Cloud
        |
        v
WhatsApp Trigger (n8n)
        |
        v
Peper - AI Agent
  |        |         |
  v        v         v
Gastos   Eventos   Recordatorios
  |        |         |
  v        v         v
Data     Google    Data Table
Table    Calendar     |
  |                  v
  v             programar_recordatorio
Google Sheets          |
                      v
                     Wait
                      |
                      v
                WhatsApp Send
                      |
                      v
                estado=completado
```

## Stack tecnológico

- n8n Cloud
- n8n Agent Builder
- Google Gemini API
- WhatsApp Business Cloud
- Meta for Developers
- n8n Data Tables
- Google Sheets
- Google Calendar

## Cómo funciona

### 1. Gastos

Ejemplo:

```text
Hoy gasté 25 mil en el almuerzo
```

Peper interpreta el valor, fecha, categoría y descripción. Luego registra la información y confirma al usuario.

### 2. Eventos

Ejemplo:

```text
Agéndame una cita mañana a las 3 pm
```

Peper normaliza la fecha y hora en `America/Bogota` y crea el evento en Google Calendar.

### 3. Recordatorios

Ejemplo:

```text
Recuérdame tomar agua en 5 minutos
```

Flujo:

```text
registrar_recordatorio
        |
        v
obtener ID real
        |
        v
programar_recordatorio
        |
        v
Wait hasta fecha_hora
        |
        v
Send message por WhatsApp
        |
        v
Update row(s): estado = completado
```

## Workflows principales

### Workflow principal

```text
WhatsApp Trigger -> Peper -> Send message
```

Responsabilidades:

- Recibir mensajes desde WhatsApp.
- Entregar el texto al agente Peper.
- Ejecutar skills y herramientas.
- Responder al usuario por WhatsApp.

### Peper - Esperar recordatorio

```text
When Executed by Another Workflow
        |
        v
Wait
        |
        v
Send message
        |
        v
Update row(s)
```

Inputs:

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | Number | ID real del recordatorio |
| `texto` | String | Texto del aviso |
| `fecha_hora` | String | Fecha/hora ISO 8601 |

El nodo **Wait** usa:

```text
Resume: At Specified Time
Date and Time: {{ $json.fecha_hora }}
```

## Skills del agente

Peper está dividido en tres skills principales:

- `Gestión de gastos`
- `Gestión de eventos`
- `Gestión de recordatorios`

Esto permite mantener instrucciones separadas y utilizar solo las herramientas necesarias para cada intención.

## Persistencia de datos

### Recordatorios

Campos principales:

- `id`
- `texto`
- `fecha_hora`
- `estado`
- `Evento`
- `createdAt`
- `updatedAt`

Estados utilizados:

```text
pendiente
completado
```

### Finanzas

Google Sheet `Peper_Finanzas`, hoja `Gastos`:

- Fecha
- Valor
- Categoría
- Descripción

## Zona horaria

El agente utiliza:

```text
America/Bogota
```

Las fechas de recordatorios y eventos se normalizan a ISO 8601, por ejemplo:

```text
2026-09-26T01:17:14-05:00
```

## Problemas resueltos durante el desarrollo

- Conflictos entre webhook de prueba y webhook de producción de WhatsApp.
- Datos antiguos reutilizados por `Pinned Data`.
- Paso de parámetros entre el agente y un sub-workflow.
- Error `dateTime parameter is not a valid date` en el nodo Wait.
- Números no autorizados en el entorno de prueba de WhatsApp.
- Recordatorios duplicados.
- Consumo excesivo de ejecuciones con `Schedule Trigger`.
- Límites de frecuencia y alta demanda de Gemini API.

La versión final reemplazó el polling periódico por un workflow con **Wait**, reduciendo ejecuciones innecesarias.

## Configuración usada para reducir consumo de IA

```text
Max parallel sub-agents: 1
Tool call concurrency: 1
Max iterations: 10
Reasoning: OFF para la demo
Custom model routing: OFF
```

## Seguridad

Este repositorio **no debe contener**:

- API keys
- App Secret de Meta
- Access Tokens
- credenciales de Google
- números personales
- URLs privadas

Las credenciales deben configurarse directamente en n8n.

Antes de publicar workflows exportados en JSON, revísalos y anonimiza cualquier dato sensible.

## Estructura sugerida del repositorio

```text
peper-assistant/
├── README.md
├── docs/
│   ├── Peper_Assistant_Documentacion_Tecnica.pdf
│   └── screenshots/
│       ├── workflow-principal.png
│       ├── agente-peper.png
│       ├── recordatorio-wait.png
│       └── google-calendar.png
├── workflows/
│   ├── main-workflow.example.json
│   └── reminder-workflow.example.json
└── .gitignore
```

## Privacidad y gestión de datos
Para la integración con Meta / WhatsApp Business Cloud, el proyecto cuenta con páginas públicas de información legal y gestión de datos:
- 📄 Términos y condiciones: https://sites.google.com/view/karen-terminosn8n/inicio
- 🗑️ Eliminación de datos: https://sites.google.com/view/karenm-eliminacion-de-datosn8n/inicio
- 🔐 Política de privacidad: https://sites.google.com/view/karenmn8n/inicio
Estas páginas permiten informar al usuario sobre el tratamiento de datos, las condiciones de uso y el procedimiento para solicitar la eliminación de información asociada al servicio.


## Demo

🎥 **YouTube:** [PEGAR ENLACE AQUÍ]

## Estado del proyecto

**Funcional / proyecto de portafolio.**

Actualmente permite:

- Registrar gastos.
- Crear y consultar eventos.
- Programar recordatorios automáticos.
- Enviar avisos por WhatsApp.
- Mantener información persistente.

## Mejoras futuras

- Soporte multiusuario.
- Recordatorios recurrentes.
- Fallback automático entre modelos de IA.
- Dashboard web.
- Reportes financieros semanales y mensuales.
- Alertas de presupuesto.
- Pruebas automatizadas de workflows.
- Mejor observabilidad y manejo de errores.

## Autor

**Karen Nathalia Martinez**

Proyecto desarrollado como parte de un portafolio técnico enfocado en automatización, integración de APIs e inteligencia artificial aplicada.

---

Si quieres conocer la implementación completa, revisa la documentación técnica incluida en `/docs` y el video de demostración.
