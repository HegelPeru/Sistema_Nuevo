# sistema-nuevo-panel

Panel web de **solo lectura** del estado de la señal de SISTEMA_NUEVO
(XAUUSD tendencia / US2000 reversión).

- Lee de **Supabase** (tablas `estado_senal` y `heartbeats`) con la **anon key**
  (solo `SELECT`, nunca escribe).
- **No** habla con los procesos de trading ni con MetaTrader. Podés borrar este
  repo entero y la operativa no se entera.
- La anon key + la URL de Supabase van **cifradas** en `config.enc`; se
  descifran en el navegador con una **contraseña** (Web Crypto, AES-GCM +
  PBKDF2, 200k iteraciones).

Diseño completo: `SISTEMA_NUEVO/docs/DISENO_PANEL_VISUAL.md` (repo privado del
proyecto de trading).

## Contenido

| Archivo | Qué es |
|---|---|
| `index.html` | El panel. Todo el markup/CSS/JS. Único archivo que sirve GitHub Pages. |
| `config.enc` | `{supabase_url, anon_key}` cifrado con la contraseña. **Lo generás vos** (ver abajo). |
| `generar_config.html` | Herramienta local para crear `config.enc`. No hace falta subirla (si la subís, no pasa nada: no tiene secretos). |
| `README.md` | Esto. |

**Nunca** subir a este repo: nada del código Python de SISTEMA_NUEVO, ningún
`.env`, la `service_role` key, tokens de Telegram, logs, journals, ni reportes
de `docs/`.

## Probar local (antes de publicar)

```
cd sistema-nuevo-panel
python -m http.server 8080
```

Abrir `http://localhost:8080`. (Hace falta `config.enc` — ver abajo. Sin él, el
panel muestra "Falta config.enc".)

## Generar `config.enc`

1. Abrir `generar_config.html` en el navegador (doble clic, o
   `http://localhost:8080/generar_config.html`).
2. Completar:
   - **SUPABASE_URL**: Supabase → Project Settings → API → "Project URL".
   - **anon key**: Supabase → Project Settings → API → Project API keys →
     **`anon` / `public`** (NO la `service_role`).
   - **Contraseña**: la que vas a compartir con quien pueda ver el panel.
3. "Generar" → "Descargar config.enc".
4. Dejar `config.enc` en la misma carpeta que `index.html`.

Para **cambiar la contraseña** o la key: volver a generar `config.enc` y
`git push`. (Ojo: quien tuvo la contraseña alguna vez pudo haberse guardado la
anon key; para cortar eso de raíz hay que rotar la anon key en Supabase.)

## Publicar en GitHub Pages

1. **Crear el repo** en GitHub: `New repository` → nombre `sistema-nuevo-panel`
   → **Public** → Create.
2. **Subir** `index.html` + `config.enc` (+ `README.md`, `generar_config.html`)
   a la rama `main`:
   ```
   git init
   git add index.html config.enc README.md generar_config.html
   git commit -m "panel v1"
   git branch -M main
   git remote add origin https://github.com/<usuario>/sistema-nuevo-panel.git
   git push -u origin main
   ```
3. **Activar Pages**: repo → **Settings → Pages** → "Build and deployment" →
   Source: **Deploy from a branch** → Branch: `main` / `/ (root)` → **Save**.
4. Esperar ~1 min. La URL aparece en esa misma pantalla:
   **`https://<usuario>.github.io/sistema-nuevo-panel/`**
5. Abrirla, poner la contraseña. En el celular: "Agregar a pantalla de inicio".

**Actualizar el panel a futuro:** editar `index.html`, `git push` a `main`,
Pages re-publica solo en ~1 min.

## Qué necesita del lado de Supabase

Las tablas y policies las crea el proyecto de trading (ver
`docs/DISENO_PANEL_VISUAL.md` §1.2). En resumen: la tabla `estado_senal` con
RLS activo + una policy de `SELECT` pública, e igual policy de `SELECT` para
`heartbeats`. La tabla `operaciones` queda cerrada (el panel no la usa).
