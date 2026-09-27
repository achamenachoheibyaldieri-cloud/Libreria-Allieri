# Sesiones de asistencia de codificación

La consigna pide documentar OpenCode (prompts, respuestas e impacto). En este equipo el desarrollo se asistió con el agente de codificación disponible en el entorno. Las sesiones de abajo son las transcripciones reales de esa asistencia: el pedido, lo que se implementó y el código que quedó en el proyecto. No se inventaron salidas de una herramienta que no se ejecutó.

## Sesión 1 — CRUD, validaciones y reportes

**Prompt.** Consigna del entregable final de Programación IV: app Django con CRUD de una entidad de al menos seis campos, identificador único, validaciones, panel de administración y al menos cinco reportes. La entidad elegida es el libro, alineada con el asistente de ventas.

**Resultado.** Modelo `Libro` con código único, ISBN único, precio y stock no negativos. Vistas de lista, detalle, alta, edición y baja, con mensajes de éxito y error. Admin con `list_display`, filtros y fieldsets. Siete reportes por estrategia. Comando `cargar_catalogo` con diez títulos de demostración.

**Impacto.** El catálogo queda operativo sin datos inventados en tiempo de consulta: los precios y el stock salen de SQLite.

## Sesión 2 — Ollama y restricciones

**Prompt.** Modelfile recibido:

```
FROM llama3.2
PARAMETER temperature 0.3
PARAMETER top_p 0.9
PARAMETER num_ctx 4096
SYSTEM """...asistente exclusivo de venta de libros..."""
```

Más el punto 2 de la consigna: Django envía la pregunta y los datos reales a `http://localhost:11434`; si no hay datos, mensaje predefinido; si Ollama no está, mensaje de error. La web debe abrirse en `http://172.25.4.249:8000/`.

**Resultado.** `libreria/services/ollama_service.py` y `prompt_asistente.py`. `POST /api/chat` repetía el sistema, así que el servicio usa `POST /api/generate` con `raw: true`. Catálogo vacío, ISBN ausente o pregunta ajena a los libros cortan la llamada y devuelven: «Lo siento, no estoy autorizado para responder lo que no esté en el catálogo.» Un fallo de red devuelve el aviso de servicio no disponible.

**Impacto.** La restricción no depende solo de que el modelo «se porte bien»: hay reglas en el servidor antes de llamar a Ollama.

## Sesión 3 — Patrones, pruebas y documentación

**Prompt.** Punto 3: al menos dos patrones (se usaron Factory, Strategy y Observer), pruebas de dos funcionalidades críticas, `README.md`, `DOCUMENTACION.md`, `requirements.txt` con `pip freeze` y `.env.example`.

**Resultado.** `ServiceFactory`, `ReporteStrategy` y señales `post_save` / `post_delete`. La suite completa quedó en 16 pruebas, todas en verde. Documentación de instalación, arquitectura e informe.

**Impacto.** Los reportes y el cliente de Ollama se prueban sin levantar el modelo, con un doble de la sesión HTTP.

## Sesión 4 — Catálogo, ingreso y frase fija

**Prompt.** Diez libros de Halo, El señor de los anillos y Narnia, precios de lista en dólares. Historial solo mientras se está en la consulta. Marca «Libreria Allieri». Quitar ensayo, clásico, distopía y misterio. Ingreso con cualquier correo. Fuera de los libros registrados, responder: «Lo siento, no estoy autorizado para responder lo que no esté en el catálogo.»

**Resultado.** 21 libros en SQLite, middleware de historial, `IngresoView` y géneros recortados en la migración `0002`.

**Impacto.** La web coincide con el catálogo pedido y no inventa títulos que no están cargados.
