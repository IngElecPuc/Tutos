# Guía de SEO, campañas web y adquisición digital con Next.js

## Objetivo

Esta guía explica cómo diseñar, desarrollar y operar una estrategia de SEO y campañas digitales para sitios web modernos, con foco técnico en Next.js. Cubre desde fundamentos básicos hasta prácticas avanzadas para empresas, páginas personales, portales de venta, SaaS, intranets, extranets y casos híbridos.

Está organizada en tres niveles:

```text
1. Básico:
   SEO técnico mínimo, contenido, keywords, metadata, indexación, robots, sitemap y estructura de sitio.

2. Intermedio:
   Next.js App Router, metadata dinámica, Open Graph, JSON-LD, Core Web Vitals, Search Console, analítica, Google Ads y campañas sociales.

3. Avanzado:
   arquitectura SEO por tipo de negocio, SEO programático, marketplaces, e-commerce, internacionalización, privacy/consent, server-side tracking, campañas multicanal, attribution, experimentación, intranets y casos híbridos.
```

Stack recomendado, siguiendo las guías previas:

```text
Frontend:
Next.js + React + TypeScript + CSS/Tailwind opcional.

Backend:
FastAPI o Route Handlers/Server Actions de Next.js según caso.

Base de datos:
PostgreSQL.

Infraestructura:
AWS, CloudFront, S3, ALB, EC2 o despliegue administrado.

Operación:
CloudWatch, logs, QA, smoke tests, CI/CD.

Seguridad:
Autenticación fuerte, backend como autoridad, cookies seguras, control de indexación.

Marketing:
Google Search Console, Google Analytics 4, Google Ads, Meta Ads, LinkedIn Ads, pixels, eventos y conversiones.
```

---

# Parte I: fundamentos básicos de SEO

## 1. Qué es SEO

SEO significa Search Engine Optimization. Es el conjunto de prácticas para que un sitio pueda ser descubierto, entendido, indexado y mostrado correctamente por motores de búsqueda.

SEO no es solo “poner palabras clave”. Incluye:

```text
Arquitectura del sitio.
Calidad del contenido.
Performance.
HTML semántico.
Metadata.
Enlaces internos.
Sitemaps.
Robots.
Datos estructurados.
Accesibilidad.
Experiencia móvil.
Autoridad.
Confianza.
Medición.
```

Una definición práctica:

```text
SEO es hacer que el contenido correcto sea accesible, comprensible, útil y confiable para usuarios y buscadores.
```

---

## 2. Cómo funciona la búsqueda

De forma simplificada:

```text
Crawling:
El buscador descubre URLs.

Indexing:
El buscador procesa el contenido y decide si lo guarda en el índice.

Serving / Ranking:
El buscador decide qué resultados mostrar ante una búsqueda.
```

Implicación:

```text
Una página puede existir y aun así no aparecer en Google si:
- No es descubierta.
- Está bloqueada.
- Tiene noindex.
- Es duplicada.
- Tiene poco valor.
- Carga mal.
- Requiere autenticación.
- El contenido depende de JavaScript mal renderizado.
```

---

## 3. SEO técnico vs SEO de contenido

### SEO técnico

Responde:

```text
¿El buscador puede acceder?
¿Puede entender la página?
¿La página carga rápido?
¿La URL es canónica?
¿Hay sitemap?
¿Hay metadata?
¿Hay errores 404/500?
¿La página funciona en mobile?
```

### SEO de contenido

Responde:

```text
¿La página satisface una intención de búsqueda?
¿El contenido es útil?
¿Tiene profundidad suficiente?
¿Está actualizado?
¿Tiene autoridad?
¿Está bien estructurado?
¿Responde mejor que alternativas?
```

Ambos son necesarios. Un sitio técnicamente perfecto con contenido débil no suele rendir. Un gran contenido con indexación rota tampoco.

---

## 4. SEO orgánico vs campañas pagadas

| Canal | Qué es | Ventaja | Riesgo |
|---|---|---|---|
| SEO orgánico | Tráfico desde resultados no pagados | Compuesto, durable, reduce CAC a largo plazo | Lento, incierto |
| Google Ads Search | Anuncios en búsquedas | Intención alta, resultados rápidos | Costo por clic, competencia |
| Google Display/YouTube | Anuncios visuales/video | Alcance y remarketing | Menor intención directa |
| Meta Ads | Facebook/Instagram | Segmentación, demanda latente, creatividad | Requiere buenos eventos y creatividades |
| LinkedIn Ads | B2B profesional | Segmentación laboral/empresa | CPC alto |
| Email/CRM | Audiencias propias | Retención y conversión | Requiere base legítima |
| Social orgánico | Contenido en redes | Marca y comunidad | Alcance variable |

Regla:

```text
SEO captura demanda existente.
Ads puede capturar demanda existente o crear demanda.
Marketing de contenidos puede construir demanda futura.
```

---

## 5. Intención de búsqueda

Antes de crear una página, identifica la intención.

Tipos:

```text
Informacional:
“cómo instalar postgresql ubuntu”

Navegacional:
“login banco estado”

Comercial:
“mejor CRM para pymes”

Transaccional:
“comprar notebook i7 32gb”

Local:
“dentista providencia”

Soporte:
“cómo cambiar contraseña plataforma x”
```

La página debe responder a la intención.

Ejemplo:

```text
Keyword:
“software de gestión de tareas para equipos”

Intención:
Comercial / comparación.

Página adecuada:
Landing de producto con casos de uso, beneficios, precios, comparación, demos y CTA.

Página inadecuada:
Post genérico “qué es una tarea”.
```

---

# Parte II: selección de palabras clave

## 6. Qué son keywords

Una keyword es una consulta o conjunto de términos que el usuario escribe o dicta.

Ejemplos:

```text
“agencia de marketing digital”
“software inventario pymes”
“consultor data science chile”
“curso react nextjs”
“comprar zapatos mujer cuero”
```

No optimices solo por palabras. Optimiza por intención y tema.

---

## 7. Tipos de keywords

```text
Head terms:
Cortas y amplias.
Ejemplo: “CRM”.

Middle-tail:
Más específicas.
Ejemplo: “CRM para inmobiliarias”.

Long-tail:
Muy específicas.
Ejemplo: “CRM para corredores de propiedades en Chile con WhatsApp”.
```

Regla:

```text
Los términos amplios tienen más volumen, pero suelen ser más competitivos y ambiguos.
Las long-tail tienen menos volumen, pero mejor intención y conversión.
```

---

## 8. Cómo seleccionar keywords

Proceso:

```text
1. Definir negocio.
2. Definir segmentos.
3. Definir problemas.
4. Listar consultas posibles.
5. Clasificar intención.
6. Revisar competencia.
7. Revisar volumen aproximado.
8. Evaluar dificultad.
9. Mapear keyword a página.
10. Medir resultados.
```

Plantilla:

| Keyword | Intención | Página | Etapa funnel | Prioridad |
|---|---|---|---|---|
| software inventario pymes | Comercial | Landing producto | Consideración | Alta |
| cómo controlar stock | Informacional | Blog/guía | Awareness | Media |
| comprar sistema inventario | Transaccional | Landing demo/precios | Decisión | Alta |
| sistema inventario excel vs app | Comercial | Comparativa | Consideración | Media |

---

## 9. Fuentes para investigar keywords

Fuentes útiles:

```text
Google Keyword Planner.
Google Search Console.
Google Trends.
Resultados sugeridos de Google.
People Also Ask.
Búsquedas relacionadas.
Competidores.
Preguntas de clientes.
Tickets de soporte.
Vendedores.
Comunidades.
Redes sociales.
Foros.
YouTube.
Marketplaces.
```

No te quedes solo con herramientas. Muchas buenas keywords salen de lenguaje real de clientes.

---

## 10. Keyword mapping

Cada página debe tener un objetivo claro.

Malo:

```text
Cinco páginas compiten por “software inventario”.
```

Mejor:

```text
/software-inventario-pymes
    Keyword principal: software inventario pymes.

/software-inventario-restaurantes
    Keyword principal: software inventario restaurantes.

/blog/como-controlar-stock
    Keyword principal: cómo controlar stock.
```

Regla:

```text
Una intención principal por página.
Una página puede cubrir keywords secundarias relacionadas.
```

---

## 11. Evitar canibalización

Canibalización ocurre cuando varias páginas del mismo sitio compiten por la misma intención.

Señales:

```text
Google alterna varias URLs para la misma consulta.
Ninguna página rankea fuerte.
Hay contenidos muy parecidos.
Search Console muestra impresiones divididas.
```

Soluciones:

```text
Fusionar contenidos.
Definir página canónica.
Redireccionar duplicados.
Diferenciar intención.
Actualizar enlaces internos.
```

---

## 12. Keywords por tipo de negocio

### Página de empresa B2B

Keywords:

```text
servicio + industria
consultoría + problema
empresa + solución
software + caso de uso
```

Ejemplos:

```text
consultoría data science retail
automatización procesos administrativos
desarrollo software a medida chile
```

### Página personal

Keywords:

```text
nombre profesional
rol + especialidad
servicio freelance
portafolio + tecnología
```

Ejemplos:

```text
consultor python chile
desarrollador nextjs freelance
portafolio data scientist
```

### Portal de venta

Keywords:

```text
comprar + producto
producto + marca
producto + atributo
categoría + uso
```

Ejemplos:

```text
comprar silla ergonómica
notebook gamer rtx 4060
zapatillas running mujer
```

### SaaS

Keywords:

```text
software + problema
herramienta + proceso
alternativa a competidor
plantilla + caso de uso
```

Ejemplos:

```text
software gestión tareas equipos
herramienta mesa de ayuda pymes
alternativa a trello para operaciones
```

### Intranet

Keywords públicas normalmente no importan.

Prioridad:

```text
No indexar.
Acceso autenticado.
Experiencia interna.
Buscador interno.
Documentación clara.
```

---

# Parte III: arquitectura de información

## 13. Estructura de sitio

Una estructura clara ayuda a usuarios y buscadores.

Ejemplo empresa:

```text
/
 /servicios
 /servicios/desarrollo-web
 /servicios/data-science
 /casos
 /casos/retail-demand-forecasting
 /blog
 /contacto
```

Ejemplo e-commerce:

```text
/
 /categorias
 /categorias/zapatillas
 /categorias/zapatillas/running
 /productos/nike-pegasus-41
 /marcas/nike
 /blog/como-elegir-zapatillas-running
```

Ejemplo SaaS:

```text
/
 /features
 /features/automatizacion
 /features/reportes
 /casos-de-uso/equipos-soporte
 /precios
 /comparativas/alternativa-a-trello
 /docs
 /blog
```

---

## 14. URLs SEO-friendly

Buenas URLs:

```text
/desarrollo-web-empresas
/blog/como-crear-sitemap-nextjs
/productos/silla-ergonomica-x100
```

Malas URLs:

```text
/page?id=123
/product.php?cat=7&item=55
/blog/post_2026_07_05_final_v3
```

Reglas:

```text
Minúsculas.
Guiones medios.
Sin acentos.
Descriptivas.
Estables.
Cortas, pero claras.
```

---

## 15. Enlaces internos

Los enlaces internos ayudan a:

```text
Descubrimiento de páginas.
Distribución de autoridad interna.
Comprensión temática.
Navegación de usuarios.
Conversión.
```

Buenas prácticas:

```text
Usa anchor text descriptivo.
Enlaza páginas relacionadas.
Crea rutas desde páginas fuertes a páginas nuevas.
No escondas páginas importantes.
Evita orphan pages.
```

Malo:

```html
<a href="/servicios/desarrollo-web">click aquí</a>
```

Mejor:

```html
<a href="/servicios/desarrollo-web">servicio de desarrollo web para empresas</a>
```

---

## 16. Menú, breadcrumbs y footer

Elementos recomendados:

```text
Menú principal:
Páginas principales.

Breadcrumbs:
Ubicación jerárquica.

Footer:
Links corporativos, legales, contacto, categorías principales.

Sitemap HTML:
Útil en sitios grandes.
```

Breadcrumb:

```text
Inicio > Servicios > Desarrollo web
```

En e-commerce, los breadcrumbs ayudan especialmente por jerarquía de categorías.

---

# Parte IV: SEO técnico básico

## 17. Title tag

El título es uno de los elementos más importantes.

Debe:

```text
Describir la página.
Incluir intención principal.
Ser único.
No ser artificial.
Tener longitud razonable.
```

Ejemplo empresa:

```text
Desarrollo web para empresas | Acme Digital
```

Ejemplo producto:

```text
Silla ergonómica X100 con soporte lumbar | Tienda Acme
```

Ejemplo blog:

```text
Cómo crear un sitemap en Next.js App Router
```

---

## 18. Meta description

No es factor directo de ranking en el mismo sentido que el contenido, pero puede influir en clics.

Debe:

```text
Resumir valor.
Incluir propuesta.
Tener CTA suave.
Ser única.
```

Ejemplo:

```text
Aprende a implementar SEO técnico en Next.js con metadata, sitemap, robots, canonical, Open Graph y datos estructurados.
```

---

## 19. Headings

Usa encabezados para estructura, no para tamaño visual.

Reglas:

```text
Un H1 principal por página.
H2 para secciones.
H3 para subsecciones.
No saltar jerarquías sin razón.
No usar headings solo para estilo.
```

Ejemplo:

```html
<h1>Software de inventario para pymes</h1>
<h2>Control de stock en tiempo real</h2>
<h2>Reportes y alertas</h2>
<h2>Preguntas frecuentes</h2>
```

---

## 20. HTML semántico

Usa elementos con significado:

```html
<header>
<nav>
<main>
<article>
<section>
<aside>
<footer>
```

Ejemplo:

```tsx
export default function Page() {
  return (
    <>
      <header>
        <nav>{/* enlaces */}</nav>
      </header>

      <main>
        <article>
          <h1>Guía de SEO para Next.js</h1>
          <p>Contenido principal.</p>
        </article>
      </main>

      <footer>© Empresa</footer>
    </>
  )
}
```

---

## 21. Imágenes

Buenas prácticas:

```text
Usar alt descriptivo.
Comprimir imágenes.
Usar dimensiones correctas.
Usar formatos modernos.
Evitar imágenes enormes.
Usar lazy loading cuando corresponda.
```

En Next.js:

```tsx
import Image from 'next/image'

export function HeroImage() {
  return (
    <Image
      src="/images/desarrollo-web-empresas.webp"
      alt="Equipo revisando una plataforma web empresarial"
      width={1200}
      height={630}
      priority
    />
  )
}
```

`priority` solo para imágenes críticas above-the-fold.

---

## 22. Robots.txt

`robots.txt` indica a crawlers qué URLs pueden rastrear. No es un mecanismo de seguridad ni garantiza que una URL no aparezca indexada si hay otros enlaces o señales.

Ejemplo público:

```text
User-Agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
```

Ejemplo con área privada:

```text
User-Agent: *
Allow: /
Disallow: /admin/
Disallow: /private/

Sitemap: https://example.com/sitemap.xml
```

Para contenido que no quieres indexar:

```text
Usa noindex.
O requiere autenticación.
O ambas.
```

No uses robots.txt como protección de datos sensibles.

---

## 23. Noindex

`noindex` indica que una página no debe aparecer en resultados.

Útil para:

```text
Paneles internos.
Páginas de staging.
Resultados de búsqueda internos.
Páginas duplicadas.
Filtros sin valor SEO.
Flujos de checkout.
Páginas de cuenta.
```

HTML:

```html
<meta name="robots" content="noindex, nofollow">
```

En Next.js:

```tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  robots: {
    index: false,
    follow: false,
  },
}
```

---

## 24. Sitemap

Un sitemap XML ayuda a descubrir URLs importantes.

Debe incluir:

```text
URLs canónicas.
Páginas indexables.
Fechas de actualización si son confiables.
```

No debe incluir:

```text
Páginas noindex.
Páginas privadas.
Errores 404.
Redirecciones.
URLs duplicadas.
Filtros sin valor.
```

---

## 25. Canonical

Canonical indica la URL preferida cuando hay duplicados o contenido similar.

Ejemplo:

```html
<link rel="canonical" href="https://example.com/productos/silla-x100">
```

Casos frecuentes:

```text
Parámetros UTM.
Paginación.
Filtros.
Versiones con slash/sin slash.
HTTP vs HTTPS.
www vs no-www.
Contenido duplicado por categorías.
```

---

# Parte V: SEO con Next.js App Router

## 26. Por qué Next.js ayuda al SEO

Next.js ayuda porque permite:

```text
Renderizado del lado servidor.
Static generation.
Metadata por ruta.
Sitemaps dinámicos.
Robots dinámico.
Open Graph dinámico.
Imágenes optimizadas.
Buen performance si está bien implementado.
```

Pero Next.js no garantiza SEO automáticamente. Debes configurar contenido, metadata, indexación, performance y medición.

---

## 27. Metadata estática

Archivo:

```tsx
// app/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  metadataBase: new URL('https://example.com'),
  title: {
    default: 'Acme Digital',
    template: '%s | Acme Digital',
  },
  description: 'Desarrollo web, datos y automatización para empresas.',
  alternates: {
    canonical: '/',
  },
  openGraph: {
    type: 'website',
    siteName: 'Acme Digital',
    title: 'Acme Digital',
    description: 'Desarrollo web, datos y automatización para empresas.',
    url: '/',
    images: [
      {
        url: '/og/default.png',
        width: 1200,
        height: 630,
        alt: 'Acme Digital',
      },
    ],
  },
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="es">
      <body>{children}</body>
    </html>
  )
}
```

---

## 28. Metadata por página

```tsx
// app/servicios/desarrollo-web/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Desarrollo web para empresas',
  description:
    'Creamos sitios y plataformas web rápidas, seguras y optimizadas para SEO.',
  alternates: {
    canonical: '/servicios/desarrollo-web',
  },
}

export default function Page() {
  return (
    <main>
      <h1>Desarrollo web para empresas</h1>
      <p>Diseñamos y construimos plataformas web modernas.</p>
    </main>
  )
}
```

---

## 29. Metadata dinámica con `generateMetadata`

Útil para productos, posts, categorías o perfiles.

```tsx
// app/productos/[slug]/page.tsx
import type { Metadata } from 'next'

type Props = {
  params: Promise<{ slug: string }>
}

async function getProduct(slug: string) {
  return {
    name: 'Silla ergonómica X100',
    description: 'Silla ergonómica con soporte lumbar.',
    image: '/products/silla-x100.webp',
  }
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params
  const product = await getProduct(slug)

  return {
    title: product.name,
    description: product.description,
    alternates: {
      canonical: `/productos/${slug}`,
    },
    openGraph: {
      title: product.name,
      description: product.description,
      images: [
        {
          url: product.image,
          width: 1200,
          height: 630,
          alt: product.name,
        },
      ],
    },
  }
}

export default async function ProductPage({ params }: Props) {
  const { slug } = await params
  const product = await getProduct(slug)

  return (
    <main>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </main>
  )
}
```

---

## 30. Robots en Next.js

Archivo estático:

```text
app/robots.txt
```

Contenido:

```text
User-Agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
```

Archivo dinámico:

```tsx
// app/robots.ts
import type { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  const baseUrl = 'https://example.com'

  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/admin/', '/cuenta/', '/checkout/'],
      },
    ],
    sitemap: `${baseUrl}/sitemap.xml`,
  }
}
```

---

## 31. Sitemap en Next.js

```tsx
// app/sitemap.ts
import type { MetadataRoute } from 'next'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://example.com'

  const staticRoutes = [
    '',
    '/servicios',
    '/servicios/desarrollo-web',
    '/blog',
    '/contacto',
  ]

  return staticRoutes.map((route) => ({
    url: `${baseUrl}${route}`,
    lastModified: new Date(),
    changeFrequency: 'weekly',
    priority: route === '' ? 1 : 0.7,
  }))
}
```

Para productos dinámicos:

```tsx
async function getProducts() {
  return [
    {
      slug: 'silla-ergonomica-x100',
      updatedAt: new Date(),
    },
  ]
}

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://example.com'
  const products = await getProducts()

  return products.map((product) => ({
    url: `${baseUrl}/productos/${product.slug}`,
    lastModified: product.updatedAt,
  }))
}
```

---

## 32. JSON-LD en Next.js

Google recomienda JSON-LD como formato común para structured data.

Ejemplo Organization:

```tsx
export function OrganizationJsonLd() {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Acme Digital',
    url: 'https://example.com',
    logo: 'https://example.com/logo.png',
    sameAs: [
      'https://www.linkedin.com/company/acme',
      'https://www.instagram.com/acme',
    ],
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data),
      }}
    />
  )
}
```

Producto:

```tsx
export function ProductJsonLd({
  name,
  description,
  image,
  price,
}: {
  name: string
  description: string
  image: string
  price: number
}) {
  const data = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name,
    description,
    image,
    offers: {
      '@type': 'Offer',
      priceCurrency: 'CLP',
      price,
      availability: 'https://schema.org/InStock',
    },
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data),
      }}
    />
  )
}
```

Reglas:

```text
El structured data debe describir el contenido visible.
No inventes reviews, precios o disponibilidad.
Usa el tipo más específico posible.
Valida con herramientas de rich results.
```

---

## 33. Open Graph y redes sociales

Open Graph mejora cómo se ve una página al compartirla en redes.

Metadata:

```tsx
export const metadata: Metadata = {
  title: 'Guía de SEO para Next.js',
  description: 'SEO técnico, contenido, campañas y analítica.',
  openGraph: {
    title: 'Guía de SEO para Next.js',
    description: 'SEO técnico, contenido, campañas y analítica.',
    url: 'https://example.com/blog/seo-nextjs',
    siteName: 'Acme Digital',
    images: [
      {
        url: 'https://example.com/og/seo-nextjs.png',
        width: 1200,
        height: 630,
        alt: 'Guía de SEO para Next.js',
      },
    ],
    locale: 'es_CL',
    type: 'article',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Guía de SEO para Next.js',
    description: 'SEO técnico, contenido, campañas y analítica.',
    images: ['https://example.com/og/seo-nextjs.png'],
  },
}
```

---

## 34. OG images dinámicas

Next.js permite generar imágenes OG dinámicas.

```tsx
// app/blog/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og'

export const size = {
  width: 1200,
  height: 630,
}

export const contentType = 'image/png'

export default async function Image({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params

  return new ImageResponse(
    (
      <div
        style={{
          background: '#111827',
          color: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: 64,
          padding: 64,
        }}
      >
        {slug}
      </div>
    ),
    size,
  )
}
```

Uso:

```text
Blogs.
Productos.
Perfiles.
Casos de estudio.
Eventos.
```

---

# Parte VI: indexación por caso de negocio

## 35. Páginas que quieren exposición completa

Ejemplos:

```text
Página de empresa.
Blog público.
Portal de venta.
Marketplace.
Página personal profesional.
SaaS con landing pública.
Documentación pública.
```

Configuración:

```text
robots.txt permite crawling.
Páginas indexables.
Sitemap completo.
Metadata completa.
Canonical bien definido.
Structured data cuando aplica.
Contenido visible sin login.
Performance alta.
Open Graph completo.
```

Ejemplo Next.js:

```tsx
export const metadata: Metadata = {
  robots: {
    index: true,
    follow: true,
  },
}
```

---

## 36. Intranets que no quieren exposición

Ejemplos:

```text
Portal interno de empleados.
Panel administrativo.
Dashboard privado.
Sistema de gestión interno.
Backoffice.
```

Reglas:

```text
Autenticación obligatoria.
No depender solo de robots.txt.
noindex por defensa adicional.
No incluir en sitemap.
No usar metadata de marketing.
No exponer datos en HTML público.
Control de acceso backend.
```

Next.js:

```tsx
export const metadata: Metadata = {
  title: 'Intranet',
  robots: {
    index: false,
    follow: false,
    nocache: true,
  },
}
```

Middleware o layout protegido:

```tsx
// app/(private)/layout.tsx
import { redirect } from 'next/navigation'

async function getSession() {
  return null
}

export default async function PrivateLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await getSession()

  if (!session) {
    redirect('/login')
  }

  return <>{children}</>
}
```

Robots:

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: [
      {
        userAgent: '*',
        disallow: ['/admin/', '/intranet/', '/api/'],
      },
    ],
  }
}
```

Pero recuerda:

```text
robots.txt no es seguridad.
La seguridad real es autenticación, autorización y no exponer datos.
```

---

## 37. Casos híbridos

Ejemplos:

```text
SaaS:
Landing pública + app privada.

E-commerce:
Catálogo público + cuenta/checkout privado.

Portal educativo:
Cursos públicos + campus privado.

Marketplace:
Listings públicos + panel de vendedores privado.

Empresa:
Sitio corporativo público + intranet empleados.
```

Arquitectura:

```text
/(public)
    /
    /servicios
    /blog
    /precios

/(private)
    /dashboard
    /cuenta
    /admin
    /checkout
```

Metadata:

```text
Público:
index, follow.

Privado:
noindex, nofollow, auth obligatoria.
```

Sitemap:

```text
Incluir solo rutas públicas indexables.
Excluir dashboard, cuenta, checkout y admin.
```

---

## 38. Staging y ambientes de prueba

Staging no debe indexarse.

Opciones:

```text
Autenticación básica.
IP allowlist.
noindex.
robots disallow.
Dominio no público.
```

Configuración Next.js:

```tsx
const isProduction = process.env.NEXT_PUBLIC_APP_ENV === 'production'

export const metadata: Metadata = {
  robots: {
    index: isProduction,
    follow: isProduction,
  },
}
```

Mejor todavía:

```text
Proteger staging con autenticación.
```

No confíes solo en `noindex` para información sensible.

---

# Parte VII: SEO por tipo de sitio

## 39. Página de empresa

Objetivo:

```text
Generar confianza.
Explicar servicios.
Capturar leads.
Mostrar casos.
Posicionar marca.
```

Páginas mínimas:

```text
Inicio.
Servicios.
Servicio específico por intención.
Casos de éxito.
Sobre nosotros.
Blog o recursos.
Contacto.
Política de privacidad.
```

SEO:

```text
Keywords por servicio.
Keywords por industria.
Casos con resultados.
Schema Organization.
Schema LocalBusiness si aplica.
CTA claro.
Pruebas de confianza.
```

Ejemplo estructura:

```text
/servicios/desarrollo-web
/servicios/data-science
/servicios/automatizacion
/casos/forecasting-retail
/blog/como-automatizar-reportes
```

---

## 40. Página personal

Objetivo:

```text
Reputación.
Portafolio.
Contacto.
Autoridad temática.
Búsqueda por nombre.
Servicios profesionales.
```

Páginas:

```text
Inicio.
Sobre mí.
Proyectos.
Blog.
Servicios.
Contacto.
CV o experiencia.
```

SEO:

```text
Nombre completo.
Rol profesional.
Tecnologías.
Casos concretos.
Schema Person.
Open Graph cuidado.
```

Ejemplo keywords:

```text
Felipe científico de datos
consultor python
desarrollador nextjs freelance
portafolio data science
```

---

## 41. Portal de venta / e-commerce

Objetivo:

```text
Capturar demanda transaccional.
Indexar categorías y productos.
Convertir tráfico en ventas.
```

Páginas:

```text
Categorías.
Subcategorías.
Productos.
Marcas.
Comparativas.
Guías de compra.
Ofertas.
Preguntas frecuentes.
```

SEO:

```text
Schema Product.
Schema Offer.
Schema BreadcrumbList.
Canonical para variantes.
Noindex en filtros sin valor.
Control de faceted navigation.
Imágenes optimizadas.
Reviews reales.
Stock y precio actualizados.
```

Ejemplo:

```text
/categorias/sillas-ergonomicas
/productos/silla-ergonomica-x100
/blog/como-elegir-silla-ergonomica
```

---

## 42. SaaS

Objetivo:

```text
Generar demanda.
Explicar problema.
Capturar leads.
Activar trials.
Compararse con alternativas.
```

Páginas:

```text
Inicio.
Features.
Casos de uso.
Industrias.
Precios.
Comparativas.
Integraciones.
Blog.
Docs.
Changelog.
```

SEO:

```text
Keywords por problema.
Keywords por alternativa.
Keywords por industria.
Landing por caso de uso.
Schema SoftwareApplication.
Contenido educativo.
Lead magnets.
```

Ejemplo:

```text
/casos-de-uso/equipos-soporte
/features/automatizacion
/comparativas/alternativa-a-trello
/integraciones/slack
```

---

## 43. Portal de contenido

Objetivo:

```text
Capturar búsquedas informacionales.
Construir autoridad.
Convertir usuarios a newsletter, lead o venta.
```

SEO:

```text
Clusters temáticos.
Autores.
Fechas de actualización.
Enlaces internos.
Schema Article.
E-E-A-T: experiencia, expertise, autoridad y confianza.
Contenido actualizado.
```

Estructura:

```text
/guias
/tutoriales
/comparativas
/casos
/glosario
```

---

## 44. Documentación pública

Objetivo:

```text
Soporte.
Developer experience.
Reducción de tickets.
Adopción.
```

SEO:

```text
URLs estables.
Búsqueda interna.
Versionado.
Sitemap.
Canonical por versión.
Noindex en versiones obsoletas si corresponde.
Schema TechArticle si aplica.
```

---

# Parte VIII: campañas de Google Ads

## 45. Cuándo usar Google Ads

Google Ads es útil cuando:

```text
Hay intención de búsqueda clara.
Quieres validar demanda rápido.
Necesitas leads o ventas en el corto plazo.
Quieres testear keywords antes de invertir en SEO.
Quieres competir por búsquedas transaccionales.
```

No es ideal cuando:

```text
No hay conversión medible.
El sitio no convierte.
No hay presupuesto suficiente para aprender.
No tienes tracking.
La oferta no está clara.
```

---

## 46. Tipos de campañas relevantes

```text
Search:
Captura demanda activa.

Performance Max:
Distribuye en inventario Google usando señales y objetivos.

Display:
Awareness y remarketing.

YouTube:
Marca, consideración y remarketing.

Shopping:
E-commerce con feed de productos.

Demand Gen:
Visual, intención media y audiencias.

App:
Instalaciones o eventos de app.
```

Para empezar en B2B o servicios:

```text
Search + remarketing.
```

Para e-commerce:

```text
Shopping / Performance Max + Search de marca/categorías.
```

---

## 47. Estructura de campaña Search

Ejemplo empresa de desarrollo web:

```text
Campaña:
Servicios desarrollo web

Ad groups:
- desarrollo web empresas
- desarrollo nextjs
- desarrollo ecommerce
- agencia desarrollo web

Keywords:
[desarrollo web empresas]
"desarrollo web empresas"
+ variantes relevantes

Anuncios:
Responsive Search Ads

Landing:
Página específica por servicio.
```

Regla:

```text
Cada grupo de anuncios debe apuntar a una landing coherente con la intención.
```

---

## 48. Selección de keywords para Ads

Criterios:

```text
Intención comercial.
Relevancia.
Volumen.
Competencia.
Costo por clic.
Probabilidad de conversión.
Margen o valor del cliente.
```

Tipos de match:

```text
Exact match:
Mayor control.

Phrase match:
Balance.

Broad match:
Más amplio, requiere buenas negativas y conversion tracking.
```

Negativas:

```text
gratis
empleo
curso
pdf
plantilla
definición
```

Pero no todas son siempre negativas. Depende del negocio.

---

## 49. Landing pages para Ads

Una landing para Ads debe:

```text
Responder exactamente a la búsqueda.
Cargar rápido.
Tener título claro.
Mostrar propuesta de valor.
Tener CTA visible.
Tener prueba social.
Reducir distracciones.
Tener formulario simple.
Medir conversiones.
```

Ejemplo:

```text
Keyword:
desarrollo nextjs empresas

Landing:
Landing específica de desarrollo Next.js para empresas,
no página genérica de inicio.
```

---

## 50. Conversion tracking

Define conversiones:

```text
Lead enviado.
Compra.
Registro.
Reserva demo.
Click WhatsApp.
Llamada.
Descarga de recurso.
Inicio de trial.
```

Eventos recomendados:

```text
view_item
add_to_cart
begin_checkout
purchase
generate_lead
sign_up
contact
book_demo
```

En Next.js:

```tsx
'use client'

import Script from 'next/script'

export function GoogleTag({ id }: { id: string }) {
  return (
    <>
      <Script
        src={`https://www.googletagmanager.com/gtag/js?id=${id}`}
        strategy="afterInteractive"
      />
      <Script id="google-tag" strategy="afterInteractive">
        {`
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          window.gtag = gtag;
          gtag('js', new Date());
          gtag('config', '${id}');
        `}
      </Script>
    </>
  )
}
```

Evento:

```tsx
export function trackLead() {
  window.gtag?.('event', 'generate_lead', {
    value: 1,
    currency: 'CLP',
  })
}
```

Para producción, conviene integrar consentimiento y evitar enviar eventos antes de autorización cuando corresponda.

---

## 51. Consent Mode

Consent Mode comunica a Google si el usuario otorgó consentimiento para cookies o identificadores. No reemplaza el banner ni la política de privacidad. Interactúa con tu mecanismo de consentimiento.

Categorías frecuentes:

```text
analytics_storage
ad_storage
ad_user_data
ad_personalization
```

Patrón:

```text
1. Cargar estado por defecto denegado si aplica.
2. Mostrar banner.
3. Actualizar consentimiento según elección.
4. Disparar tags de acuerdo con consentimiento.
```

Esto es especialmente importante en sitios con tráfico europeo o jurisdicciones con reglas estrictas de privacidad.

---

# Parte IX: campañas en redes sociales

## 52. Meta Ads

Meta Ads cubre Facebook, Instagram y otros placements.

Útil para:

```text
Awareness.
Remarketing.
Generación de demanda.
E-commerce visual.
Lead forms.
Lookalikes.
Contenido educativo.
```

Requiere:

```text
Meta Pixel.
Eventos.
Conversions API si se busca medición más robusta.
Catálogo si e-commerce.
Creatividades.
Audiencias.
Consentimiento y privacidad.
```

Eventos típicos:

```text
PageView.
ViewContent.
Lead.
CompleteRegistration.
AddToCart.
InitiateCheckout.
Purchase.
```

---

## 53. Meta Pixel en Next.js

```tsx
'use client'

import Script from 'next/script'

export function MetaPixel({ pixelId }: { pixelId: string }) {
  return (
    <>
      <Script id="meta-pixel" strategy="afterInteractive">
        {`
          !function(f,b,e,v,n,t,s)
          {if(f.fbq)return;n=f.fbq=function(){n.callMethod?
          n.callMethod.apply(n,arguments):n.queue.push(arguments)};
          if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
          n.queue=[];t=b.createElement(e);t.async=!0;
          t.src=v;s=b.getElementsByTagName(e)[0];
          s.parentNode.insertBefore(t,s)}(window, document,'script',
          'https://connect.facebook.net/en_US/fbevents.js');
          fbq('init', '${pixelId}');
          fbq('track', 'PageView');
        `}
      </Script>
    </>
  )
}
```

Evento:

```tsx
export function trackLead() {
  window.fbq?.('track', 'Lead')
}
```

En producción:

```text
Cargar según consentimiento.
Evitar enviar datos sensibles.
Configurar Conversions API para eventos críticos.
Deduplicar eventos browser/server.
```

---

## 54. Meta Conversions API

Conversions API envía eventos desde servidor a Meta.

Ventajas:

```text
Más control.
Mejor resiliencia ante bloqueos de navegador.
Medición más confiable.
Integración con CRM/backend.
```

En stack Next.js:

```text
Client Component captura acción.
Server Action o Route Handler valida evento.
Backend envía evento a Meta CAPI.
Guardar event_id para deduplicación.
```

Ejemplo conceptual Route Handler:

```tsx
// app/api/track/lead/route.ts
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const payload = await request.json()

  // Validar consentimiento, origen y datos.
  // Hashear datos permitidos si corresponde.
  // Enviar a Meta Conversions API con event_id.

  return NextResponse.json({ ok: true })
}
```

No envíes datos personales sin base legal y consentimiento cuando aplique.

---

## 55. LinkedIn Ads

Útil para B2B:

```text
Servicios profesionales.
SaaS B2B.
Consultoría.
Cargos específicos.
Industrias.
Empresas objetivo.
Contenido experto.
```

Formatos:

```text
Sponsored Content.
Lead Gen Forms.
Message Ads.
Document Ads.
Video Ads.
Text Ads.
```

Casos:

```text
Consultoría data science.
Software para empresas.
Servicios cloud.
Capacitación ejecutiva.
```

Landing:

```text
Muy orientada al cargo y problema.
Caso de negocio claro.
CTA de demo, diagnóstico o recurso descargable.
```

---

## 56. Campañas orgánicas en redes

SEO y redes se conectan por:

```text
Distribución de contenido.
Construcción de marca.
Generación de enlaces.
Confianza.
Tráfico recurrente.
Remarketing.
```

Ejemplo de flujo:

```text
Publicas guía SEO.
La compartes en LinkedIn.
Generas tráfico.
Creas audiencia de remarketing.
Campaña Ads muestra demo.
Usuario convierte.
```

---

# Parte X: medición, analítica y attribution

## 57. Herramientas mínimas

```text
Google Search Console:
SEO, indexación, consultas, Core Web Vitals.

Google Analytics 4:
Eventos, usuarios, conversiones, funnels.

Google Tag Manager:
Gestión de tags.

Google Ads:
Campañas y conversiones.

Meta Events Manager:
Pixel, eventos y CAPI.

Hotjar/Microsoft Clarity:
Mapas de calor y sesiones, si privacidad lo permite.

PostHog/Amplitude/Mixpanel:
Producto, eventos y funnels.
```

---

## 58. Eventos recomendados

Para empresa B2B:

```text
view_service
click_contact
submit_contact_form
book_demo
download_case_study
```

Para e-commerce:

```text
view_item
add_to_cart
begin_checkout
purchase
refund
```

Para SaaS:

```text
sign_up
start_trial
activate_feature
invite_user
upgrade_plan
cancel_plan
```

Para página personal:

```text
view_project
click_email
download_cv
click_linkedin
```

---

## 59. UTM

Usa UTM para campañas.

Parámetros:

```text
utm_source
utm_medium
utm_campaign
utm_content
utm_term
```

Ejemplo conceptual:

```text
source: google
medium: cpc
campaign: nextjs_consulting
content: ad_variant_a
term: desarrollo nextjs empresas
```

Convención recomendada:

```text
Minúsculas.
Sin espacios.
Guiones bajos o medios.
Nombres estables.
Documentar campañas.
```

---

## 60. Dashboard mínimo

SEO:

```text
Clicks orgánicos.
Impresiones.
CTR.
Posición promedio.
Páginas indexadas.
Errores de cobertura.
Core Web Vitals.
Top queries.
Top pages.
```

Ads:

```text
Gasto.
CPC.
CTR.
Conversiones.
CPA.
ROAS.
Quality Score.
Impression share.
```

Producto:

```text
Activación.
Retención.
Conversión.
Revenue.
Churn.
Eventos clave.
```

---

# Parte XI: performance y Core Web Vitals

## 61. Métricas clave

Métricas actuales relevantes:

```text
LCP:
Largest Contentful Paint.

INP:
Interaction to Next Paint.

CLS:
Cumulative Layout Shift.

TTFB:
Time to First Byte.

FCP:
First Contentful Paint.
```

SEO y conversión sufren si el sitio es lento.

---

## 62. Buenas prácticas Next.js

```text
Usar Server Components por defecto.
Reducir Client Components.
Evitar JavaScript innecesario.
Usar next/image.
Usar next/font.
Usar caching correctamente.
Usar streaming/Suspense cuando aporta.
Optimizar queries.
Usar CDN.
Medir bundle.
```

Antipatrones:

```text
Marcar todo con 'use client'.
Cargar librerías pesadas globalmente.
Renderizar contenido crítico solo después de fetch client-side.
Imágenes sin dimensiones.
Sliders pesados en hero.
Tags de marketing sin control.
```

---

## 63. Performance y scripts de marketing

Pixels y tags pueden afectar performance.

Recomendaciones:

```text
Cargar scripts con next/script.
Elegir strategy adecuada.
No duplicar tags.
Auditar terceros.
Cargar según consentimiento.
Medir impacto en Web Vitals.
```

Next.js:

```tsx
<Script src="..." strategy="afterInteractive" />
```

Para scripts no críticos:

```tsx
<Script src="..." strategy="lazyOnload" />
```

---

# Parte XII: SEO avanzado

## 64. SEO programático

SEO programático crea muchas páginas desde datos estructurados.

Ejemplos:

```text
/servicios/desarrollo-web-para-restaurantes
/servicios/desarrollo-web-para-clinicas
/software-inventario/restaurantes
/software-inventario/retail
```

Riesgo:

```text
Páginas delgadas.
Contenido duplicado.
Escala sin calidad.
Index bloat.
```

Regla:

```text
Cada página programática debe tener valor específico, no solo cambiar una palabra.
```

---

## 65. Faceted navigation

E-commerce y marketplaces tienen filtros:

```text
/color/rojo
/talla/42
/precio/0-50000
/marca/nike
```

Problema:

```text
Miles de URLs casi duplicadas.
Crawl budget desperdiciado.
Canibalización.
Indexación de páginas sin valor.
```

Estrategia:

```text
Indexar categorías principales.
Indexar combinaciones con demanda real.
Canonical a categoría cuando el filtro no aporta valor.
Noindex en filtros sin demanda.
Controlar parámetros.
No incluir filtros irrelevantes en sitemap.
```

---

## 66. Internacionalización

Para sitios multilenguaje:

```text
URLs por idioma.
Metadata traducida.
Contenido realmente localizado.
hreflang.
Canonical por idioma.
Sitemap con alternates.
```

Ejemplo:

```text
/es/desarrollo-web
/en/web-development
/pt/desenvolvimento-web
```

Next.js con metadata:

```tsx
export const metadata: Metadata = {
  alternates: {
    canonical: '/es/desarrollo-web',
    languages: {
      es: '/es/desarrollo-web',
      en: '/en/web-development',
      pt: '/pt/desenvolvimento-web',
    },
  },
}
```

No hagas traducción automática masiva sin revisión si el objetivo es autoridad y conversión.

---

## 67. Contenido para AI Search

Las experiencias generativas de búsqueda siguen dependiendo de que el contenido sea accesible, útil y entendible. No conviene tratar “AEO” o “GEO” como disciplina separada de SEO técnico y de contenido.

Buenas prácticas:

```text
Responder preguntas con claridad.
Usar estructura semántica.
Citar fuentes cuando aplique.
Mostrar experiencia real.
Mantener contenido actualizado.
Usar datos estructurados correctos.
Evitar contenido genérico sin valor.
```

---

## 68. E-E-A-T

Para temas sensibles o competitivos, importa demostrar:

```text
Experience:
Experiencia real.

Expertise:
Conocimiento especializado.

Authoritativeness:
Autoridad reconocible.

Trustworthiness:
Confianza, transparencia y seguridad.
```

Aplicaciones:

```text
Autores visibles.
Credenciales.
Casos reales.
Fuentes.
Políticas claras.
Contacto.
Información legal.
Actualización.
Reviews verificables.
```

---

# Parte XIII: seguridad, privacidad y SEO

## 69. SEO y seguridad

Un sitio inseguro afecta confianza y campañas.

Mínimos:

```text
HTTPS.
Headers de seguridad.
Cookies seguras.
No exponer datos privados.
No indexar páginas privadas.
Protección de formularios.
Rate limiting.
Validación backend.
```

SEO no debe comprometer seguridad.

Ejemplo:

```text
No publiques una página privada solo para que Google la vea.
No pongas datos sensibles en metadata.
No incluyas URLs privadas en sitemap.
```

---

## 70. Consentimiento y privacidad

Antes de cargar pixels o tags:

```text
Revisa jurisdicción.
Define política de privacidad.
Implementa banner si corresponde.
Respeta consentimiento.
Evita recolectar datos sensibles.
Documenta proveedores.
```

Patrón:

```text
Consent default.
Banner.
Update.
Carga condicional de tags.
Server-side events solo con base legal.
```

---

# Parte XIV: QA SEO

## 71. Checklist SEO técnico

```text
[ ] Todas las páginas importantes tienen title único.
[ ] Todas las páginas importantes tienen description única.
[ ] Hay un H1 claro.
[ ] URLs son limpias.
[ ] Canonical correcto.
[ ] Sitemap existe.
[ ] Robots correcto.
[ ] Páginas privadas no están en sitemap.
[ ] Páginas privadas tienen noindex o auth.
[ ] Open Graph correcto.
[ ] JSON-LD válido cuando aplica.
[ ] No hay 404 internos.
[ ] No hay cadenas largas de redirects.
[ ] Sitio funciona en mobile.
[ ] Core Web Vitals aceptables.
[ ] Search Console configurado.
```

---

## 72. Checklist Next.js

```text
[ ] metadataBase configurado.
[ ] metadata por layout/página.
[ ] generateMetadata para rutas dinámicas.
[ ] app/sitemap.ts creado.
[ ] app/robots.ts creado.
[ ] canonical definido.
[ ] Open Graph definido.
[ ] Twitter cards definidas.
[ ] JSON-LD insertado correctamente.
[ ] Imágenes con next/image.
[ ] Se evita 'use client' innecesario.
[ ] Scripts externos auditados.
[ ] Staging bloqueado/noindex.
```

---

## 73. Checklist campañas

```text
[ ] Objetivo de campaña definido.
[ ] Conversiones configuradas.
[ ] Landing específica.
[ ] UTM consistente.
[ ] Keywords agrupadas por intención.
[ ] Negativas configuradas.
[ ] Creatividades alineadas a landing.
[ ] Pixel/Tags funcionando.
[ ] Consentimiento implementado.
[ ] Dashboard creado.
[ ] Presupuesto y CPA objetivo definidos.
```

---

# Parte XV: plan de implementación por etapas

## 74. Etapa 1: base técnica

```text
1. Definir arquitectura pública/privada.
2. Configurar Next.js metadata.
3. Crear sitemap.
4. Crear robots.
5. Agregar canonical.
6. Configurar Open Graph.
7. Configurar Search Console.
8. Revisar performance.
```

---

## 75. Etapa 2: contenido y keywords

```text
1. Definir segmentos.
2. Investigar keywords.
3. Mapear keywords a páginas.
4. Crear landings principales.
5. Crear contenidos de apoyo.
6. Crear enlaces internos.
7. Medir con Search Console.
```

---

## 76. Etapa 3: campañas

```text
1. Definir conversiones.
2. Instalar tags con consentimiento.
3. Crear landing por intención.
4. Lanzar Google Search.
5. Lanzar remarketing.
6. Probar Meta/LinkedIn según caso.
7. Medir CPA/ROAS.
8. Optimizar.
```

---

## 77. Etapa 4: avanzado

```text
1. Structured data por tipo.
2. SEO programático si hay datos suficientes.
3. Internacionalización.
4. Server-side tracking.
5. Experimentación A/B.
6. Automatización de reportes.
7. Auditorías recurrentes.
```

---

# Parte XVI: resumen de reglas principales

```text
1. SEO empieza por intención de búsqueda.
2. Cada página debe resolver una intención clara.
3. Next.js ayuda, pero no hace SEO solo.
4. Configura metadata, sitemap, robots, canonical y Open Graph.
5. Usa JSON-LD cuando describa contenido real.
6. No incluyas páginas privadas en sitemap.
7. robots.txt no es seguridad.
8. Para intranets usa autenticación y noindex.
9. Para sitios públicos prioriza contenido visible, útil y rápido.
10. Para campañas, crea landings específicas por intención.
11. No compres tráfico sin conversion tracking.
12. Usa UTM con convención estable.
13. Google Ads captura intención; redes sociales suelen crear o reactivar demanda.
14. Meta Pixel y CAPI requieren privacidad y consentimiento.
15. LinkedIn Ads sirve especialmente para B2B.
16. Mide SEO con Search Console.
17. Mide producto y campañas con eventos.
18. Optimiza Core Web Vitals.
19. Evita canibalización de keywords.
20. No escales SEO programático sin calidad.
21. Mantén staging protegido.
22. Revisa logs, 404, redirects y performance después de cada deploy.
23. SEO, campañas y producto deben compartir métricas de negocio.
```

---

# Fuentes de referencia recomendadas

```text
- Google Search Central: SEO Starter Guide.
- Google Search Central: robots.txt.
- Google Search Central: canonical URLs.
- Google Search Central: structured data.
- Google Search Console.
- Google Ads Help: Search campaigns.
- Google Ads Help: responsive search ads.
- Google Tag Platform: Consent Mode.
- Next.js Docs: Metadata and OG Images.
- Next.js Docs: generateMetadata.
- Next.js Docs: robots.txt metadata file.
- Next.js Docs: sitemap metadata file.
- Meta Business Help: Conversions API.
- Meta for Developers: Conversions API.
```
