# Decisiones técnicas y arquitectura

## Entidad

Se gestiona un catálogo de libros de una librería. El identificador de negocio es `codigo` (único). El ISBN también es único. El modelo tiene once campos de dominio: código, ISBN, título, autor, género, editorial, idioma, año, precio, stock y descripción, más las marcas de tiempo.

El precio y el stock no pueden ser negativos. Esa regla vive en el modelo (`MinValueValidator` y `clean`) y otra vez en el formulario, para que falle tanto el panel de administración como el CRUD de la web.

## Capas

- `libreria/models.py`: persistencia.
- `libreria/forms.py`: validación de entrada.
- `libreria/views.py`: vistas basadas en clases para listar, ver, crear, editar, eliminar, reportes y chat.
- `libreria/services/`: reglas que no deben quedar pegadas a la plantilla.
- `libreria/signals.py`: auditoría.
- SQLite como base local, según la consigna.

La raíz del sitio es el catálogo. No hay autenticación para el CRUD de mostrador; el admin de Django sí exige usuario.

## Patrones

### Factory

`ServiceFactory` crea `OllamaService` y `GestorReportes`. Las vistas piden el servicio y no conocen la URL, el modelo ni el tiempo de espera: salen de variables de entorno.

### Strategy

Cada reporte es una subclase de `ReporteStrategy` con el método `ejecutar`. `GestorReportes` recorre la lista y agrega un resumen más sin tocar la vista. Hay siete estrategias: total, más caro, más barato, stock bajo, agotados, valor de inventario y libros por género.

### Observer

`post_save` y `post_delete` de `Libro` escriben `RegistroAuditoria`. La vista de reportes muestra los últimos movimientos y el admin los deja en solo lectura.

## Integración con Ollama

`OllamaService.consultar`:

1. Si el catálogo está vacío, responde con el mensaje predefinido y no llama al modelo.
2. Si la pregunta trae un ISBN de 10 o 13 dígitos que no está cargado, responde con el mismo mensaje y no llama al modelo.
3. Arma un JSON solo con los campos reales (`a_contexto`).
4. Arma el texto con los tokens de Llama 3 y lo envía a `POST {OLLAMA_URL}/api/generate` con `raw: true`, el system prompt de restricciones y las opciones `temperature 0.3`, `top_p 0.9`, `num_ctx 4096`. En este equipo el servidor local aplicaba la plantilla `chatml`, que Llama 3.2 no interpreta.
5. Si no hay conexión, si el HTTP falla o si el cuerpo no trae `message.content`, el usuario ve el aviso de que Ollama no está disponible.

El modelo configurado es `asistente-libros`, creado con el `Modelfile` a partir de `llama3.2:1b`. La variante de 3B se descargó y el sistema la terminó por falta de memoria en un equipo de 4 GB.

## Variables de entorno

| Variable | Uso |
| --- | --- |
| `SECRET_KEY` | Clave de Django |
| `DEBUG` | Modo depuración |
| `ALLOWED_HOSTS` | Hosts aceptados, incluye `172.25.4.249` |
| `CSRF_TRUSTED_ORIGINS` | Origen del formulario en el puerto 8000 |
| `OLLAMA_URL` | API local, por defecto `http://localhost:11434` |
| `OLLAMA_MODEL` | Nombre del modelo, por defecto `asistente-libros` |
| `OLLAMA_TIMEOUT` | Segundos de espera de la respuesta |

## Calidad

Las pruebas cubren código único, precio y stock negativos, los cálculos de reporte, el observer, el alta desde el cliente HTTP, el catálogo vacío, el ISBN desconocido, el cuerpo enviado a Ollama, la caída del servicio y la idempotencia de `cargar_catalogo`.
