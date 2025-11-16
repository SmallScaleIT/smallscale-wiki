---
sidebar_position: 2
---

# Programación en Astro + Tailwind

Esta guía está diseñada para desarrolladores que trabajen con esta boilerplate de Astro + React + Tailwind integrada con Strapi. Aprenderás cómo crear páginas, componentes y mantener la consistencia del proyecto.

## Tabla de contenidos

1. Conceptos clave
2. Estructura de carpetas y convenciones
3. Crear una nueva página
4. Rutas dinámicas con `getStaticPaths`
5. Paginación con `paginate`
6. Trabajar con componentes Astro
7. Componentes React (UI y helpers)
8. Fetching de datos desde Strapi
9. Estilos con Tailwind CSS
10. SEO y metadatos
11. Mejores prácticas

---

## Conceptos clave

### Server-Side Rendering (SSR) con Node

Este proyecto usa **SSR con el adaptador `@astrojs/node`** en modo `standalone`. Esto significa:

- Las páginas se renderizan en el servidor en tiempo de ejecución.
- Puedes usar `await` en componentes Astro (son síncronos en el servidor).
- Los datos se fetchen **en el servidor**, no en el cliente.
- El bundle de JavaScript del cliente es más pequeño (no necesitas incluir lógica de fetch en el cliente).

### Flujo de datos

```
Página Astro → Layout.astro (fetches seoConfig) → Header + Slot + Footer
                    ↓
              Componentes Astro (pueden fetchar datos)
                    ↓
              Componentes React (hidratados solo si es necesario)
                    ↓
              Strapi API
```

---

## Estructura de carpetas y convenciones

```
src/
├── components/           # Componentes Astro (lógica SSR, meta, markup)
│   ├── ui/              # Componentes React atómicos (Radix UI + Tailwind)
│   └── *.astro          # Componentes Astro de página/sección
├── helpers/             # Componentes React complejos con estado
│   └── *.tsx            # Carruseles, modales, formularios interactivos
├── interfaces/          # Tipos TypeScript
│   └── *.ts             # SeoConfig.ts, Image.ts, Faq.ts, etc.
├── layouts/             # Layouts globales
│   └── Layout.astro     # Wrapper central (meta, header, footer)
├── lib/                 # Utilidades y servicios
│   ├── strapi.ts        # Wrapper de API Strapi
│   ├── seo-config.ts    # Configuración SEO por defecto
│   └── utils.ts         # Helpers reutilizables
├── pages/               # Rutas (mapeo 1:1 con URLs)
│   └── index.astro      # / → index.astro
├── styles/
│   └── global.css       # Entry de Tailwind + estilos globales
└── public/              # Assets estáticos (imágenes, favicons)
    └── fav/
    └── fonts/
```

### Convenciones de nombres

| Tipo | Ubicación | Convención | Ejemplo |
|------|-----------|-----------|---------|
| Página | pages | `kebab-case.astro` | `sobre-nosotros.astro` |
| Componente Astro | components | `PascalCase.astro` | `Header.astro` |
| Componente React (UI) | ui | `kebab-case.tsx` | `button.tsx` |
| Componente React (Helper) | helpers | `PascalCase.tsx` | `StrapiRichText.tsx` |
| Tipo/Interface | interfaces | `PascalCase.ts` | `SeoConfig.ts` |
| Utilidad | lib | `kebab-case.ts` | `strapi.ts` |

---

## Crear una nueva página

### 1. Crear el archivo de página

Crea `src/pages/productos.astro`:

````astro
---
import Layout from '@/layouts/Layout.astro';
import ProductCard from '@/components/ProductCard.astro';
import Section from '@/components/Section.astro';
import fetchApi from '@/lib/strapi';

interface Product {
  id: number;
  title: string;
  description: string;
  image: { url: string };
}

const productos = await fetchApi<Product[]>({
  endpoint: 'productos?populate=image',
  wrappedByList: 'data',
});
---

<Layout
  title="Productos"
  description="Explora nuestro catálogo de productos"
  index={true}
  follow={true}
>
  <Section>
    <h1 class="text-4xl font-bold mb-8">Nuestros Productos</h1>
    
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {productos.map((product) => (
        <ProductCard product={product} />
      ))}
    </div>
  </Section>
</Layout>
````

### 2. Props del Layout

El componente Layout.astro acepta estas props:

```ts
interface Props {
  title: string;                    // Título de la página (requerido)
  description?: string;             // Meta description
  preload?: Array<Preload>;         // Resources a preloadear
  canonical?: string;               // URL canónica (por defecto: pathname)
  prev?: string;                    // Link rel=prev (paginación)
  next?: string;                    // Link rel=next (paginación)
  faq?: Faq[];                      // FAQ para rich results
  image?: string;                   // OG image (por defecto: seoConfig.image)
  index?: boolean;                  // Meta robots: index (default: true)
  follow?: boolean;                 // Meta robots: follow (default: true)
}
```

---

## Rutas dinámicas con `getStaticPaths`

### ¿Qué es `getStaticPaths`?

`getStaticPaths()` es una función especial que **le dice a Astro qué rutas dinámicas deben generarse** durante el build. Sin ella, Astro no sabría qué páginas crear.

### Ruta dinámica simple: `[slug].astro`

Crea `src/pages/blog/[slug].astro`:

````astro
---
import Layout from '@/layouts/Layout.astro';
import Section from '@/components/Section.astro';
import fetchApi, { getImageUrl } from '@/lib/strapi';
import { Image } from 'astro:assets';
import type Article from '@/interfaces/Article';

export async function getStaticPaths() {
  // Fetch todas las artículos de Strapi
  const articles = await fetchApi<Article[]>({
    endpoint: 'articles?populate=image',
    wrappedByList: 'data',
  });

  // Devuelve un array de rutas: una por cada artículo
  return articles.map((article) => ({
    params: { slug: article.slug },  // Define el parámetro de la ruta
    props: article,                  // Pasa el artículo como prop
  }));
}

type Props = Article;
const article = Astro.props;
---

<Layout 
  title={article.title} 
  description={article.description}
>
  <Section>
    <h1 class="text-4xl font-bold mb-8">{article.title}</h1>
    <Image 
      src={getImageUrl(article.image.url)} 
      alt={article.title} 
      width={800}
      height={400}
    />
    <div class="prose mt-8" set:html={article.content} />
  </Section>
</Layout>
````

**Resultado:**
- 10 artículos en Strapi → 10 archivos HTML en `dist/blog/`:
  - `dist/blog/mi-primer-articulo/index.html`
  - `dist/blog/segundo-articulo/index.html`
  - etc.

### Datos adicionales en `getStaticPaths`

A veces necesitas más datos que solo el objeto principal:

````astro
---
import type Article from '@/interfaces/Article';
import type Category from '@/interfaces/Category';

export async function getStaticPaths() {
  const articles = await fetchApi<Article[]>({
    endpoint: 'articles?populate=category',
    wrappedByList: 'data',
  });

  // Fetch adicionales para cada página
  return articles.map((article) => ({
    params: { slug: article.slug },
    props: {
      article,
      relatedArticles: [], // Puedes añadir más datos aquí
    },
  }));
}

interface Props {
  article: Article;
  relatedArticles: Article[];
}

const { article, relatedArticles } = Astro.props;
---
````

---

## Paginación con `paginate`

### ¿Qué es `paginate`?

`paginate()` es un helper de Astro que genera **múltiples páginas de un array** automáticamente. Ideal para blogs, listados de productos, etc.

### Ruta de paginación: `[...page].astro`

La sintaxis `[...page]` es un **catch-all route** (ruta comodín). Captura cualquier número de segmentos en la URL.

Crea `src/pages/blog/[...page].astro`:

````astro
---
import Layout from '@/layouts/Layout.astro';
import Pagination from '@/components/Pagination.astro';
import ArticleCard from '@/components/ArticleCard.astro';
import fetchApi from '@/lib/strapi';
import type Article from '@/interfaces/Article';
import type Meta from '@/interfaces/Meta';
import type { GetStaticPathsOptions } from 'astro';

// GetStaticPathsOptions incluye la función paginate
export async function getStaticPaths({ paginate }: GetStaticPathsOptions) {
  // Fetch todos los artículos
  const { data: articles } = await fetchApi<{ 
    data: Article[]; 
    meta: Meta;
  }>({
    endpoint: 'articles?populate=image&sort=publishedAt:desc',
  });

  // paginate() genera un array de rutas automáticamente
  // pageSize: 6 = 6 artículos por página
  return paginate(articles, { pageSize: 6 });
}

// Astro inyecta automáticamente la prop "page"
const { page } = Astro.props;
const articles = page.data;
---

<Layout 
  title="Blog" 
  description="Últimos artículos"
  prev={page.currentPage === 2 ? undefined : page.url.prev} 
  next={page.url.next}
>
  <section class="container mx-auto px-4 py-12">
    <h1 class="text-4xl font-bold mb-8">Blog</h1>
    
    {articles.length === 0 ? (
      <p class="text-gray-500">No hay artículos disponibles.</p>
    ) : (
      <>
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 mb-8">
          {articles.map((article) => (
            <ArticleCard article={article} />
          ))}
        </div>
        <Pagination page={page} />
      </>
    )}
  </section>
</Layout>
````

### ¿Qué genera `paginate()`?

Si tienes 20 artículos y `pageSize: 6`:

```
paginate(articles, { pageSize: 6 })
↓
Genera 4 archivos HTML:
  dist/blog/index.html         (página 1, artículos 1-6)
  dist/blog/2/index.html       (página 2, artículos 7-12)
  dist/blog/3/index.html       (página 3, artículos 13-18)
  dist/blog/4/index.html       (página 4, artículos 19-20)
```

### Props de la página paginada

El objeto `page` contiene:

```ts
page = {
  data: Article[],           // Array de items para esta página
  start: 0,                  // Índice del primer item
  end: 6,                    // Índice del último item
  total: 20,                 // Total de items
  size: 6,                   // Items por página
  currentPage: 1,            // Número de página actual (1-indexed)
  lastPage: 4,               // Número de última página
  url: {
    current: '/blog/',
    prev: '/blog/',          // undefined si es página 1
    next: '/blog/2/',
  },
}
```

### Componente `Pagination`

Un componente útil para renderizar botones de anterior/siguiente:

````astro
---
import type { Page } from "astro";
import { ChevronLeft, ChevronRight } from "@lucide/astro";

interface Props {
  page: Page;
}

const { page } = Astro.props;

function getPageUrl(pageNumber: number) {
  const params = new URLSearchParams(Astro.url.searchParams);
  let path = '';

  if (page.currentPage === 1) {
    if (pageNumber === 1) {
      path = `${page.url.current}`;
    } else {
      path = `${page.url.current}/${pageNumber}`;
    }
  } else {
    if (pageNumber === 1) {
      path = `${page.url.current.replace(`/${page.currentPage}`, '')}`;
    } else {
      path = page.url.current.replace(`/${page.currentPage}`, `/${pageNumber}`);
    }
  }
  
  if (params.size === 0) {
    return path;
  }
  
  return `${path}?${params.toString()}`;
}
---

<nav class="flex justify-center items-center gap-2">
  {
    page.url.prev && (
      <a
        href={page.url.prev}
        class="bg-gray-100 text-foreground w-11 h-11 flex justify-center items-center"
        aria-label="Ir a la página anterior"
        rel="prev"
        aria-disabled={page.url.prev === page.url.first}
      >
        <ChevronLeft />
      </a>
    )
  }
  {
    [...Array(page.lastPage)].map((_, i) => (
      <a
        class={`w-11 h-11 flex justify-center items-center ${i + 1 === page.currentPage ? "bg-primary text-white" : "bg-gray-100 text-foreground"}`}
        href={getPageUrl(i + 1)}
      >
        {i + 1}
      </a>
    ))
  }
  {
    page.url.next && (
      <a
        href={page.url.next}
        class="bg-gray-100 text-foreground w-11 h-11 flex justify-center items-center"
        aria-label="Ir a la página siguiente"
        rel="next"
        aria-disabled={page.url.next === page.url.last}
      >
        <ChevronRight />
      </a>
    )
  }
</nav>
````

### Ordenamiento y filtrado en paginación

````astro
---
export async function getStaticPaths({ paginate }: GetStaticPathsOptions) {
  const { data: articles } = await fetchApi<{ data: Article[] }>({
    endpoint: 'articles?populate=image&sort=publishedAt:desc&filters[status][$eq]=published',
  });

  return paginate(articles, { pageSize: 10 });
}
---
````

---


## Trabajar con componentes Astro

### Componentes de sección reutilizables

Crea `src/components/ProductCard.astro`:

````astro
---
import { getImageUrl } from '@/lib/strapi';

interface Props {
  product: {
    id: number;
    title: string;
    description: string;
    image: { url: string };
  };
}

const { product } = Astro.props;
const imageUrl = getImageUrl(product.image.url);
---

<article class="rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
  <img 
    src={imageUrl} 
    alt={product.title}
    class="w-full h-48 object-cover"
  />
  <div class="p-4">
    <h2 class="text-xl font-semibold mb-2">{product.title}</h2>
    <p class="text-gray-600">{product.description}</p>
  </div>
</article>
````

### Props tipadas en Astro

```astro
---
interface Props {
  title: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

const { title, variant = 'primary', disabled = false } = Astro.props;
---

<button class={`btn btn-${variant}`} disabled={disabled}>
  {title}
</button>
```

### Componentes que fetchen datos

Astro permite `await` en la sección frontmatter. Esto es seguro porque ocurre en el servidor:

````astro
---
import PostPreview from '@/components/PostPreview.astro';
import fetchApi from '@/lib/strapi';

interface Post {
  id: number;
  title: string;
  slug: string;
}

const posts = await fetchApi<Post[]>({
  endpoint: 'posts?sort=-createdAt&pagination[limit]=5',
  wrappedByList: 'data',
});
---

<section class="py-12">
  <h2 class="text-3xl font-bold mb-6">Últimas noticias</h2>
  <div class="space-y-4">
    {posts.map((post) => (
      <PostPreview post={post} />
    ))}
  </div>
</section>
````

---

## Componentes React (UI y helpers)

### UI Primitivos con Radix UI + CVA

Estos componentes son **atómicos** y viven en ui. Utilizan [Radix UI](https://www.radix-ui.com/) como base y [CVA](https://cva.style/) para variantes de Tailwind.

Ejemplo: button.tsx

````tsx
import { cva, type VariantProps } from 'class-variance-authority';
import React from 'react';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors focus-visible:outline-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-blue-600 text-white hover:bg-blue-700',
        outline: 'border border-gray-300 hover:bg-gray-50',
        ghost: 'hover:bg-gray-100 text-gray-900',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4 text-base',
        lg: 'h-12 px-6 text-lg',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

interface ButtonProps 
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, ...props }, ref) => (
    <button
      className={buttonVariants({ variant, size, className })}
      ref={ref}
      {...props}
    />
  )
);

Button.displayName = 'Button';

export default Button;
````

**Uso en Astro:**

```astro
---
import Button from '@/components/ui/button.tsx';
---

<Button client:load variant="primary" size="lg">
  Click me
</Button>
```

### Componentes React complejos (Helpers)

Para componentes con **estado interno, event handlers y lógica de negocio**, usa helpers. Estos se cargan con `client:load` porque necesitan hidratación inmediata.

Ejemplo: StrapiRichText.tsx

````tsx
import React, { useState } from 'react';

interface RichTextProps {
  content: string;
  maxLength?: number;
}

const StrapiRichText: React.FC<RichTextProps> = ({ content, maxLength = 200 }) => {
  const [expanded, setExpanded] = useState(false);

  const displayText = expanded ? content : content.slice(0, maxLength) + '...';

  return (
    <div className="prose prose-sm max-w-none">
      <p>{displayText}</p>
      {content.length > maxLength && (
        <button
          onClick={() => setExpanded(!expanded)}
          className="text-blue-600 hover:underline mt-2"
        >
          {expanded ? 'Mostrar menos' : 'Mostrar más'}
        </button>
      )}
    </div>
  );
};

export default StrapiRichText;
````

**Uso en Astro:**

```astro
---
import StrapiRichText from '@/helpers/StrapiRichText.tsx';
---

<StrapiRichText client:load content={largeText} maxLength={150} />
```

### Diferencia: UI vs Helpers

| Aspecto | UI (`components/ui/`) | Helpers (`helpers/`) |
|--------|----------------------|----------------------|
| **Lógica** | Presentación | Negocio / interactividad |
| **Props** | Simples | Complejas |
| **Ejemplos** | button, input, dialog | carrusel, formulario, modal con lógica |
| **Directiva** | No tiene por qué | `client:X` si tiene interactividad |

---

## Fetching de datos desde Strapi

### La función `fetchApi`

Todos los fetches a Strapi deben pasar por strapi.ts. Esta función:

- Añade automáticamente el header `Authorization: Bearer ${STRAPI_API_TOKEN}`
- Construye la URL usando `PUBLIC_STRAPI_URL`
- Maneja responses envueltos en `data` o listas

**Uso básico:**

````ts
import fetchApi from '@/lib/strapi';

// Fetch un objeto
const config = await fetchApi<SeoConfig>({
  endpoint: 'seo-config?populate=*',
  wrappedByKey: 'data', // La response es de tipo SeoConfig
});

// Fetch una lista
const products = await fetchApi<Product[]>({
  endpoint: 'products?populate=image,category',
  wrappedByList: 'data', // La response es de tipo Product[]
});

// Fetch con filtros
const featured = await fetchApi<Product[]>({
  endpoint: 'products?filters[featured][$eq]=true&populate=*',
  wrappedByList: 'data',
});
````

### Construcción de URLs de imagen

Usa `getImageUrl` para construir URLs completas desde rutas relativas:

```ts
import { getImageUrl } from '@/lib/strapi';

const fullUrl = getImageUrl('/uploads/image.webp');
// → https://strapi.example.com/uploads/image.webp
```

### Manejo de errores

````ts
import fetchApi from '@/lib/strapi';

try {
  const data = await fetchApi<Product>({
    endpoint: 'products/1',
    wrappedByKey: 'data',
  });
} catch (error) {
  console.error('Error fetching product:', error);
  // Renderizar fallback o página de error
}
````

### Variables de entorno

Asegúrate de que estos .env estén configurados:

```env
PUBLIC_STRAPI_URL=https://strapi.example.com
STRAPI_API_TOKEN=your_api_token_here
```

El token es **privado** (sin `PUBLIC_` prefix) y solo se usa en el servidor.

---

## Estilos con Tailwind CSS

### Configuración

Tailwind está configurado en global.css (entry point) y astro.config.mjs (via Vite). No necesitas hacer nada adicional: **los estilos se aplican automáticamente**.

### Clases Tailwind en Astro y React

**En Astro:**

```astro
---
import Button from '@/components/ui/button.tsx';
---

<div class="container mx-auto px-4 py-12">
  <h1 class="text-4xl font-bold mb-8">Título</h1>
  <Button variant="primary">Click</Button>
</div>
```

**En React:**

```tsx
export default function Card({ title }) {
  return (
    <div className="rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-2xl font-semibold">{title}</h2>
    </div>
  );
}
```

### Responsive design

```astro
<div class="text-base md:text-lg lg:text-2xl">
  Responsive text
</div>

<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <!-- Columnas: 1 mobile, 2 tablet, 3 desktop -->
</div>
```

### Evitar CSS duplicado

- Prefiere **clases de utilidad** en lugar de CSS personalizado.
- Si necesitas estilos custom, añádelos a global.css:

```css
/* src/styles/global.css */
@layer components {
  .card-custom {
    @apply rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow;
  }
}
```

Luego usa `class="card-custom"` en lugar de repetir las clases.

---

## SEO y metadatos

### Layout automático

El componente Layout.astro maneja automáticamente:

- **Meta tags**: description, Open Graph, Twitter Card, robots
- **Canonical URL**: basada en `Astro.url.pathname`
- **Favicon**: referencias a `/fav/`
- **Structured data**: via `<RichResults />`

### Pasar datos SEO a Layout

```astro
---
import Layout from '@/layouts/Layout.astro';

const title = "Mi página";
const description = "Una descripción breve";
const image = "/images/og-image.png";
---

<Layout
  title={title}
  description={description}
  image={image}
  index={true}
  follow={true}
>
  <!-- Contenido -->
</Layout>
```

### Canonical URL personalizada

Por defecto, la canonical es `Astro.url.pathname`. Para sobrescribirla:

```astro
<Layout canonical="https://example.com/custom-path">
  <!-- ... -->
</Layout>
```

### FAQ Schema (Rich Results)

Para añadir FAQ a los rich results:

```astro
---
import Layout from '@/layouts/Layout.astro';
import type Faq from '@/interfaces/Faq';

const faqs: Faq[] = [
  {
    question: "¿Cuál es tu política de devolución?",
    answer: "Aceptamos devoluciones en 30 días..."
  },
  // ...
];
---

<Layout
  title="Preguntas frecuentes"
  faq={faqs}
>
  <!-- Contenido -->
</Layout>
```

---

## Mejores prácticas

### 1. Usa la ruta `@/` para imports

✅ **Correcto:**
```ts
import Layout from '@/layouts/Layout.astro';
import fetchApi from '@/lib/strapi';
import type SeoConfig from '@/interfaces/SeoConfig';
```

❌ **Incorrecto:**
```ts
import Layout from '../../../layouts/Layout.astro';
import fetchApi from '../../../lib/strapi';
```

### 2. Fetching: servidor, no cliente

✅ **Correcto (en componente Astro):**
```astro
---
const data = await fetchApi({ endpoint: 'products' });
---

<div>{data.map(...)}</div>
```

❌ **Incorrecto (en componente React):**
```tsx
const [data, setData] = useState([]);

useEffect(() => {
  fetch('/api/products').then(res => res.json()).then(setData);
}, []);
```

### 3. Separa UI de lógica

✅ **Correcto:**
- button.tsx → presentación pura
- `src/helpers/FormWithValidation.tsx` → state + validación

❌ **Incorrecto:**
- Lógica de validación en `button.tsx`
- Presentación compleja en `FormWithValidation.tsx` sin separación

### 4. Tipos en interfaces

✅ **Correcto:**
```ts
// src/interfaces/Product.ts
export default interface Product {
  id: number;
  title: string;
}

// En un componente:
import type Product from '@/interfaces/Product';
const products = await fetchApi<Product[]>({ ... });
```

### 5. Usa `wrappedByKey`

Strapi envuelve responses. La función `fetchApi` lo maneja automáticamente:

```ts
const posts = await fetchApi<Post[]>({
  endpoint: "posts?populate=*",
  wrappedByKey: "data", // Devuelve un array de posts
});
```

### 6. Props opcionales en Layouts

Si una página no necesita FAQ o preload, no la pases:

```astro
<Layout
  title="Simple page"
  description="No hay FAQ aquí"
>
  <!-- ... -->
</Layout>
```

### 7. Manejo de imágenes

- Siempre usa `getImageUrl()` para URLs de Strapi.
- Guarda assets estáticos en public y referencialos como `/path/to/asset`.
- Añade `alt` text a todas las imágenes.

```astro
---
import { getImageUrl } from '@/lib/strapi';
const imageUrl = getImageUrl(product.image.url);
---

<img src={imageUrl} alt={product.title} />
<img src="/images/logo.svg" alt="Logo de la empresa" />
```

### 8. Variables globales reutilizables

Guarda datos globales en seo-config.ts. El Layout.astro los mutará con la configuración de Strapi:

```ts
// src/lib/seo-config.ts
export const seoConfig = {
  siteName: 'Mi Sitio',
  baseURL: 'https://example.com',
  image: { url: '/images/og-default.png' },
  twitter: { site: '@mysite' },
};
```

Luego en cualquier componente:

```ts
import { seoConfig } from '@/lib/seo-config';

console.log(seoConfig.siteName); // Valores por defecto o fetched
```

---

## Checklist para crear una página nueva

- [ ] Crear archivo `src/pages/mi-pagina.astro`
- [ ] Importar `Layout` y cualquier componente necesario
- [ ] Fetchar datos en el frontmatter (si aplica) usando `fetchApi`
- [ ] Pasar `title` y `description` a `Layout`
- [ ] Estructura HTML semántica (h1, section, article, etc.)
- [ ] Clases Tailwind para responsive design
- [ ] Props SEO adicionales si es necesario (canonical, index, follow, faq)
- [ ] Testear en `pnpm dev` (debe escuchar en `localhost:4321`)

---

## Recursos

- [Documentación de Astro](https://docs.astro.build)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Radix UI](https://www.radix-ui.com/)
- [CVA (Class Variance Authority)](https://cva.style/)
- [Strapi REST API](https://docs.strapi.io/dev-docs/api/rest)

---

**¿Dudas?** Consulta el archivo copilot-instructions.md para detalles de arquitectura o comunícate con el equipo de desarrollo.