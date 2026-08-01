# HHM Proyectos — Sistema Administrativo

> Documento de contexto del proyecto. Si eres una IA que acaba de conectarse a
> este repositorio, **lee este archivo completo antes de hacer cambios**. Aquí está
> el propósito, cómo funciona la app, las reglas de trabajo y las decisiones tomadas.

---

## 1. Propósito y contexto

Esta es la **app administrativa de HHM Proyectos**, una empresa de **instalaciones
eléctricas y plomería** ubicada en **Escobedo, Nuevo León, México**.

El dueño (usuario del repo) la está construyendo desde una versión beta hasta la
versión final, para usarla como la "administradora" de su negocio. Reemplaza un
Excel (`sistema_administrativo_completo-2.xlsx`, incluido en el repo como referencia
de los datos reales).

**La app se usa la mayor parte del tiempo en el celular.**

## 2. Qué es la app

Es una **aplicación web de una sola página** (todo vive en `index.html`). Secciones:

- **Inicio / Dashboard** — resumen: obras activas, valor de inventario, materiales
  bajo mínimo, herramientas en campo, alertas.
- **Datos de la empresa** — nombre, RFC, teléfono, correo, dirección (alimentan el PDF).
- **Obras / Proyectos** — tarjetas con estado (Activa/Pendiente/Completada/Pausada),
  cliente, responsable. Botón "Ver Detalle" muestra movimientos y herramientas de esa obra.
- **Cotizador** — genera cotizaciones profesionales; los conceptos se eligen del
  Catálogo de Precios de Venta (precio y unidad automáticos) o como "concepto libre".
  Calcula subtotal + IVA (16%) + total. Exporta a **PDF con membrete HHM** (vía impresión).
- **Historial de Cotizaciones** — cotizaciones guardadas, con desglose, PDF y estado.
- **Catálogo de Precios de Venta** — lista de cobro al cliente (material + mano de obra).
- **Catálogo de Materiales** — costo de referencia y stock mínimo.
- **Inventario de Bodega** — stock actual vs. mínimo; el **Estado (Óptimo/Reabastecer)
  es automático** (se calcula: stock <= mínimo → Reabastecer).
- **Movimientos de inventario** — entradas/salidas/consumos/devoluciones.
- **Catálogo de Herramientas**, **Asignaciones a Personal**, **Ubicación de Herramientas**.

Cada una de las 9 secciones de datos tiene **Agregar, Editar y Eliminar** (con confirmación).

## 3. Objetivo (qué se busca conseguir)

- Una herramienta **profesional, sencilla y bonita** que el dueño y su equipo usen a
  diario desde el celular.
- **Datos guardados en la nube** y **sincronizados** entre todos los dispositivos.
- **Privada**: solo personal autorizado entra.
- Que la **cotizadora** genere PDFs presentables para enviar a clientes.

## 4. Arquitectura técnica

- **Frontend:** un único archivo `index.html`. HTML + Tailwind (vía CDN
  `cdn.tailwindcss.com`) + Font Awesome (CDN) + JavaScript vanilla, todo inline. **No
  hay build ni frameworks.** Se edita el HTML directamente.
- **Tema visual:** sistema de color de marca (`brand`) manejado por variables CSS,
  con selector de tonalidad (Azul HHM por defecto, Azul Marino, Cian, Grafito).
  Preferencia guardada en `localStorage`.
- **Backend / datos:** **Firebase**.
  - **Firestore**: toda la base vive en **un solo documento** `hhm/datos` (objeto
    JSON con todas las colecciones). Se sincroniza en tiempo real con `onSnapshot`.
  - **Auth**: **correo + contraseña** (`signInWithEmailAndPassword`).
  - **Hosting**: la app se publica en `https://administradorahhm.web.app`.
- **Repo:** el código está en GitHub. Se trabaja en la rama **`main`**.

### Archivos del repo
- `index.html` — la app completa (esto es lo que se edita).
- `manifest.json` — PWA (ícono, pantalla completa `display: standalone`).
- `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` — íconos de pantalla de inicio.
- `HHM_PROYECTOS_LOGO.jpeg` — logo original.
- `firebase.json` — config de Hosting (incluye headers no-cache para el HTML).
- `.firebaserc` — proyecto por defecto (`administradorahhm`).
- `firestore.rules` — reglas de seguridad (referencia; se publican en la consola).
- `sistema_administrativo_completo-2.xlsx` — Excel original con los datos reales (referencia).

## 5. Modelo de datos (`appData` en index.html)

Objeto global `appData` con estas colecciones (todas se guardan en Firestore `hhm/datos`):
`empresa`, `materiales`, `bodega`, `movimientos`, `obras`, `preciosVenta`,
`herramientas_cat`, `herramientas_asig`, `herramientas_ubi`, `cotizaciones`.

- Al guardar cualquier cambio se llama a **`persistir()`** (escribe todo a Firestore).
- El sistema de **Agregar/Editar/Eliminar es genérico**: se define en el objeto
  **`COLECCIONES`** (esquema de campos por sección) y usa el modal `#modal-editor`.
  Para agregar una sección nueva editable, se añade su entrada a `COLECCIONES`.

## 6. Autenticación y seguridad

- Acceso con **correo + contraseña**. Las cuentas se crean **manualmente** en
  Firebase → Authentication → Users.
- **Lista blanca de correos** en dos lugares (deben coincidir):
  1. `CORREOS_AUTORIZADOS` en `index.html`.
  2. La función `autorizado()` en las **reglas de Firestore** (se publican en la consola).
- Correos autorizados actuales:
  - `azpek2524@gmail.com`
  - `hectorhugo.martinez@hotmail.com`
  - `hugoslavia.martinez@gmail.com`
- **Para autorizar a alguien nuevo:** agregar su correo en (a) `CORREOS_AUTORIZADOS`,
  (b) las reglas de Firestore (publicar), y (c) crear el usuario en Authentication → Users.
- Las reglas **no** exigen `email_verified` (las cuentas de correo/contraseña no lo tienen).

## 7. Despliegue (cómo publicar cambios)

El código en GitHub **no se publica solo**. El dueño publica desde su carpeta local
(clon de `main`, llamada `HHM-APP` en su PC):

```powershell
git pull
firebase deploy --only hosting --project administradorahhm
```

- Los **datos** (obras, cotizaciones, etc.) sí se sincronizan solos vía Firebase; solo
  el **código** requiere deploy.
- El `index.html` está configurado como **no-cache**, así que tras `deploy` los cambios
  se ven al recargar.
- Requisitos en la máquina: Node.js, `npm install -g firebase-tools`, `firebase login`
  (con la cuenta de Google dueña del proyecto). En Windows quizá haga falta
  `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

## 8. Reglas y convenciones para trabajar

- **Trabajar siempre en la rama `main`.** (Históricamente hubo una rama
  `claude/hhm-proyectos-app-beta-x8e00w` que ya se fusionó a main.)
- **Editar `index.html` directamente**, con cuidado (es un solo archivo grande).
- **Probar antes de subir.** No hay tests automáticos; validar la lógica (idealmente
  cargando el HTML con un mock de Firebase, ya que los CDN/Firebase pueden estar
  bloqueados en entornos sandbox).
- **Commits claros** y hacer `push` a `main`.
- **No romper el login ni las reglas** al hacer cambios.
- El usuario **no es programador**: darle instrucciones simples, paso a paso, y evitar
  que asistentes dentro de VS Code editen el archivo (han introducido bugs, p. ej. un
  typo `initializeAApp`). El repo es la fuente de verdad.
- Comunicación con el usuario: **en español**, claro y directo.

## 9. Decisiones importantes (y por qué)

- **Login por correo/contraseña en vez de Google:** con Google + pantalla completa
  (PWA standalone) en iPhone, el inicio de sesión se rompe (la sesión no vuelve tras
  el redirect de Google). Correo/contraseña sí funciona en pantalla completa.
- **Estado de Bodega automático:** se calcula desde stock vs. mínimo; no se captura a mano.
- **Toda la base en un solo documento Firestore:** simple y suficiente para el volumen
  actual (cientos de registros). Last-write-wins.
- **Tailwind/Font Awesome por CDN:** requieren internet; en el navegador del usuario
  cargan bien. En entornos sandbox pueden estar bloqueados (afecta solo las pruebas).

## 10. Ideas / pendientes (a confirmar con el dueño)

- Retroalimentación continua del dueño y su cliente (se trabaja por tandas).
- Posibles mejoras futuras: reportes, filtros, edición de desglose en historial, etc.

## 11. Datos clave

- **URL en vivo:** https://administradorahhm.web.app
- **Proyecto Firebase:** `administradorahhm`
- **Repo GitHub:** https://github.com/azpek2524-prog/ADMINISTRADORAHHM
- La `firebaseConfig` (apiKey, etc.) está en `index.html`. **No es secreta** (así se usa
  en apps web; la seguridad la dan Auth + las reglas de Firestore).
