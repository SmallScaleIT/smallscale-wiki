# Guía de Desarrollo - Strapi CMS

Esta guía está diseñada para desarrolladores que configuren y mantengan la instancia de Strapi que alimenta el frontend de Astro. Aprenderás cómo estructurar tipos de contenido, crear componentes reutilizables y mantener la consistencia con el frontend.

## Tabla de contenidos

1. Conceptos clave
2. Arquitectura de tipos de contenido
3. Single Types (páginas configurables)
4. Collection Types (contenido dinámico)
5. Componentes reutilizables
6. Atributos comunes (SEO, slug, etc.)
7. Relaciones entre tipos
8. Media Manager
9. API REST y tokens
10. Mejores prácticas
11. Checklist para nuevas páginas

---

## Conceptos clave

### Single Type vs Collection Type

**Single Type (Página única):**
- Una sola instancia de contenido
- Ejemplos: HomePage, AboutPage, ContactPage, ServicesPage
- El cliente configura **una sola vez**
- No tienen slug (la URL está definida en Astro)

**Collection Type (Contenido dinámico):**
- Múltiples instancias del mismo tipo
- Ejemplos: Article, Service, Product, Team Member
- El cliente puede crear, editar, eliminar instancias
- Tienen slug para generar rutas dinámicas en Astro

### Flujo de datos: Strapi → Astro

```
Cliente edita contenido en Strapi
           ↓
Strapi almacena en BD
           ↓
Astro fetcha datos via REST API
           ↓
Astro renderiza páginas estáticas (SSR)
           ↓
Usuario ve página en el navegador
```

### Componentes reutilizables

Los componentes en Strapi son **bloques de contenido reutilizables** que se pueden usar en múltiples Single Types y Collection Types. Reduce duplicación y mantiene consistencia.

---

## Arquitectura de tipos de contenido

### Estructura recomendada

```
Strapi (Content Manager)
├── Single Types
│   ├── HomePage
│   ├── AboutPage
│   ├── ServicesPage
│   ├── ContactPage
│   └── GlobalConfig (para datos globales: logo, redes sociales, etc.)
│
├── Collection Types
│   ├── Service (detalle de cada servicio)
│   ├── Article (blog posts)
│   ├── Product (catálogo)
│   ├── TeamMember (equipo)
│   └── Testimonial (testimonios)
│
└── Components (carpeta conceptual, no literal)
    ├── Sections
    │   ├── HeroSection
    │   ├── FeaturesSection
    │   ├── CtaSection
    │   └── GallerySection
    ├── Elements
    │   ├── Button
    │   ├── Card
    │   └── Testimonial
    └── Metadata
        ├── SeoMeta
        └── Author
```

---

## Single Types (Páginas configurables)

### Ejemplo: ServicesPage

Una página que muestra un listado de servicios y permite al cliente configurar el hero, descripción, etc.

#### Paso 1: Crear el Single Type

1. Ve a **Content-Type Builder** → **Single Types**
2. Haz clic en **Create new single type**
3. Nombre: `ServicesPage`

#### Paso 2: Añadir atributos

**Atributos base (siempre en páginas):**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `meta_title` | Text | ✓ | Título SEO (50-60 chars) |
| `meta_description` | Text (long) | ✓ | Descripción SEO (150-160 chars) |

**Atributos específicos de ServicesPage:**

| Campo | Tipo | Requerido |
|-------|------|-----------|
| `hero_section` | Component (HeroSection) | ✓ |
| `intro_section` | Component (RichTextSection) | ✗ |
| `services_intro_text` | Text (long) | ✗ |

````
Atributos de ServicesPage:

meta_title
├─ Type: String
├─ Required: true
└─ Max length: 60

meta_description
├─ Type: Text
├─ Required: true
└─ Max length: 160

hero_section
├─ Type: Component
├─ Component: HeroSection
└─ Required: true

services_intro_text
├─ Type: Text (long)
├─ Required: false
└─ Markdown: enabled

order
├─ Type: Integer
├─ Default: 0
└─ Help: Para ordenar servicios destacados
````

#### Paso 3: Configurar permisos

1. Ve a **Settings** → **Roles**
2. Selecciona el rol del cliente (ej: "Editor")
3. Asegúrate de que puede:
   - ✓ Read ServicesPage
   - ✓ Update ServicesPage
   - ✗ Create/Delete (no queremos duplicados)

#### Contenido en ServicesPage

El cliente va a **Content Manager** → **ServicesPage** y rellena:

```json
{
  "meta_title": "Nuestros Servicios - TuEmpresa",
  "meta_description": "Ofrecemos servicios de consultoría, desarrollo web y más",
  "hero_section": {
    "title": "Nuestros Servicios",
    "subtitle": "Soluciones integrales para tu negocio",
    "image": { "id": 1, "name": "hero-services.png" }
  },
  "services_intro_text": "Trabajamos con empresas de todas las industrias..."
}
```

### Ejemplo: HomePage

```
HomePage
├─ meta_title (String)
├─ meta_description (String)
├─ hero_section (Component: HeroSection)
├─ featured_services (Component: ServicesGrid)
├─ testimonials_section (Component: TestimonialsSection)
├─ cta_section (Component: CtaSection)
└─ published_at (DateTime)
```

---

## Collection Types (Contenido dinámico)

### Ejemplo: Service

Una colección de servicios individuales. Cada servicio puede ser una página separada en Astro (ej: `/servicios/desarrollo-web`).

#### Paso 1: Crear el Collection Type

1. Ve a **Content-Type Builder** → **Collection Types**
2. Haz clic en **Create new collection type**
3. Nombre: `Service`

#### Paso 2: Atributos

**Atributos base:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `title` | String | ✓ | Nombre del servicio |
| `slug` | String | ✓ | URL-friendly (auto-generado desde title) |
| `meta_title` | String | ✓ | SEO title |
| `meta_description` | String | ✓ | SEO description |

**Atributos específicos:**

| Campo | Tipo | Requerido |
|-------|------|-----------|
| `description` | Text (long) | ✓ |
| `featured_image` | Media (single) | ✓ |
| `content_sections` | Component (repeatable) | ✗ |
| `benefits` | Component (repeatable: BenefitCard) | ✓ |
| `cta_section` | Component (CtaSection) | ✗ |
| `order` | Integer | ✗ |

````
Atributos de Service:

title
├─ Type: String
├─ Required: true
└─ Max length: 100

slug
├─ Type: String (UID)
├─ Required: true
├─ Attached field: title (auto-generate)
└─ Regex: ^[a-z0-9]+(?:-[a-z0-9]+)*$

meta_title
├─ Type: String
├─ Required: true
└─ Max length: 60

meta_description
├─ Type: String
├─ Required: true
└─ Max length: 160

description
├─ Type: Text (long)
├─ Required: true
└─ Markdown: enabled

featured_image
├─ Type: Media
├─ Required: true
└─ Single file: true

benefits
├─ Type: Component (repeatable)
├─ Component: BenefitCard
└─ Required: true

order
├─ Type: Integer
├─ Default: 0
└─ Help: Para ordenar servicios en la página principal
````

#### Paso 3: Contenido

El cliente crea múltiples servicios en **Content Manager** → **Service**:

```json
[
  {
    "id": 1,
    "title": "Desarrollo Web",
    "slug": "desarrollo-web",
    "meta_title": "Desarrollo Web Personalizado",
    "meta_description": "Webs rápidas, modernas y SEO-friendly",
    "description": "Creamos sitios web modernos...",
    "featured_image": { "id": 5 },
    "benefits": [
      {
        "id": 1,
        "title": "100% Responsive",
        "description": "Funciona en todos los dispositivos"
      }
    ],
    "order": 1
  },
  {
    "id": 2,
    "title": "Consultoría",
    "slug": "consultoria",
    ...
  }
]
```

---

## Componentes reutilizables

Los **Components** son bloques que se pueden usar en múltiples Single Types y Collection Types.

### Paso 1: Crear un componente

Ve a **Content-Type Builder** → **Components** → **Create new component**.

### Componentes recomendados

#### 1. HeroSection

Se usa en: HomePage, AboutPage, páginas de detalle

```
HeroSection
├─ title (String, required)
├─ subtitle (String)
├─ description (Text)
├─ image (Media, required)
├─ button_text (String)
├─ button_link (String, URL o slug interno)
└─ background_color (Enumeration: light/dark)
```

**Contenido de ejemplo:**

```json
{
  "title": "Bienvenido a nuestros servicios",
  "subtitle": "Soluciones innovadoras para tu negocio",
  "image": { "id": 1 },
  "button_text": "Explorar más",
  "button_link": "/servicios",
  "background_color": "light"
}
```

#### 2. RichTextSection

Se usa en: páginas informativas, blogs

```
RichTextSection
├─ title (String)
├─ content (Text, long, markdown)
└─ alignment (Enumeration: left/center/right)
```

#### 3. FeatureCard (Repeatable)

Se usa en: FeaturesSection, beneficios

```
FeatureCard
├─ title (String, required)
├─ description (Text)
├─ icon (String o Media)
└─ order (Integer)
```

#### 4. TestimonialCard (Repeatable)

Se usa en: testimonios, reseñas

```
TestimonialCard
├─ author_name (String, required)
├─ author_position (String)
├─ author_image (Media)
├─ content (Text, required)
└─ rating (Integer, 1-5)
```

#### 5. BenefitCard (Repeatable)

Se usa en: Service, Product

```
BenefitCard
├─ title (String, required)
├─ description (Text)
├─ icon (String)
└─ order (Integer)
```

#### 6. CtaSection

Se usa en: llamadas a acción, finales de páginas

```
CtaSection
├─ title (String, required)
├─ subtitle (String)
├─ button_text (String, required)
├─ button_link (String, required)
└─ background_image (Media)
```

#### 7. GallerySection

Se usa en: portfolios, galerías de proyectos

```
GallerySection
├─ title (String)
├─ images (Media, multiple)
└─ columns (Integer, default: 3)
```

---

## Atributos comunes (SEO, slug, etc.)

### SEO Meta (reutilizable)

En lugar de repetir `meta_title` y `meta_description` en cada tipo, puedes crear un componente:

```
SeoMeta
├─ meta_title (String, required)
└─ meta_description (String, required)
```

Luego usarlo en todos los Single/Collection Types:

```
Service
├─ seo (Component: SeoMeta, required)
├─ title (String)
├─ slug (UID)
└─ ...
```

### Slug (URL dinámica)

**Configuración de UID en Strapi:**

- **Field name:** `slug`
- **Attached field:** `title` (auto-genera slug desde el título)
- **Regex:** `^[a-z0-9]+(?:-[a-z0-9]+)*$` (valida formato)

**Resultado:**

```
Cliente escribe: "Mi Nuevo Servicio"
         ↓
Slug generado: "mi-nuevo-servicio"
         ↓
URL en Astro: /servicios/mi-nuevo-servicio
```

### Timestamps

Strapi añade automáticamente:

- `createdAt` (DateTime)
- `updatedAt` (DateTime)
- `publishedAt` (DateTime)

Úsalos en Astro para:

```ts
const articles = await fetchApi<Article[]>({
  endpoint: 'articles?sort=publishedAt:desc',
  wrappedByList: 'data',
});
```

### Orden y estado

Añade a Collection Types si necesitas ordenamiento manual:

```
Service
├─ order (Integer, default: 0)
└─ featured (Boolean, default: false)
```

Luego en Astro:

```ts
const featuredServices = await fetchApi<Service[]>({
  endpoint: 'services?filters[featured][$eq]=true&sort=order:asc',
  wrappedByList: 'data',
});
```

---

## Relaciones entre tipos

### Relación: Service tiene muchos Articles

Si quieres mostrar artículos relacionados con un servicio:

**En el Collection Type Article:**

Añade un atributo:

```
Article
├─ title (String)
├─ slug (UID)
├─ service (Relation: one-to-many with Service)
└─ ...
```

**En Astro (detalle de Service):**

```astro
---
const articles = await fetchApi<Article[]>({
  endpoint: `articles?filters[service][$eq]=${serviceId}&populate=*`,
  wrappedByList: 'data',
});
---

<h2>Artículos relacionados</h2>
{articles.map(article => (
  <a href={`/blog/${article.slug}`}>{article.title}</a>
))}
```

### Relación: Testimonial pertenece a Service

```
Testimonial
├─ author_name (String)
├─ content (Text)
├─ service (Relation: many-to-one with Service)
└─ ...
```

**En Astro:**

```astro
---
const testimonials = await fetchApi<Testimonial[]>({
  endpoint: `testimonials?filters[service][$eq]=${serviceId}&populate=*`,
  wrappedByList: 'data',
});
---
```

---

## Media Manager

### Organización de archivos

Crea carpetas en **Media Library** para mantener orden:

```
uploads/
├─ homepage/
│  ├─ hero-bg.webp
│  └─ featured-img.webp
├─ services/
│  ├─ desarrollo-web.webp
│  └─ consultoria.webp
├─ blog/
│  └─ article-1.webp
├─ team/
│  └─ member-1.webp
└─ global/
   ├─ logo.svg
   └─ favicon.ico
```

### Optimización de imágenes

- **Formato:** WebP o AVIF (más pequeño que PNG/JPG)
- **Tamaño:** máximo 200KB por imagen
- **Nombres:** kebab-case sin espacios
- **Alt text:** siempre añadirlo en Strapi

**Ejemplo de media en Strapi:**

```json
{
  "id": 1,
  "name": "hero-services.webp",
  "alternativeText": "Equipo en una reunión de trabajo",
  "caption": "Nuestro equipo",
  "width": 1920,
  "height": 1080,
  "size": 180,
  "formats": {
    "large": { "url": "/uploads/large_hero_services.webp" },
    "medium": { "url": "/uploads/medium_hero_services.webp" }
  },
  "url": "/uploads/hero_services.webp"
}
```

### Referencia en componentes

```
HeroSection
├─ title (String)
└─ image (Media)
   └─ url es automáticamente "/uploads/..."
      (Astro lo convierte a URL completa con getImageUrl())
```

---

## API REST y tokens

### ¿Qué es un API Token?

Un **token** es una clave que autoriza acceso a la API de Strapi. Permite a Astro fetchar contenido publicado.

### Usar el Token por defecto (opción recomendada)

Cuando creas un proyecto Strapi por primera vez, **Strapi genera automáticamente un token por defecto**. Este token es perfecto para usar en Astro:

#### Paso 1: Encontrar el token por defecto

1. Ve a **Settings** → **API Tokens**
2. Verás un token llamado algo como `DefaultToken` o similar (creado automáticamente)
3. Haz clic en **Copy** para copiar el token
4. ⚠️ **Guárdalo en un lugar seguro** - no puedes verlo de nuevo

#### Paso 2: Configurar en Astro

En el `.env` del proyecto Astro:

```env
PUBLIC_STRAPI_URL=https://strapi.example.com
STRAPI_API_TOKEN=your_default_token_here_xyz123abc
```

**Ventaja:** No necesitas crear un nuevo token ni configurar permisos.

---

### Crear un API Token personalizado (opción alternativa)

Si prefieres crear un token nuevo con permisos específicos, sigue estos pasos:

#### Paso 1: Crear el token

1. Ve a **Settings** → **API Tokens**
2. Haz clic en **Create new API Token**
3. Rellena:
   - **Name:** `Astro Frontend`
   - **Description:** `Token para acceso de lectura desde Astro`
   - **Type:** `Read-only` (el cliente no necesita escribir)
   - **Duration:** `Unlimited` (o personalizado)

#### Paso 2: Configurar permisos

En **Permissions**, asegúrate de marcar:

- ✓ All locales (si usas localización multiidioma)
- ✓ api::service.service (lectura)
- ✓ api::article.article (lectura)
- ✓ api::product.product (lectura)
- etc. (todos los tipos que Astro necesita)

⚠️ **No marques permisos de escritura** si es read-only.

#### Paso 3: Copiar el token

Haz clic en **Copy** y guárdalo. **No puedes verlo de nuevo**, así que guárdalo en un lugar seguro.

#### Paso 4: Configurar en Astro

En el `.env` del proyecto Astro:

```env
PUBLIC_STRAPI_URL=https://strapi.example.com
STRAPI_API_TOKEN=your_custom_token_here_xyz123abc
```

**Nota:** `PUBLIC_STRAPI_URL` es pública (sin `PUBLIC_` prefix sería privada). `STRAPI_API_TOKEN` es privado y solo se usa en el servidor durante el build.

---

### Testear el token

Independientemente de si usas el token por defecto o uno personalizado, en Strapi ve a **Plugins** → **REST API** (si está disponible) o usa curl:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://strapi.example.com/api/services?populate=*
```

Debe devolver:

```json
{
  "data": [
    {
      "id": 1,
      "attributes": {
        "title": "Desarrollo Web",
        "slug": "desarrollo-web",
        ...
      }
    }
  ]
}
```

---

## Mejores prácticas

### 1. Naming consistency (convenciones de nombres)

| Tipo | Convención | Ejemplo |
|------|-----------|---------|
| Single Type | `PascalCase` | `HomePage`, `ServicesPage`, `ContactPage` |
| Collection Type | `PascalCase` (singular) | `Service`, `Article`, `Product` |
| Component | `PascalCase` | `HeroSection`, `FeatureCard` |
| Atributo | `snake_case` | `meta_title`, `featured_image`, `button_text` |
| Media folder | `kebab-case` | `homepage/`, `blog-posts/` |

### 2. Reutiliza componentes

✅ **Correcto:**

```
HeroSection (component)
└─ usado en: HomePage, ServicesPage, AboutPage, Service (detail)
```

❌ **Incorrecto:**

```
HomeHeroSection, ServicesHeroSection, AboutHeroSection
(repetido 3 veces)
```

### 3. Atributos requeridos vs opcionales

- **Requerido:** meta_title, meta_description, slug, title
- **Opcional:** subtitle, button_text, background_image (si tiene default)

Minimiza campos requeridos para no abrumar al cliente.

### 4. Documentación interna

En la **Description** de cada atributo, añade:

```
HeroSection → title (description)
├─ "Título principal del hero (máx 100 caracteres)"

Service → meta_description (description)
├─ "Descripción para buscadores (150-160 caracteres, aparece bajo el título)"

Service → order (description)
├─ "Orden de aparición en la página principal (menor = primero)"
```

### 5. Localización (i18n)

Si necesitas múltiples idiomas:

1. Ve a **Settings** → **Internationalization**
2. Haz clic en **Create new locale**
3. Añade: Español, English, Catalán, etc.

Strapi duplicará todos los atributos:

```json
{
  "es": { "title": "Desarrollo Web" },
  "en": { "title": "Web Development" },
  "ca": { "title": "Desenvolupament Web" }
}
```

En Astro, fetcha por locale:

```ts
const articles = await fetchApi<Article[]>({
  endpoint: 'articles?locale=es&populate=*',
  wrappedByList: 'data',
});
```

### 6. Roles y permisos

Crea dos roles:

**Editor (cliente):**
- ✓ Read/Update (Single Types)
- ✓ Create/Read/Update/Delete (Collection Types)
- ✗ Delete (páginas configurables)
- ✗ Settings/Plugins

**Admin (desarrollador):**
- ✓ Todo

### 7. Valida contenido

Usa **Regex** en campos para evitar errores:

```
slug
├─ Regex: ^[a-z0-9]+(?:-[a-z0-9]+)*$
└─ Error message: "Solo letras minúsculas, números y guiones"

order
├─ Min: 0
└─ Max: 1000
```

### 8. Versionado y backups

- **Backups:** exporta regularmente datos en **Settings** → **Backup**
- **Versionado:** Strapi guarda `createdAt` y `updatedAt` automáticamente
- **Publicación:** usa `publishedAt` para controlar visibilidad

### 9. Campos Markdown

Algunos campos soportan Markdown (texto enriquecido):

```
Article → description (Text, long)
├─ Markdown: enabled
└─ Permite: **bold**, _italic_, [links](), etc.
```

Desde Astro, renderiza con:

```astro
---
import { marked } from 'marked';

const html = await marked(article.description);
---

<div set:html={html}></div>
```

O usa un componente React:

```tsx
import { marked } from 'marked';

export default function RichText({ content }) {
  const html = marked(content);
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

### 10. Draft/Published state

Siempre usa **Publish/Draft** en Strapi:

```
Service
├─ (borrador/draft)
└─ → Haz clic en "Publish" para hacerlo visible a Astro
```

En Astro, fetcha solo publicados (by default lo hace, pero confirma):

```ts
const articles = await fetchApi<Article[]>({
  endpoint: 'articles?populate=*', // Por defecto, solo publicados
  wrappedByList: 'data',
});
```

---

## Ejemplo completo: ServicesPage + Service

### Paso 1: Crear componentes

**HeroSection:**

```
title (String, required)
subtitle (String)
image (Media, required)
button_text (String)
button_link (String)
```

**BenefitCard (repeatable):**

```
title (String, required)
description (Text)
icon (String)
```

**CtaSection:**

```
title (String, required)
button_text (String, required)
button_link (String, required)
```

### Paso 2: Crear Single Type ServicesPage

```
meta_title (String, required)
meta_description (String, required)
hero_section (Component: HeroSection, required)
intro_text (Text, long)
cta_section (Component: CtaSection)
```

**Contenido:**

```json
{
  "meta_title": "Nuestros Servicios",
  "meta_description": "Ofrecemos consultoría, desarrollo web y más",
  "hero_section": {
    "title": "Servicios que transforman negocios",
    "subtitle": "Soluciones integrales a medida",
    "image": { "id": 1 },
    "button_text": "Ver detalle",
    "button_link": "#servicios"
  },
  "intro_text": "Contamos con un equipo experto...",
  "cta_section": { ... }
}
```

### Paso 3: Crear Collection Type Service

```
title (String, required)
slug (UID, required, attached: title)
meta_title (String, required)
meta_description (String, required)
description (Text, long, markdown)
featured_image (Media, required)
benefits (Component repeatable: BenefitCard, required)
cta_section (Component: CtaSection)
order (Integer, default: 0)
featured (Boolean, default: false)
```

**Contenido (ejemplo 1):**

```json
{
  "title": "Desarrollo Web",
  "slug": "desarrollo-web",
  "meta_title": "Desarrollo Web Personalizado | TuEmpresa",
  "meta_description": "Webs rápidas, modernas y optimizadas para SEO",
  "description": "Creamos sitios web...",
  "featured_image": { "id": 5 },
  "benefits": [
    {
      "title": "100% Responsive",
      "description": "Funciona perfectamente en móvil, tablet y desktop",
      "icon": "mobile"
    },
    {
      "title": "SEO Optimizado",
      "description": "Posicionamiento garantizado en buscadores",
      "icon": "search"
    }
  ],
  "cta_section": {
    "title": "¿Listo para tu nueva web?",
    "button_text": "Solicitar presupuesto",
    "button_link": "/contacto"
  },
  "order": 1,
  "featured": true,
  "publishedAt": "2024-01-15T10:00:00.000Z"
}
```

### Paso 4: Fetchear en Astro

**Página: /servicios (ServicesPage)**

```astro
---
import Layout from '@/layouts/Layout.astro';
import Section from '@/components/Section.astro';
import ServiceCard from '@/components/ServiceCard.astro';
import fetchApi from '@/lib/strapi';

// Fetch la página configurada
const servicesPage = await fetchApi({
  endpoint: 'services-page?populate=*',
  wrappedByKey: 'data',
});

// Fetch todos los servicios, ordenados
const { data: services } = await fetchApi({
  endpoint: 'services?sort=order:asc&populate=featured_image,benefits',
  wrappedByList: 'data',
});
---

<Layout
  title={servicesPage.meta_title}
  description={servicesPage.meta_description}
>
  <!-- Hero -->
  <Hero
    title={servicesPage.hero_section.title}
    subtitle={servicesPage.hero_section.subtitle}
    image={servicesPage.hero_section.image}
  />

  <!-- Intro -->
  <Section>
    <p class="text-lg text-gray-600 max-w-2xl mx-auto">
      {servicesPage.intro_text}
    </p>
  </Section>

  <!-- Servicios -->
  <Section>
    <h2 class="text-4xl font-bold mb-12">Nuestros Servicios</h2>
    <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
      {services.map(service => (
        <ServiceCard service={service} />
      ))}
    </div>
  </Section>

  <!-- CTA -->
  <CtaSection cta={servicesPage.cta_section} />
</Layout>
```

**Página: /servicios/[slug] (detalle de Service)**

```astro
---
import Layout from '@/layouts/Layout.astro';
import Section from '@/components/Section.astro';
import { Image } from 'astro:assets';
import { getImageUrl } from '@/lib/strapi';
import fetchApi from '@/lib/strapi';
import type Service from '@/interfaces/Service';

export async function getStaticPaths() {
  const { data: services } = await fetchApi({
    endpoint: 'services?populate=*',
    wrappedByList: 'data',
  });

  return services.map(service => ({
    params: { slug: service.slug },
    props: service,
  }));
}

type Props = Service;
const service = Astro.props;
---

<Layout
  title={service.meta_title}
  description={service.meta_description}
>
  <!-- Hero del servicio -->
  <div class="relative h-96 overflow-hidden">
    <Image
      src={getImageUrl(service.featured_image.url)}
      alt={service.title}
      width={1920}
      height={1080}
      class="w-full h-full object-cover"
    />
    <div class="absolute inset-0 bg-black/40 flex items-center justify-center">
      <h1 class="text-5xl font-bold text-white text-center">
        {service.title}
      </h1>
    </div>
  </div>

  <!-- Descripción -->
  <Section>
    <div class="prose prose-lg max-w-3xl mx-auto">
      <div set:html={service.description}></div>
    </div>
  </Section>

  <!-- Beneficios -->
  <Section>
    <h2 class="text-3xl font-bold mb-12">Beneficios</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
      {service.benefits.map(benefit => (
        <div class="flex gap-4">
          <div class="text-3xl">{benefit.icon}</div>
          <div>
            <h3 class="text-xl font-semibold mb-2">{benefit.title}</h3>
            <p class="text-gray-600">{benefit.description}</p>
          </div>
        </div>
      ))}
    </div>
  </Section>

  <!-- CTA -->
  {service.cta_section && (
    <CtaSection cta={service.cta_section} />
  )}
</Layout>
```

---

## Checklist para crear una nueva página (Single Type)

- [ ] Crear el **Single Type** en Content-Type Builder
- [ ] Añadir campos: `meta_title`, `meta_description`
- [ ] Añadir componentes específicos de la página (HeroSection, etc.)
- [ ] Rellenar contenido en Content Manager
- [ ] Hacer **Publish**
- [ ] Testear que aparezca en Astro: `http://localhost:4321/pagina`
- [ ] Verificar metadatos en `<head>` (DevTools → Elements)

## Checklist para crear una nueva colección (Collection Type)

- [ ] Crear el **Collection Type** en Content-Type Builder
- [ ] Campos requeridos: `title`, `slug`, `meta_title`, `meta_description`
- [ ] Crear al menos 1 entrada en Content Manager
- [ ] Hacer **Publish**
- [ ] Crear página dinámica en Astro: `[slug].astro`
- [ ] Testear rutas: `http://localhost:4321/coleccion/item-slug`
- [ ] Verificar componentes Astro renderan datos

---

## Recursos

- [Documentación oficial de Strapi](https://docs.strapi.io)
- [REST API de Strapi](https://docs.strapi.io/dev-docs/api/rest)
- [Content-Type Builder](https://docs.strapi.io/user-docs/content-manager/content-types-builder)
- [API Tokens](https://docs.strapi.io/user-docs/settings/API-tokens)
- [Markdown en Strapi](https://docs.strapi.io/user-docs/content-manager/working-with-content#text-field)

---

**¿Dudas?** Consulta las guías de Astro o comunícate con el equipo de desarrollo.