# ERP no-code — prototipo (tipo Airtable) conectado a Supabase

Prototipo de un solo archivo (`index.html`, vanilla JS + [mathjs](https://mathjs.org)) de una
herramienta no-code tipo Airtable: tablas con columnas tipadas, motor de fórmulas propio (20
funciones en español: `SI`, `SIERROR`, `SUMA`, `Y`, `O`, `MAYUSC`...), filtro/orden/agrupación,
multi-tabla con vínculos entre registros, y conexión real a Supabase (Postgres) con el patrón
`app_tables` / `app_fields` / `app_records` / `app_record_links`.

## Por qué existe este repo

Este repo tiene un propósito específico además de servir el prototipo: **aislar si un error de
Supabase (`DataCloneError: Headers object could not be cloned`) era causado por el sandbox de
Claude Artifacts, o por algo real del código/Supabase.**

Investigamos el error dentro del entorno de Artifacts y encontramos evidencia de que el sandbox
intercepta `window.fetch` para relevar peticiones fuera del iframe vía `postMessage` — y que un
objeto `Headers` nativo (que cualquier `fetch()` real siempre produce en la respuesta) no
sobrevive ese relevo. La corrección aplicada fue reemplazar `fetch` por una implementación
propia basada en `XMLHttpRequest` (`xhrFetch`, en `index.html` y también en `test-supabase.html`).

**`test-supabase.html`** es la prueba de esa hipótesis: corre la misma consulta a Supabase con
`fetch` nativo y con `xhrFetch`, lado a lado, y muestra un veredicto. Fuera del sandbox de
Artifacts (en Vercel, en un navegador normal), **ambas deberían funcionar** — si es así, confirma
que el problema era exclusivo del entorno del Artifact, no de Supabase ni del código.

## Desplegar en Vercel

No hace falta configurar nada manualmente — `vercel.json` ya deja todo listo para un sitio
estático sin build.

1. Sube esta carpeta a un repositorio de GitHub (ver abajo).
2. Entra a [vercel.com/new](https://vercel.com/new) e importa ese repositorio.
3. Vercel detecta automáticamente que es un sitio estático (gracias a `vercel.json`) — no
   selecciones ningún framework, no hace falta build command ni output directory, ya vienen
   definidos.
4. Dale "Deploy". En menos de un minuto tienes una URL pública (`tu-proyecto.vercel.app`).

### Subir esta carpeta a GitHub desde cero

```bash
cd carpeta-del-proyecto
git init
git add .
git commit -m "Prototipo inicial: ERP no-code + diagnóstico de Supabase"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
git push -u origin main
```

## Cómo interpretar el diagnóstico

Abre `tu-proyecto.vercel.app/test-supabase.html`, dale clic a "Correr diagnóstico", y compara:

| Resultado | Qué significa |
|---|---|
| ✅ Ambos funcionan | El `DataCloneError` era exclusivo del sandbox de Claude Artifacts. Supabase y el código siempre estuvieron bien — en producción (Vercel) no vas a ver este error, con o sin el arreglo de `xhrFetch`. |
| ⚠️ Solo xhrFetch funciona | El problema no era 100% exclusivo del Artifact — hay que revisar el error específico que muestra el fetch nativo. |
| ❌ Ambos fallan | El problema no es el sandbox — revisar credenciales, RLS, o la tabla `app_tables` en Supabase. |

## ⚠️ Antes de compartir este repo o el despliegue con alguien más

- **La anon key de Supabase está embebida en el HTML a propósito** — es el patrón esperado para
  un prototipo de un solo archivo sin backend propio. No es un secreto de verdad (está diseñada
  para exponerse en el cliente), pero sí controla qué puede hacer cualquiera que la tenga.
- **RLS (seguridad a nivel de fila) está en modo prueba**: abierto también a usuarios anónimos
  (`anon`), no solo a `authenticated`. Esto es intencional mientras no exista login — pero
  significa que cualquiera con la URL desplegada puede leer y escribir datos reales en la base.
  **No compartas la URL de Vercel públicamente ni la uses con datos reales de un cliente** hasta
  conectar Supabase Auth y cerrar las políticas de `anon`.

## Estado del proyecto / próximos pasos

- [x] Prototipo funcional: columnas tipadas, motor de fórmulas, filtro/orden/grupo, multi-tabla
- [x] Conexión a Supabase (esquema `app_tables`/`app_fields`/`app_records`/`app_record_links`)
- [x] Diagnóstico y corrección del `DataCloneError` en el sandbox de Artifacts
- [ ] Supabase Auth (login real)
- [ ] Cerrar RLS a `authenticated` únicamente (quitar acceso `anon`)
- [ ] Mover las credenciales a variables de entorno (al migrar a Next.js)
- [ ] Migración a Next.js (stack objetivo del proyecto)
