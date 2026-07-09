# Círculo Solar — Contexto del proyecto

> Este archivo existe para que Claude pueda retomar el proyecto en un chat nuevo sin necesitar todo el historial de conversación. Actualízalo al cierre de cada sesión importante.

## Qué es esto

Red de micrositios estáticos con Zola, contenido gnóstico/esotérico dentro de la tradición de Samael Aun Weor (SAW). Repo principal `granhalcon/circulo`, con 6 submódulos independientes en GitHub bajo `granhalcon/`.

## Entorno de trabajo

- Sistema: Linux/Debian (antes Chromebook)
- Shell: **Nushell** — `&&` no funciona, hay que usar líneas separadas; algunos binarios de sistema chocan con builtins de Nu (p. ej. `^find` para forzar el binario del sistema)
- Editor: Helix v25.07.1
- Python: 3.11 con `.venv`
- Estructura local: `~/Sites/circulo-solar/<nombre>/` (clones standalone de cada submódulo)

## Estructura del repo

```
~/Sites/circulo-solar/
├── circulo/                          # repo principal (Zola root)
│   └── submodules/
│       ├── circulo-de-iniciacion/
│       ├── biblioteca-de-consulta-saw/
│       ├── quinto-evangelio/
│       ├── primeras-letras/
│       ├── pintas-ayahuasqueras/
│       └── circulos-de-medicina/
├── circulo-de-iniciacion/            # clon standalone (independiente del submódulo)
├── biblioteca-de-consulta-saw/
├── quinto-evangelio/
├── primeras-letras/
├── pintas-ayahuasqueras/
└── circulos-de-medicina/
```

**IMPORTANTE:** los clones standalone en `~/Sites/circulo-solar/<nombre>/` y los submódulos en `circulo/submodules/<nombre>/` son git-repos separados. Un cambio en uno NO se refleja automáticamente en el otro.

## Flujo de trabajo obligatorio tras editar contenido/tema

1. Editar y hacer commit/push en el clon standalone del submódulo correspondiente.
2. Ir al repo principal `circulo` y actualizar el puntero:
   ```nu
   cd ~/Sites/circulo-solar/circulo
   git submodule update --remote submodules/<nombre>
   git add submodules/<nombre>
   git commit -m "Actualizar puntero de <nombre>"
   git push
   ```
3. Si no se hace el paso 2, **Cloudflare Pages construye con el commit viejo** — este es el bug recurrente más común del proyecto.

### Caso especial: submódulos anidados (`enso` dentro de cada submódulo de `circulo`)

`themes/enso` es un submódulo dentro de CADA uno de los 6 submódulos de `circulo` (submódulo-dentro-de-submódulo). Esto añade dos trampas adicionales, confirmadas en la práctica:

- **Timing:** si corres `git submodule update --remote themes/enso` en un sitio *antes* de que el push a `enso` termine de propagarse en GitHub, capturas el commit viejo sin ningún error visible — el comando "funciona" pero trae la versión anterior. Siempre verificar con `git log -1 --oneline` dentro de `themes/enso` después de actualizar, no asumir que quedó bien.
- **No-inicialización en cascada:** actualizar el puntero de un submódulo en `circulo` (`git submodule update --remote submodules/<nombre>`) NO actualiza automáticamente el submódulo anidado `themes/enso` dentro de ese checkout. Hace falta explícitamente, parado en `submodules/<nombre>/`:
  ```nu
  git submodule update --init themes/enso
  ```
  o desde `circulo` con `--recursive`:
  ```nu
  git submodule update --remote --recursive submodules/<nombre>
  ```
- **Verificación final recomendada** antes de dar por cerrada una actualización de tema:
  ```nu
  git submodule status --recursive | rg enso
  ```
  Todas las líneas deben mostrar el mismo hash, sin prefijo `+` (desincronizado) ni `-` (no inicializado).

## Despliegue (Cloudflare Pages)

- URL: `circulo.pages.dev`
- Variables de entorno requeridas: `GIT_SUBMODULES_RECURSE=1`, `ZOLA_VERSION=0.22.1`

## Tema `enso`

**Regla de edición:** `enso` es un repo independiente (`~/Sites/circulo-solar/enso/`), referenciado como submódulo `themes/enso` en CADA sitio que lo usa. Nunca editar el tema desde dentro del checkout de un sitio (ej. `primeras-letras/themes/enso`) — aunque técnicamente funciona (es un repo git completo), genera confusión y riesgo de perder cambios. Editar siempre desde `~/Sites/circulo-solar/enso/`, hacer push ahí, y luego propagar a TODOS los sitios que usan el tema:
```nu
cd ~/Sites/circulo-solar/<sitio>
git submodule update --remote themes/enso
git add themes/enso
git commit -m "Actualizar tema enso"
git push
```
...y después actualizar el puntero de ese submódulo en `circulo`. Si se edita por accidente desde dentro de un sitio, sincronizar el clon standalone con `git pull` antes de seguir.

- `document.html` y `chapter.html` unificados bajo un solo sistema de layout (`.reading-sidebar` / `.reading-layout`)
- `book.html`, `section.html`, `index.html` colapsados en wrappers delgados sobre `partials/portada.html`
- `book.html` detecta si un libro tiene subpáginas de capítulo y renderiza modo portada vs. modo lectura (con TOC lateral)
- Clases CSS muertas eliminadas
- `.gitignore` (excluyendo `public/`) agregado a todos los repos
- **Constraint clave de Zola:** los shortcodes NO se heredan del tema — deben vivir en `templates/shortcodes/` de cada sitio. Cualquier carpeta `templates/` local requiere `theme = "enso"` explícito en `zola.toml`

## Scripts Python (pipeline de contenido)

- `convertidor_qe.py` — procesa "El Quinto Evangelio": 11 carpetas numeradas de conferencias/libros de SAW → carpetas planas con slug bajo `content/quinto-evangelio/`, usa `extra.decade` para agrupar dinámicamente
- `convertidor_libros.py` — convierte ~500 archivos Markdown de Jekyll a TOML de Zola, 3 casos de contenido (A/B/C), normaliza licencias, maneja frontmatter sucio
- `quitar_toc.py` — elimina TOCs de anclas internas estilo Jekyll
- `publish_book.py` — procesa libros: separa en capítulos, distribuye notas al pie correctamente

Colecciones ya convertidas: `biblioteca-de-consulta-saw`, `circulo`.

## Estado actual / últimos temas activos

- Resolución de "submodule pointer drift" (commits viejos del tema `enso` rompiendo builds)
- Unificación de templates de lectura (`enso`)
- Se publicaron 3 libros nuevos en `primeras-letras` vía `split_book.py`, incluyendo "Alce Negro Habla" (John G. Neihardt), con imágenes en `static/`
- Incidente de pérdida de datos: un pipe con `iconv` sobrescribió su propio archivo fuente; ya recuperado. **Precaución:** nunca usar `iconv archivo > archivo` (mismo archivo como entrada y salida).

### Fixes aplicados al tema `enso` (commit `6b0eaf9`)

- `document-metadata.html`: ahora es **retrocompatible** con dos formatos de `extra.metadata` — formato antiguo (lista `[[extra.metadata]]` con `label`/`value`/`featured`, usado por ~300 archivos en `quinto-evangelio`) y formato nuevo (mapa `[extra.metadata]` clave-valor, usado en los libros nuevos de `primeras-letras`). Detecta cuál es con `{% if meta.0 is defined %}` (existe en listas, no en mapas).
- `portada.html`: la imagen de portada ahora se resuelve como `media/<slug>/<cover>` en vez de solo `<cover>`, usando `portada_slug = portada_source.components | last`.

### Pendiente: mejorar `split_book.py`

El script asigna correctamente `weight` a cada capítulo (según el prefijo numérico del archivo, que refleja el orden narrativo real), pero **no agrega `sort_by = "weight"` al `_index.md`** de la sección del libro. Sin esa línea, Zola no respeta el `weight` y el índice/TOC del libro aparece en orden arbitrario. Hubo que corregirlo manualmente en los 3 libros ya publicados. **Falta:** agregar `sort_by = "weight"` a la plantilla de `_index.md` que genera el script, para que los próximos libros no repitan este bug.

## `circulos-de-medicina`

- Eventos ceremoniales en 3 fechas fijas anuales: 23 de junio, 16 de julio, 8 de diciembre
- Gestión vía frontmatter Markdown
- Tarjetas de evento con Open Graph para compartir en WhatsApp
- Pendiente: generación de PDF/EPUB vía Pandoc + Nushell, y un "cuadernillo" impreso para eventos al aire libre

## Preferencias de trabajo del usuario

- Al editar/resumir textos fuente, usar **solo** la terminología del documento original — sin inferencias, paráfrasis ni conceptos introducidos que no estén en el texto
- GitHub: `granhalcon`

## Pendientes conocidos

- Integración de Pagefind (buscador) — diferida hasta terminar el build de contenido
- Generación PDF/EPUB para `circulos-de-medicina`
