Estás utilizando un patrón arquitectónico moderno y robusto basado en **Next.js (App Router)** combinado con una arquitectura enfocada en **Feature-Sliced Design (Diseño basado en Funcionalidades)** o Domain-Driven Design (DDD).

En lugar de tener carpetas globales gigantes para componentes, hooks y servicios, estás agrupando tu código por "dominio" o funcionalidad dentro de la carpeta `features/`.

## 1. La Arquitectura Principal

Estás separando las responsabilidades de tu proyecto en dos piezas fundamentales:

1. **El enrutamiento y la UI de páginas (`app/`)**: Aquí solo defines las rutas públicas y privadas usando convenciones de Next.js (`page.tsx`, `layout.tsx`, `error.tsx`, etc).
2. **La lógica de negocio y módulos (`features/`)**: Aquí reside el "corazón" de tu aplicación. Agrupas todo por dominio (por ejemplo: `auth`, `videos`, `users`, `news`).

Esta separación es excelente porque hace que el código sea altamente escalable y fácil de mantener.

---

## 2. Estructura de Rutas (Directorio `app/`)

Estás usando **Route Groups** (como `(auth)` y `(dashboard)`) para compartir layouts sin afectar la URL final. Aquí tienes el mapa completo de todas las rutas de tu aplicación:

```text
app
├── (auth)                          <-- Grupo de rutas de autenticación
│   ├── auth
│   │   └── sign-in                 <-- URL: /auth/sign-in
│   │       └── page.tsx
│   ├── error.tsx
│   └── layout.tsx                  <-- Layout solo para autenticación
├── (dashboard)                     <-- Grupo de rutas del panel de control
│   ├── dashboard
│   │   ├── citas                   <-- URL: /dashboard/citas
│   │   │   └── page.tsx
│   │   ├── cms                     <-- Sub-sección de Content Management
│   │   │   ├── anuncios            <-- URL: /dashboard/cms/anuncios
│   │   │   │   └── page.tsx
│   │   │   ├── categorias-de-videos<-- URL: /dashboard/cms/categorias-de-videos
│   │   │   │   └── page.tsx
│   │   │   ├── categorias-de-videos-youtube <-- URL: /dashboard/cms/categorias-de-videos-youtube
│   │   │   │   └── page.tsx
│   │   │   ├── eventos             <-- URL: /dashboard/cms/eventos
│   │   │   │   └── page.tsx
│   │   │   ├── mediateca           <-- URL: /dashboard/cms/mediateca
│   │   │   │   └── page.tsx
│   │   │   ├── noticias            <-- URL: /dashboard/cms/noticias
│   │   │   │   └── page.tsx
│   │   │   ├── videos              <-- URL: /dashboard/cms/videos
│   │   │   │   ├── page.tsx
│   │   │   │   └── youtube         <-- URL: /dashboard/cms/videos/youtube
│   │   │   │       └── page.tsx
│   │   │   └── layout.tsx          <-- Layout específico para el CMS
│   │   ├── pagos                   <-- URL: /dashboard/pagos
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   ├── reportes                <-- URL: /dashboard/reportes
│   │   │   └── page.tsx
│   │   ├── servicios-escolares
│   │   │   ├── horarios            <-- URL: /dashboard/servicios-escolares/horarios
│   │   │   │   └── layout.tsx
│   │   │   └── layout.tsx
│   │   ├── unauthorized            <-- URL: /dashboard/unauthorized
│   │   │   └── page.tsx
│   │   ├── users                   <-- URL: /dashboard/users
│   │   │   └── page.tsx
│   │   └── page.tsx                <-- URL: /dashboard (Inicio del panel)
│   ├── error.tsx
│   └── layout.tsx                  <-- Layout global del dashboard (sidebar, navbar, etc)
├── global-error.tsx                <-- Manejador de errores catastróficos globales
├── layout.tsx                      <-- Root Layout (HTML, Body, Providers)
├── not-found.tsx                   <-- Página de error 404 personalizada
└── page.tsx                        <-- URL: / (Landing page o redirección)
```

---

## 3. Estructura de Módulos (Directorio `features/`)

El directorio `features/` contiene todos los dominios de negocio o funcionalidades independientes. Cada carpeta aquí representa un ecosistema cerrado que expone componentes interactivos, servicios de datos, validaciones y tipos.

Los módulos que tienes actualmente son:

```text
features
├── announcements           (Anuncios)
├── auth                    (Autenticación)
├── dates                   (Citas)
├── events                  (Eventos)
├── news                    (Noticias)
├── reports                 (Reportes e Incidencias)
├── users                   (Usuarios y Roles)
├── videos                  (Gestión de Videos)
├── videos-categories       (Categorías de Videos Generales)
├── videos-categories-youtube (Categorías Específicas de YouTube)
└── videos-youtube          (Gestión de Videos de YouTube)
```

### Anatomía de cada Feature (Ejemplo: `reports`)

Tomando como ejemplo el módulo de `reports`, cada característica está subdividida en responsabilidades claras, siguiendo principios de Clean Architecture acoplados a Next.js Server Actions:

```text
features/reports/
├── actions/
│   └── reports-actions.ts      <-- Server Actions (Mutaciones que corren en el servidor: POST, PUT, DELETE)
├── components/
│   ├── edit-report-form.tsx    <-- Formularios de cliente o servidor
│   ├── edit-report-sheet.tsx   <-- Componentes de UI como modales/sheets
│   ├── ...filters.tsx          <-- Componentes de filtros de tablas
│   └── table/                  <-- Componentes dedicados para la tabla de datos
├── mappers/
│   └── report-mappers.ts       <-- Funciones para transformar datos de la BD/API al modelo del frontend
├── services/
│   └── reports-service.ts      <-- Llama a la base de datos o APIs externas (Queries de lectura)
├── types/
│   └── reports.ts              <-- Interfaces y Types de TypeScript
└── validations/
    └── reports-schema.ts       <-- Esquemas de validación (usualmente Zod) para formularios y payloads
```

## Resumen de tu Enfoque

- **Desacoplamiento:** La UI (`app/`) solo consume lo que exporta `features/`. Si mañana decides cambiar tu UI, tu lógica de negocio queda intacta.
- **Server Actions:** Las carpetas `actions/` te permiten hacer mutaciones seguras sin necesidad de crear endpoints tradicionales de API de forma manual.
- **Validaciones fuertemente tipadas:** Al tener `validations/` y `types/` juntos, garantizas que lo que envía el formulario (`components/`) pase por una comprobación estricta antes de llegar a `actions/`.
- **Mappers:** Usas mappers para adaptar las respuestas del backend (Data Transfer Objects - DTOs) a lo que los componentes esperan consumir, aislando al frontend de cambios en la base de datos.
