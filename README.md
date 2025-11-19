## Cómo generar el APK de Android

### 1. Requisitos previos

Antes de nada, asegúrate de tener:

* **Node.js** (versión LTS).
* **Ionic CLI** instalado global:

  ```bash
  npm install -g @ionic/cli
  ```
* **Android Studio** instalado (trae el SDK y las tools de Gradle). ([Ionic Framework][1])
* Java configurado (Android Studio normalmente te instala una versión de JDK).

> Todo lo siguiente hazlo **desde la carpeta raíz del proyecto** (donde está `package.json`).

---

### 2. Instalar dependencias y preparar Capacitor

```bash
# 1. Instala dependencias del proyecto
npm install

# 2. (Solo si aún no tienes Android añadido)
npm install @capacitor/android
npx cap add android
```

Si ya tienes la carpeta `android/` en el proyecto, basta con:

```bash
npm install
```

---

### 3. Generar un APK de prueba (debug)

1. **Build de la app web (Ionic):**

   ```bash
   ionic build
   ```

   Esto genera los archivos en `www/`. ([Ionic Framework][2])

2. **Sincronizar con Android (Capacitor):**

   ```bash
   ionic cap sync android
   ```

   Esto copia lo de `www/` dentro del proyecto nativo Android. ([Capacitor][3])

3. **Abrir el proyecto en Android Studio:**

   ```bash
   ionic cap open android
   ```

4. En **Android Studio**:

   * Espera a que termine de indexar / sincronizar Gradle.
   * Arriba ve a **Build → Build APK(s)** o
     **Build → Build Bundle(s) / APK(s) → Build APK(s)** (según la versión de Android Studio). ([GeeksforGeeks][4])

5. Cuando acabe, Android Studio te mostrará un popup con “Locate”, ahí verás algo como:

   * `android/app/build/outputs/apk/debug/app-debug.apk`

Ese `app-debug.apk` te sirve para instalar en tu celular (activando “instalar apps de origen desconocido”).

> Alternativa por consola (Windows), desde la carpeta `android`:
>
> ```bash
> cd android
> .\gradlew assembleDebug
> ```
>
> ([YouTube][5])

---

### 4. Generar APK firmado (para publicar / release)

1. Build de producción en Ionic:

   ```bash
   ionic build --prod
   ```

2. Sincronizar con Android:

   ```bash
   ionic cap sync android
   ```

3. Abrir Android Studio:

   ```bash
   ionic cap open android
   ```

4. En **Android Studio**:

   * Menú **Build → Generate Signed Bundle / APK…** ([Ionic][6])
   * Selecciona **APK** → Next.
   * Crea un **nuevo keystore** (si aún no tienes):

     * `Create new…`
     * Elige ruta del archivo `.jks`, contraseña, alias, etc.
   * Selecciona:

     * Module: `app`
     * Build type: `release`
   * Next → Finish.

5. Al terminar, el APK firmado suele quedar en:

   * `android/app/build/outputs/apk/release/app-release.apk` ([Ionic][6])

Ese es el APK listo para subir a Play Store (o para compartir como versión release).

> 🔐 Guarda bien el keystore y la contraseña. Sin eso, no podrás actualizar la app en la Play Store en el futuro.

---

### 5. Ojo con Supabase en producción

Antes de hacer el `ionic build --prod` revisa que:

* Tu archivo de configuración (por ejemplo `environment.prod.ts` o donde creas el cliente de Supabase) tenga:

  * `supabaseUrl` correcto.
  * `supabaseKey` correcta (la **anon** key, NO la service_role).

Si en dev te funciona, normalmente solo es copiar esa config a la versión de producción.

# 📱 Tigo Conecta (Examen TIGO) – App móvil con Ionic + Supabase

Aplicación móvil híbrida construida con **Ionic + Angular + Capacitor** que simula
el flujo de contratación de planes móviles TIGO:

- Catálogo de planes para invitados y usuarios registrados.
- Registro e inicio de sesión.
- Chat en tiempo real con un asesor.
- Panel de asesor para gestionar planes y conversaciones.
- Integración completa con **Supabase** (auth, base de datos, storage y realtime).

> Proyecto pensado para ejecutarse como app web (`ionic serve`) y como app móvil Android
  mediante **Capacitor**.

---

## 🧩 Funcionalidades principales

- **Onboarding / Landing pública**
  - Pantallas iniciales con la explicación de la app.
  - Botones de acceso a **login**, **registro** o **catálogo como invitado**.

- **Autenticación de usuarios**
  - Registro de usuario con correo y contraseña.
  - Inicio de sesión con Supabase Auth.
  - Manejo de rol por defecto: `usuario_registrado`.

- **Perfiles de usuario**
  - Tabla `profiles` vinculada a `auth.users`.
  - Trigger en Supabase que crea el perfil al registrarse.
  - Edición de datos básicos (nombre, teléfono, etc.).

- **Catálogo de planes móviles**
  - Tabla `planes_moviles` con:
    - Nombre de plan, precio, segmento (Básico / Medio / Premium).
    - Datos, minutos, SMS, redes sociales, etc.
  - Vista de catálogo:
    - **Invitado**: acceso a planes activos sin necesidad de login.
    - **Usuario**: puede contratar un plan y ver sus contrataciones.

- **Contratación de planes**
  - Tabla `contrataciones` que relaciona `user_id` con un `plan_id`.
  - Estado del plan (activo / cancelado).
  - Historial de planes contratados por el usuario.

- **Chat cliente ↔ asesor**
  - Tabla `mensajes_chat`.
  - Chat en tiempo real (Supabase Realtime) entre:
    - Usuario final.
    - Asesor comercial.
  - Diferenciación visual de mensajes del asesor y del usuario.

- **Panel de asesor**
  - Secciones típicas (dependiendo de tu código):
    - Dashboard de resumen.
    - Editor de planes (`plan-editor`).
    - Lista de conversaciones con usuarios.
    - Perfil del asesor.

---

## 🏗️ Tecnologías utilizadas

- **Ionic Framework** (Angular) :contentReference[oaicite:7]{index=7}  
- **Capacitor** (Android/iOS bridge) :contentReference[oaicite:8]{index=8}  
- **Angular**
- **Supabase**
  - Auth (usuarios y roles).
  - Postgres (tablas, RLS).
  - Storage (bucket de imágenes de planes).
  - Realtime (chat).
- **TypeScript**
- **SCSS** para estilos y theming.

---

## 📁 Estructura general del proyecto

La estructura exacta puede variar, pero típicamente:

```txt
src/
  app/
    core/
      auth.guard.ts
      role.guard.ts
      supabase-client.ts
    pages/
      onboarding/
      login/
      registro/
      public/
        landing/
      user/
        home/          # Catálogo / listado de planes
        tabs/          # Navegación principal usuario
      advisor/
        dashboard/
        plan-editor/
        profile/
        conversations/
  theme/
    variables.scss     # Paleta de colores y tema global
  global.scss          # Estilos globales y layout
````

---

## ⚙️ Configuración de entorno (Supabase)

1. Crea un proyecto en **Supabase**.

2. Obtén:

   * `SUPABASE_URL`
   * `SUPABASE_ANON_KEY`

3. En tu código, busca dónde se crea el cliente de Supabase, por ejemplo:

   ```ts
   import { createClient } from '@supabase/supabase-js';

   const supabaseUrl = 'https://TU_PROYECTO.supabase.co';
   const supabaseKey = 'TU_ANON_PUBLIC_KEY';

   export const supabase = createClient(supabaseUrl, supabaseKey);
   ```

4. Asegúrate de usar **la misma configuración tanto en desarrollo como en producción**
   (por ejemplo, en `environment.ts` y `environment.prod.ts` si los usas).

---

## 🛢️ Base de datos en Supabase

Las tablas mínimas recomendadas (simplificado):

* `profiles`
* `planes_moviles`
* `contrataciones`
* `mensajes_chat`

Ejemplo de campos (resumen):

```txt
profiles
  - id (uuid, PK, FK auth.users.id)
  - email (text)
  - rol ('usuario_registrado' | 'asesor_comercial')
  - nombre_completo, telefono, avatar_url, ...

planes_moviles
  - id (bigserial PK)
  - nombre, precio, segmento
  - datos, minutos, sms, velocidad, redes_sociales, ...
  - activo (boolean)

contrataciones
  - id (bigserial PK)
  - user_id (uuid FK auth.users.id)
  - plan_id (bigint FK planes_moviles.id)
  - estado ('activo' | 'cancelado')
  - fecha_inicio (timestamptz)

mensajes_chat
  - id (bigserial PK)
  - user_id (uuid FK auth.users.id)
  - contenido (text)
  - es_asesor (boolean)
  - created_at (timestamptz)
```

> En tu proyecto puedes tener un archivo `.sql` con el script completo que puedes ejecutar
> en el **SQL Editor** de Supabase.

---

## Políticas de seguridad (RLS) en Supabase

Activa Row Level Security y define políticas, por ejemplo:

* `profiles`

  * Cada usuario ve/actualiza **solo su propio perfil**.
  * Los asesores pueden ver todos los perfiles.

* `planes_moviles`

  * Cualquiera (incluido anon) puede **`SELECT`** sobre planes activos.
  * Solo asesores pueden **insertar / actualizar / borrar** planes.

* `contrataciones`

  * Cada usuario ve sus propias contrataciones.
  * Asesores pueden ver todas (opcional).
  * Usuario crea/actualiza solo sus contrataciones.

* `mensajes_chat`

  * Usuario ve y crea solo sus mensajes.
  * Asesor ve y puede enviar mensajes a todos.

* `storage.objects` (bucket `planes-imagenes`)

  * Cualquiera puede **ver (`SELECT`)** las imágenes del bucket.
  * Solo asesores pueden **subir/editar/borrar** (`INSERT/UPDATE/DELETE`) en ese bucket.

---

## ▶️ Cómo ejecutar el proyecto en desarrollo (web)

1. Instalar dependencias:

   ```bash
   npm install
   ```

2. Levantar el servidor de desarrollo:

   ```bash
   ionic serve
   ```

3. Abrir en el navegador:

   * `http://localhost:8100`

---

## 📲 Cómo correr en dispositivo / emulador Android

1. Compilar la app:

   ```bash
   ionic build
   ```

2. Sincronizar con Android:

   ```bash
   ionic cap sync android
   ```

3. Abrir Android Studio:

   ```bash
   ionic cap open android
   ```

4. Desde Android Studio, selecciona un emulador o un dispositivo real y
   pulsa **Run ▶**.

---

## 🧪 Build web de producción

```bash
ionic build --prod
```

El resultado quedará en `www/`, listo para servir como app web (por ejemplo, en un hosting estático).

---

## 🏗️ Generar APK de Android

### APK de prueba (debug)

```bash
ionic build
ionic cap sync android
ionic cap open android
```

Luego en Android Studio:

* **Build → Build APK(s)**
* Obtendrás `app-debug.apk` dentro de `android/app/build/outputs/apk/debug/`.

### APK firmado (release)

```bash
ionic build --prod
ionic cap sync android
ionic cap open android
```

Después en Android Studio:

* **Build → Generate Signed Bundle / APK…**
* Selecciona **APK**.
* Elige o crea un **keystore**.
* Elige `release` como build type.
* Finaliza el asistente.

El APK firmado aparece en:

* `android/app/build/outputs/apk/release/app-release.apk`

---


### Correo Admin - Admin@admin.com
### Contraseña - Admin123.

