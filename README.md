# Tutor Virtual ESUCAR — Trabajo de Título

Aplicación móvil Android (Kotlin + Jetpack Compose) que funciona como **tutor virtual** para la asignatura
Trabajo de Título I, II y III de la Escuela de Suboficiales de Carabineros (caso DSY1105, Duoc UC).
Guía paso a paso la construcción de la tesis, centraliza pautas y FAQ, y permite hacer seguimiento de entregables con recordatorios locales.

**Integrantes:** _Nombre Apellido 1_ · _Nombre Apellido 2_  (completar)  
**Equipo N°:** _(completar)_  
**Asignatura:** DSY1105 – Desarrollo de Aplicaciones Móviles · Evaluación Parcial 2

## Funcionalidades

| HU | Funcionalidad | Estado |
|----|---------------|--------|
| HU-01 | Listado de etapas con estado, descripción y fecha límite (offline) | ✅ |
| HU-02 | Detalle de etapa: descripción, formato, fecha límite y pauta | ✅ |
| HU-03 | Guía paso a paso con "Anterior/Siguiente" y progreso ("Paso 2 de 5") | ✅ |
| HU-04 | FAQ por categoría con respuestas desplegables | ✅ |
| HU-05 | Búsqueda en tiempo real (sin distinguir mayúsculas ni tildes) | ✅ |
| HU-06 | Estado visual de cada entregable (color + ícono + etiqueta), persistente | ✅ |
| HU-07 | Actualizar estado, fecha de entrega, notas y foto de evidencia | ✅ |
| HU-08 | Checklist ordenado por fecha, con alerta ≤ 3 días | ✅ |
| HU-09 | Recordatorios locales por hito (WorkManager, sin internet) | ✅ |
| HU-10 | Pauta/rúbrica simplificada por etapa | ✅ |
| HU-11 | Actualización de contenidos con conexión | ⏳ Pendiente (el caso no incluye servidor en el MVP; se aborda en la EFT) |
| HU-12 | Interfaz accesible, alto contraste, adaptable a teléfono y tablet | ✅ |

Además: registro e inicio de sesión local (contraseña con PBKDF2 + sal) para separar el avance de cada estudiante.

## Arquitectura (MVVM)

```
UI (Compose)  →  ViewModel (StateFlow)  →  Repository  →  Room / DataStore / assets
                        ↓
                 domain/Validators  (validaciones puras, sin Android)
```

```
app/src/main/java/cl/esucar/tutorvirtual/
├── di/             AppContainer (inyección manual) + fábrica de ViewModels
├── data/local/     Entidades Room, DAOs y AppDatabase
├── data/prefs/     DataStore (sesión)
├── data/repository/ Auth, Content (siembra desde JSON), Progress
├── domain/         Validators, modelos, fechas, hash de contraseñas
├── notifications/  ReminderScheduler + ReminderWorker (WorkManager)
└── ui/
    ├── theme/      Colores de alto contraste (WCAG AA), tipografía
    ├── components/ Campos con ícono y mensaje de error, chips, barra superior
    ├── navigation/ NavHost, barra inferior (teléfono) / riel lateral (tablet)
    ├── auth/  stages/  faq/  checklist/  settings/   (pantalla + ViewModel)
```

Contenido académico en `app/src/main/assets/content.json` (etapas, pasos, rúbricas, FAQ y fechas).
**Es contenido de ejemplo**: reemplazar por el material oficial entregado por la contraparte ESUCAR.
Las fechas límite son absolutas (ISO `yyyy-MM-dd`); ajústalas al calendario académico vigente.

## Requisitos de la evaluación

| Indicador | Dónde está |
|-----------|-----------|
| Interfaz y navegación | `ui/navigation`, `ui/components`, pantallas por módulo |
| Formularios validados con íconos y mensajes | Registro, Login, Actualizar entregable, Días de recordatorio (`AppTextField`) |
| Validaciones desde la lógica | `domain/Validators.kt` + ViewModels; pruebas en `app/src/test` |
| Animaciones | FAQ desplegable, color del estado, barras de progreso, transición entre pasos, navegación |
| Modularidad y persistencia local | MVVM por paquetes; Room + DataStore |
| GitHub + Trello | Ver `docs/PLAN_COMMITS_Y_TRELLO.md` |
| Dos recursos nativos | **Notificaciones locales** (permiso `POST_NOTIFICATIONS`) y **cámara** (intent del sistema + FileProvider) |

## Cómo ejecutar

1. Abrir la carpeta con **Android Studio** (Ladybug 2024.2 o superior) y esperar el *Gradle Sync*.
   Usar **JDK 17** (Settings → Build Tools → Gradle → Gradle JDK).
2. Crear un emulador (API 26+; recomendado API 34/35 con cámara virtual) o conectar un teléfono.
3. Ejecutar la configuración `app` (▶).
4. Crear una cuenta en la pantalla de registro e ingresar.
5. Pruebas unitarias: `./gradlew test` (o clic derecho en `ValidatorsTest` → Run).

> Si falta `gradlew`, ejecutar `gradle wrapper --gradle-version 8.9` o dejar que Android Studio use la versión de `gradle-wrapper.properties`.

**Probar recordatorios:** en *Ajustes* activa el interruptor de un hito (acepta el permiso). Los avisos se programan
a las 09:00 de cada día de anticipación configurado. Para una demo rápida, edita una fecha en `content.json`
(borra los datos de la app o desinstala para que se vuelva a sembrar).

## Seguridad y privacidad

- Sin datos personales sensibles; todo se guarda localmente (`allowBackup=false`).
- Contraseñas con PBKDF2-HMAC-SHA256 + sal aleatoria; no hay credenciales en el código.
- Permisos solo en el momento de uso; la cámara usa el intent del sistema, por lo que la app no requiere el permiso `CAMERA`.
- Fotos de evidencia en almacenamiento privado de la app, compartidas con la cámara solo vía `FileProvider`.
