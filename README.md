# 🏆 CHETOS DE ORO

> *Premiando a los personajes que hacen única a nuestra manada.*

Aplicación web de votación para los premios universitarios **Chetos de Oro** de la **Facultad de Matemáticas (FMAT)**. Los usuarios se registran, inician sesión y votan **una sola vez por categoría**. La regla está protegida en el backend **y** en la base de datos (`UNIQUE(user_id, category_id)`).

## 1. Tecnologías

| Capa | Tecnología |
|---|---|
| Frontend | HTML + CSS + JavaScript vanilla (sin frameworks) |
| Backend | Node.js + Express |
| Base de datos | SQLite (módulo integrado `node:sqlite`) |
| Seguridad | Contraseñas con hash `bcryptjs`, sesiones con JWT |

## 2. Estructura

```
chetos-de-oro/
├── index.html · login.html · registro.html · categorias.html
├── votacion.html · mis-votos.html · resultados.html · admin.html
├── css/        style.css (global + admin) · login.css · votacion.css
├── js/         main.js (API, sesión, cabecera) · auth.js · votacion.js · admin.js · home.js
├── img/        logo/ · candidatos/ · categorias/ · fondos/ · premios/
├── backend/
│   ├── server.js · database.js · seed.js · create-admin.js
│   ├── routes/ (auth, api, admin)
│   ├── controllers/ (authController, publicController, adminController)
│   └── middleware/ (auth.js)
├── database/   chetos-de-oro.sqlite  ← se crea sola, no se sube a GitHub
├── .env.example · .gitignore · package.json · README.md
```

## 3. Instalar Node.js y dependencias

1. Instala **Node.js 22.5 o superior** (recomendado la versión LTS actual): <https://nodejs.org>. Verifica con `node -v`.
2. En la carpeta del proyecto:
   ```bash
   npm install
   cp .env.example .env     # en Windows: copy .env.example .env
   ```
3. Abre `.env` y cambia `JWT_SECRET` por una cadena larga y aleatoria.

## 4. Ejecutar

```bash
npm start
```
Abre <http://localhost:3000>. La primera vez se crea `database/chetos-de-oro.sqlite` con las tablas y las 8 categorías.

Para probar con datos de demostración (3 candidatos ficticios por categoría):
```bash
npm run seed
```

## 5. Crear el usuario administrador

1. Regístrate normalmente, o usa directamente el comando (crea o promueve la cuenta):
   ```bash
   npm run create-admin -- "Tu Nombre" admin@correo.com MiContraseñaSegura123
   ```
2. Inicia sesión: aparecerá el enlace **Admin** (`admin.html`).
3. Desde el panel puedes agregar/editar categorías y candidatos, ver votos y usuarios, y **habilitar o deshabilitar los resultados**. Los usuarios normales reciben 403 en toda la API de administración.

## 6. Cómo funciona la base de datos

Tablas: `users`, `categories`, `candidates`, `votes` y `settings` (guarda si los resultados son públicos).

```sql
CREATE TABLE votes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL, category_id INTEGER NOT NULL, candidate_id INTEGER NOT NULL,
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  UNIQUE (user_id, category_id)   -- un voto por usuario y categoría
);
```
Si alguien intenta votar dos veces (desde la web, con `curl` o directo en SQL), la base de datos lo rechaza y la API responde `409`.

Como el archivo `.sqlite` contiene usuarios y votos reales, **está en `.gitignore`**: cada instalación crea su propia base. Para reiniciar la votación, detén el servidor y borra `database/chetos-de-oro.sqlite*`.

## 7. Agregar categorías

**Opción A (panel admin):** pestaña *Categorías* → nombre, imagen y descripción → *Guardar*.

**Opción B (SQL):**
```sql
INSERT INTO categories (slug, name, description, image, sort_order)
VALUES ('lobo-cocinero', 'LOBO COCINERO', 'Quien nos salva con su comida.', 'img/categorias/lobo-cocinero.jpg', 9);
```

## 8. Agregar candidatos

Panel admin → *Candidatos* → nombre, categoría y nombre de la foto. Cada candidato pertenece a **una** categoría. Los candidatos con votos no pueden borrarse ni cambiarse de categoría.

## 9. Dónde colocar las imágenes

Todas se cargan por **ruta relativa**; ninguna está incrustada en Base64.

| Carpeta | Qué poner | Archivo esperado |
|---|---|---|
| `img/logo/` | Logo de Chetos de Oro y de FMAT | `logo-chetos.png` (cabecera), `logo-fmat.png` (portada) |
| `img/candidatos/` | Fotos de los nominados (vertical 4:5, ej. 800×1000) | `candidato_01.jpg`, `candidato_02.jpg`… |
| `img/categorias/` | Ilustración de cada categoría (16:9, ej. 960×540) | `<slug>.jpg`, ej. `lobo-fosil.jpg` |
| `img/fondos/` | Fondos decorativos | `fondo-principal.jpg` (opcional) |
| `img/premios/` | Estatuilla del cheto dorado (PNG con transparencia) | `cheto-dorado.png` |

Si una imagen no existe, se muestra automáticamente el `placeholder` de esa carpeta. Los logos se ocultan si no existen.

### Cambiar el logo
Guarda tu logo como `img/logo/logo-chetos.png` (y `logo-fmat.png`). Si quieres otro nombre, edita `renderHeader()` en `js/main.js` (comentario `LOGO`) y `index.html` (comentario `LOGO FMAT`).

### Cambiar los fondos
Coloca `img/fondos/fondo-principal.jpg`. Para otro nombre, cambia `--bg-image` al inicio de `css/style.css`.

### Cambiar la estatuilla
Reemplaza `img/premios/cheto-dorado.png` (se usa en portada, mensaje de voto y resultados).

### Agregar o cambiar la fotografía de un candidato
1. Coloca la imagen en `img/candidatos/`.
2. Nómbrala, por ejemplo, `candidato_nombre.jpg`.
3. Registra la ruta: en el panel admin → *Candidatos* → *Editar* → campo *Foto* = `candidato_nombre.jpg`. O por SQL:
   ```sql
   UPDATE candidates SET image = 'img/candidatos/candidato_nombre.jpg' WHERE id = 3;
   ```

### Imágenes de categorías
Por defecto cada categoría busca `img/categorias/<slug>.jpg`, con estos slugs: `lobito-en-ascenso`, `lobito-migajero`, `fiestero-de-alto-rendimiento`, `lobo-foraneo`, `el-10-10`, `la-10-10`, `lobo-deportista`, `lobo-fosil`.

## 10. API (resumen)

| Método | Ruta | Acceso |
|---|---|---|
| POST | `/api/auth/register`, `/api/auth/login` | público |
| GET | `/api/categories` | público (marca votos si hay sesión) |
| GET | `/api/categories/:id/candidates` | sesión |
| POST | `/api/votes` | sesión (409 si ya votó en esa categoría) |
| GET | `/api/my-votes` | sesión |
| GET | `/api/results` | público **solo si el admin lo habilitó** |
| `*` | `/api/admin/...` | solo administradores |

## 11. Notas de seguridad

- Cambia `JWT_SECRET` antes de publicar el sitio.
- Si lo expones en internet, ponlo detrás de HTTPS (por ejemplo con un proxy inverso) y considera limitar intentos de login.
- Solo `css/`, `js/`, `img/` y las páginas HTML se sirven al público; `backend/` y `database/` no.
