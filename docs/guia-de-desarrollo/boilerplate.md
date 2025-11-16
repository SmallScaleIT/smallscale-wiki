---
sidebar_position: 1
---

# Documentación de Boilerplate Astro + Strapi

## 📋 Descripción General

Este boilerplate automatiza la creación de un proyecto completo con **Astro** (frontend) y **Strapi** (CMS headless), proporcionando una estructura lista para desarrollar aplicaciones web modernas con contenido dinámico.

## 🚀 ¿Qué hace este boilerplate?

El script principal `create-project.ts` automatiza:

- **Creación de estructura de carpetas** para frontend y backend
- **Generación de archivos de configuración** (.env) con secretos seguros
- **Copia de templates predefinidos** para funcionalidades comunes
- **Configuración inicial** de variables de entorno

## Estructura de Templates

El boilerplate te permite elegir entre distintas templates o en su defecto por una base:
- Base:
  - [Frontend](https://github.com/SmallScaleIT/web-boilerplate-astro)
  - [Backend](https://github.com/SmallScaleIT/api-boilerplate-strapi)


## 📦 Funcionalidades extra

El boilerplate incluye funcionalides extra listas para usar:

### 1. **Analytics** (analytics)
Integración con Google Analytics
- `GoogleAnalytics.astro` - Componente de seguimiento

### 2. **Blog** (blog)
Sistema de blog funcional
- Interfaces TypeScript (`Article.ts`)
- Páginas predefinidas en `pages/blog/`

### 3. **Cookie Consent** (cookie-consent)
Gestión de consentimiento de cookies
- Componente `CookieConsent.astro`
- Configuración personalizable (`consent-config.ts`)
- Tipos de entorno (`env.d.ts`)

### 4. **Tag Manager** (tag-manager)
Integración con Google Tag Manager
- `GoogleTagManager.astro` - Script principal
- `GoogleTagManagerNoScript.astro` - Fallback sin JavaScript

## 🔧 Utilidades de Script

Las utilidades en utils facilitan la automatización:

| Archivo | Función |
|---------|---------|
| `generate-env.ts` | Genera archivos `.env` con secretos cifrados |
| `copy-dir.ts` | Copia directorios completos |
| `copy-file.ts` | Copia archivos individuales |
| `replace-in-file.ts` | Reemplaza contenido en archivos |
| `write-file.ts` | Escribe nuevos archivos |

## 🔐 Generación de Variables de Entorno

La función `generateStrapiEnv()` en generate-env.ts crea secretos seguros automáticamente:

```typescript
// Genera 4 claves de aplicación (APP_KEYS)
// Tokens JWT para autenticación de admin
// Claves de cifrado y salts para proteger datos
```

Utiliza `crypto.randomBytes()` para generar valores aleatorios seguros basados en base64.

## 📝 Cómo Usar

### 1. **Instalación de Dependencias**

```bash
pnpm install
```

### 2. **Ejecutar el Script de Creación**

```bash
pnpm run create-project
```

Esto ejecutará `create-project.ts` que:
- Solicita información del proyecto
- Genera la estructura de carpetas
- Crea archivos `.env` con secretos cifrados
- Copia los templates seleccionados

### 3. **Configurar Variables de Entorno**

Los archivos `.env` se generan automáticamente con:
- **Frontend** (`.env` web): URL de Strapi y token API
- **Backend** (`.env` Strapi): Puertos, secretos y configuración de base de datos

### 4. **Seleccionar utilidades**

Durante la ejecución, elige qué utilidades incluir:
- ✅ Analytics
- ✅ Blog
- ✅ Cookie Consent
- ✅ Tag Manager

## ⚙️ Configuración de Proyecto

Revisa astro.config.mjs y package.json para:
- Scripts disponibles
- Dependencias instaladas
- Configuración de Astro

## 🎯 Próximos Pasos

1. Navega a las carpetas generadas
2. Instala dependencias en ambos proyectos (si no has seleccionado previamente que se instale automáticamente)
3. Inicia los servidores:
   ```bash
   # Frontend (Astro)
   pnpm dev
   
   # Backend (Strapi)
   cd api-mi-web-strapi && pnpm dev
   ```

## 📚 Recursos Adicionales

- [Documentación de Astro](https://docs.astro.build)
- [Documentación de Strapi](https://docs.strapi.io)
- API Strapi Headless

---

**Nota:** Este boilerplate está diseñado para acelerar el desarrollo inicial. Personaliza según las necesidades de tu proyecto.