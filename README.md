# Educar para Transformar — Campus Virtual

Plataforma de campus virtual para el colegio **"Educar para Transformar"**.
Permite a estudiantes, docentes, padres y administradores gestionar el ciclo educativo completo desde el navegador.

Monorepo con backend en Node.js / Express / Prisma y frontend en HTML + CSS + JavaScript vanilla.

---

## Funcionalidades principales

| Rol | Funcionalidades |
|---|---|
| **Estudiante** | Materias, actividades, notas, asistencia, foros y mensajería interna |
| **Docente** | Planes de estudio, actividades, correcciones, asistencia, calificaciones y comunicación |
| **Padre / Tutor** | Progreso del/los hijo(s), pagos, mensajes y anuncios institucionales |
| **Administrador** | Gestión de usuarios, materias, ciclos lectivos, inscripciones, opiniones y postulaciones laborales |

---

## Requisitos previos

- [Node.js](https://nodejs.org/) >= 20
- [pnpm](https://pnpm.io/) >= 9
- [PostgreSQL](https://www.postgresql.org/) 16

---

## Cómo ejecutar el proyecto

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd CentroEducativo-main
```

### 2. Instalar dependencias

```bash
pnpm install
```

### 3. Configurar variables de entorno

```bash
cp backend/.env.example backend/.env
# Editar backend/.env con los datos de la base de datos y el secreto JWT
```

### 4. Inicializar la base de datos

```bash
cd backend
pnpm prisma migrate dev
pnpm prisma db seed   # (si existe el seed)
```

### 5. Iniciar el backend

```bash
cd backend
pnpm dev
```

### 6. Servir el frontend

Abrir `frontend/index.html` directamente en el navegador, o usar un servidor estático:

```bash
npx serve frontend
```

---

## Estructura del proyecto

```
.
├── backend/        # API REST — Node.js + Express + Prisma + PostgreSQL
│   ├── src/
│   │   ├── routes/     # Endpoints por recurso (auth, admin, docente, etc.)
│   │   ├── middleware/ # Autenticación JWT, manejo de errores
│   │   ├── services/   # Lógica de negocio (mailer, etc.)
│   │   └── db/         # Cliente Prisma
│   └── prisma/         # Esquema y migraciones
└── frontend/       # Interfaz de usuario — HTML + CSS + JS vanilla
    ├── campus.js       # Helpers compartidos: auth, fetch, logout, formato
    ├── script.js       # Lógica de la página de inicio (login, registro)
    ├── panel_admin.html
    ├── panel_docente.html
    ├── panel_estudiante.html
    └── panel_padre.html
```

---

## Integrantes del equipo

- Jáuregui Facundo Manuel
- Alem Nahuel
- ...

---

## Estado
rama de trabajo: `mejora-buenas-practicas`.


## 3. Refactorizaciones (Actividad 3)
### Refactorización 1 — Unificación de `logout()` *(Problema 5)*

**Antes:** `logout()` existía en 5 archivos con 3 implementaciones distintas. Solo `panel_admin.html` avisaba al servidor antes de cerrar sesión; el resto simplemente borraba el `localStorage`, dejando el refresh token activo en el servidor.

**Después:** Se creó `window.logout()` en `campus.js` con la implementación correcta:
```js
window.logout = function () {
    fetch('/api/auth/logout', { method: 'POST', credentials: 'include' }).catch(() => {});
    localStorage.removeItem('usuarioActual');
    sessionStorage.removeItem('token');
    window.location.href = 'index.html';
};
```
Todos los paneles y `script.js` delegan en esta función. La corrección se aplica automáticamente en todos los puntos.

---

### Refactorización 2 — Extracción de `borrarRecurso()` *(Problema 2)*

**Antes:** Tres funciones casi idénticas en `panel_admin.html`:
```js
// Cada una repite: confirmar → DELETE → toast → recargar
async function borrarInsc(id) { /* 5 líneas */ }
async function borrarOp(id)   { /* 5 líneas */ }
async function borrarEmp(id)  { /* 5 líneas */ }
```

**Después:** Se extrae `borrarRecurso()` que encapsula la lógica común; las tres funciones pasan a ser envoltorios de configuración:
```js
async function borrarRecurso({ pregunta, titulo, endpoint, mensajeOk, recargar }) {
    const confirmado = await window.uxConfirm(pregunta, { title: titulo, danger: true, okLabel: 'Eliminar' });
    if (!confirmado) return;
    const respuesta = await window.apiDelete(endpoint);
    if (respuesta.exito) { toastSuccess(mensajeOk); recargar(); }
    else toastError(respuesta.mensaje || 'Error.');
}

async function borrarInsc(id) {
    await borrarRecurso({ pregunta: '¿Eliminar este registro?', titulo: 'Eliminar inscripción',
                          endpoint: '/api/admin/inscriptions/' + id, mensajeOk: 'Eliminado.', recargar: cargarInscripciones });
}
```
Si en el futuro cambia el comportamiento del borrado (log de auditoría, animación, etc.), se modifica **un solo lugar**.
