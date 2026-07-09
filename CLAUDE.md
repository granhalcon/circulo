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

## Despliegue (Cloudflare Pages)

- URL: `circulo.pages.dev`
- Variables de entorno requeridas: `GIT_SUBMODULES_RECURSE=1`, `ZOLA_VERSION=0.22.1`

## Tema `enso`

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
- Se subieron 3 libros nuevos a `primeras-letras/content` (build en curso en Cloudflare) — *(actualizar aquí con los nombres cuando se confirme el build)*
- Incidente de pérdida de datos: un pipe con `iconv` sobrescribió su propio archivo fuente; ya recuperado. **Precaución:** nunca usar `iconv archivo > archivo` (mismo archivo como entrada y salida).

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
