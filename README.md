# Libreria Allieri

Sistema de información en Django 5 para administrar un catálogo de libros (CRUD), siete reportes y un chat con un modelo local de Ollama. El asistente solo responde sobre venta y consulta de libros, y solo con los datos guardados en SQLite.

## Requisitos

- Python 3.11 o superior
- Entorno virtual
- Ollama en ejecución (API en `http://localhost:11434`)
- Modelo `llama3.2:1b` y el modelo derivado `asistente-libros`

## Instalación

```bash
cd proyecto
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python manage.py migrate
python manage.py cargar_catalogo
python manage.py createsuperuser
```

Si `.env` ya existe, no hace falta copiar el ejemplo.

## Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2:1b
ollama create asistente-libros -f Modelfile
```

El `Modelfile` fija `temperature 0.3`, `top_p 0.9`, `num_ctx 4096` y el texto de sistema del asistente de ventas. Django vuelve a enviar ese texto y el catálogo real en cada consulta. Como el proceso local usa la plantilla `chatml`, el servicio escribe los tokens de Llama 3 y llama a `POST /api/generate` con `raw: true`.

En un equipo con 4 GB de RAM, `llama3.2` (3B) llega a ser terminado por el sistema por falta de memoria. Por eso el `Modelfile` de este proyecto usa `llama3.2:1b`, la variante más chica de la misma familia, con el mismo texto de restricciones.

## Ejecución

La aplicación debe abrirse en `http://172.25.4.249:8000/`.

```bash
source .venv/bin/activate
python manage.py runserver 0.0.0.0:8000
```

`ALLOWED_HOSTS` incluye `172.25.4.249`.

Panel de administración: `http://172.25.4.249:8000/admin/`

- Usuario: `admin`
- Contraseña inicial de este entorno: `Libreria2026!`

Conviene cambiar esa contraseña antes de compartir el equipo.

## Pruebas

```bash
python manage.py test libreria
```

## Comandos útiles

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py cargar_catalogo
```

`cargar_catalogo` no duplica códigos.
