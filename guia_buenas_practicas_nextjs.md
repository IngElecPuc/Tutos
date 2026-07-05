# Guía de buenas prácticas para Next.js

## Objetivo

Esta guía resume buenas prácticas para construir aplicaciones Next.js modernas con React y TypeScript. Está pensada como continuación natural de una guía de React: parte desde fundamentos, avanza a rutas, datos y formularios, y termina en patrones avanzados con App Router, Server Components, Server Functions, Route Handlers, TanStack Query, Supabase, validaciones, persistencia, overlays, inlays y CRUD de tareas.

Está organizada en tres niveles:

```text
1. Básico: proyecto, estructura, componentes, exports, estilos, imágenes, eventos, promesas, async/await y fetch.
2. Intermedio: App Router, rutas, layouts, Link, navegación, search params, Server/Client Components, formularios y APIs.
3. Avanzado: Server Functions, Route Handlers, caching, revalidación, TanStack Query, Supabase, contratos, transacciones, overlays, inlays, drag and drop, seguridad y arquitectura.
```

La guía usa el App Router porque es el router moderno de Next.js. El Pages Router sigue existiendo, pero no será el foco principal.

---

# Parte I: fundamentos básicos

## 1. Qué es Next.js

Next.js es un framework de React para crear aplicaciones web con:

```text
Routing basado en archivos.
Renderizado en servidor.
Server Components.
Client Components.
Server Functions.
Route Handlers.
Optimización de imágenes.
Layouts persistentes.
Streaming.
Caching.
Soporte TypeScript.
```

React se encarga de construir UI con componentes. Next.js agrega estructura de aplicación, servidor, routing, optimización y patrones full-stack.

---

## 2. Crear un proyecto Next.js

Comando recomendado:

```bash
npx create-next-app@latest my-next-app
cd my-next-app
npm run dev
```

Durante la creación, para proyectos modernos conviene elegir:

```text
TypeScript: sí.
ESLint: sí.
App Router: sí.
src directory: opcional, recomendado para proyectos medianos/grandes.
Tailwind: según el stack del equipo.
Import alias: sí, normalmente @/*.
```

Estructura típica:

```text
my-next-app/
    app/
        layout.tsx
        page.tsx
        globals.css
    public/
        images/
    next.config.ts
    package.json
    tsconfig.json
```

Con `src/`:

```text
my-next-app/
    src/
        app/
            layout.tsx
            page.tsx
            globals.css
        components/
        features/
        lib/
        types/
    public/
```

---

## 3. App Router

El App Router usa carpetas y archivos especiales dentro de `app/`.

```text
app/
    layout.tsx       layout raíz
    page.tsx         página de /
    loading.tsx      UI de carga
    error.tsx        UI de error
    not-found.tsx    UI 404
```

Ejemplo:

```tsx
// app/page.tsx
export default function HomePage() {
    return (
        <main>
            <h1>Inicio</h1>
            <p>Bienvenido a la aplicación.</p>
        </main>
    );
}
```

Ruta generada:

```http
/
```

---

## 4. Páginas y layouts

Una página se define con `page.tsx`.

```tsx
// app/tasks/page.tsx
export default function TasksPage() {
    return <h1>Tareas</h1>;
}
```

Ruta generada:

```http
/tasks
```

Un layout envuelve a las páginas hijas.

```tsx
// app/layout.tsx
import type { ReactNode } from "react";
import "./globals.css";

type RootLayoutProps = {
    children: ReactNode;
};

export default function RootLayout({ children }: RootLayoutProps) {
    return (
        <html lang="es">
            <body>{children}</body>
        </html>
    );
}
```

El layout raíz debe incluir `<html>` y `<body>`.

---

## 5. Componentes en Next.js

Los componentes React funcionan igual, pero debes distinguir si corren en servidor o cliente.

Por defecto, los componentes dentro de `app/` son Server Components.

```tsx
// app/components/AppTitle.tsx
export function AppTitle() {
    return <h1>Task Manager</h1>;
}
```

Un componente interactivo necesita `'use client'`.

```tsx
// app/components/Counter.tsx
"use client";

import { useState } from "react";

export function Counter() {
    const [count, setCount] = useState(0);

    return (
        <button type="button" onClick={() => setCount(count + 1)}>
            Contador: {count}
        </button>
    );
}
```

Regla práctica:

```text
Server Component por defecto.
Client Component solo cuando necesitas estado, eventos, hooks de cliente o APIs del navegador.
```

---

## 6. Server Components

Un Server Component se renderiza en el servidor. Puede hacer `await` directamente.

```tsx
type Task = {
    id: string;
    title: string;
};

async function getTasks(): Promise<Task[]> {
    const response = await fetch("https://api.example.com/tasks", {
        cache: "no-store",
    });

    if (!response.ok) {
        throw new Error("Could not fetch tasks");
    }

    return response.json();
}

export default async function TasksPage() {
    const tasks = await getTasks();

    return (
        <ul>
            {tasks.map((task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

Ventajas:

```text
No envían JavaScript innecesario al cliente.
Pueden leer datos directamente del servidor.
Pueden usar secretos del servidor si no se filtran al cliente.
Mejoran rendimiento inicial.
```

Limitaciones:

```text
No pueden usar useState.
No pueden usar useEffect.
No pueden manejar eventos onClick.
No pueden acceder a window, document o localStorage.
```

---

## 7. Client Components

Un Client Component se ejecuta en el navegador después de hidratarse.

Usa `'use client'` al inicio del archivo.

```tsx
"use client";

import { useState } from "react";

export function TaskSearchInput() {
    const [query, setQuery] = useState("");

    return (
        <input
            value={query}
            onChange={(event) => setQuery(event.currentTarget.value)}
            placeholder="Buscar tareas"
        />
    );
}
```

Usa Client Components para:

```text
useState.
useEffect.
useMemo.
useRef.
Eventos.
APIs del navegador.
Drag and drop.
Interacciones locales.
TanStack Query en cliente.
```

Evita convertir toda la app en cliente si solo una parte necesita interacción.

---

## 8. Composición servidor-cliente

Patrón recomendado:

```text
Server Component:
- Obtiene datos.
- Define estructura.
- Renderiza partes estáticas.

Client Component:
- Maneja interacción.
- Maneja estado local.
- Recibe props serializables.
```

Ejemplo:

```tsx
// app/tasks/page.tsx
import { TaskListClient } from "./TaskListClient";

export default async function TasksPage() {
    const tasks = await getTasks();

    return (
        <main>
            <h1>Tareas</h1>
            <TaskListClient initialTasks={tasks} />
        </main>
    );
}
```

```tsx
// app/tasks/TaskListClient.tsx
"use client";

import { useState } from "react";

type Task = {
    id: string;
    title: string;
    completed: boolean;
};

type TaskListClientProps = {
    initialTasks: Task[];
};

export function TaskListClient({ initialTasks }: TaskListClientProps) {
    const [tasks, setTasks] = useState(initialTasks);

    return (
        <ul>
            {tasks.map((task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

Regla:

```text
Pasa datos simples y serializables desde servidor a cliente.
No pases funciones arbitrarias desde Server Components a Client Components, salvo Server Functions en patrones soportados.
```

---

## 9. Exports e imports

Next.js usa módulos igual que React.

### Named export

```tsx
export function Button() {
    return <button type="button">Aceptar</button>;
}
```

```tsx
import { Button } from "@/components/Button";
```

### Default export

Los archivos `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx` y `not-found.tsx` normalmente exportan por defecto.

```tsx
export default function Page() {
    return <h1>Inicio</h1>;
}
```

Recomendación:

```text
Usa default export para convenciones de Next.js.
Usa named exports para componentes reutilizables.
```

---

## 10. Alias de imports

Con alias:

```tsx
import { Button } from "@/components/Button";
import { formatDate } from "@/lib/formatDate";
```

Esto evita rutas relativas largas:

```tsx
import { Button } from "../../../../components/Button";
```

Ejemplo en `tsconfig.json`:

```json
{
    "compilerOptions": {
        "baseUrl": ".",
        "paths": {
            "@/*": ["./src/*"]
        }
    }
}
```

Si no usas `src/`, el alias puede apuntar a `./*`.

---

## 11. Estilos en Next.js

Opciones comunes:

```text
CSS global.
CSS Modules.
Tailwind CSS.
Sass.
CSS-in-JS compatible con SSR.
```

CSS global:

```tsx
// app/layout.tsx
import "./globals.css";
```

CSS Modules:

```tsx
import styles from "./TaskCard.module.css";

export function TaskCard() {
    return <article className={styles.card}>Tarea</article>;
}
```

```css
.card {
    border: 1px solid #ddd;
    border-radius: 0.75rem;
    padding: 1rem;
}
```

Regla:

```text
Usa CSS global para tokens, reset, layout base.
Usa CSS Modules o componentes para estilos locales.
```

---

## 12. `type CSSProperties`

Igual que en React, sirve para estilos inline tipados.

```tsx
import type { CSSProperties } from "react";

type ProgressProps = {
    value: number;
};

export function Progress({ value }: ProgressProps) {
    const style = {
        "--progress-value": `${value}%`,
    } as CSSProperties;

    return <div className="progress" style={style} />;
}
```

```css
.progress {
    width: var(--progress-value);
}
```

Úsalo para variables dinámicas, no como reemplazo completo del CSS.

---

## 13. Imágenes locales con `next/image`

Next.js incluye optimización de imágenes con `next/image`.

```tsx
import Image from "next/image";
import heroImage from "@/assets/hero.jpg";

export function Hero() {
    return (
        <Image
            src={heroImage}
            alt="Panel de tareas"
            priority
        />
    );
}
```

Para imágenes en `public/`:

```text
public/
    images/
        logo.png
```

```tsx
import Image from "next/image";

export function Logo() {
    return (
        <Image
            src="/images/logo.png"
            alt="Logo"
            width={160}
            height={48}
        />
    );
}
```

Buenas prácticas:

```text
Usa alt descriptivo.
Define width y height cuando uses rutas públicas.
Usa priority solo para imágenes críticas above the fold.
Evita imágenes enormes sin necesidad.
```

---

## 14. Lazy loading de imágenes

`next/image` aplica lazy loading por defecto para imágenes no prioritarias.

```tsx
<Image
    src="/images/task-cover.jpg"
    alt="Portada de tarea"
    width={800}
    height={450}
/>
```

Usa `priority` para una imagen principal crítica:

```tsx
<Image
    src="/images/hero.jpg"
    alt="Dashboard principal"
    width={1200}
    height={700}
    priority
/>
```

Regla:

```text
No marques todas las imágenes como priority.
Solo las que afectan el Largest Contentful Paint.
```

---

## 15. Lazy loading de componentes

Next.js permite carga dinámica.

```tsx
import dynamic from "next/dynamic";

const HeavyChart = dynamic(() => import("@/components/HeavyChart"), {
    loading: () => <p>Cargando gráfico...</p>,
});

export default function ReportsPage() {
    return <HeavyChart />;
}
```

Para desactivar SSR en un componente que depende de `window`:

```tsx
const ClientOnlyMap = dynamic(() => import("@/components/ClientOnlyMap"), {
    ssr: false,
});
```

Úsalo con criterio:

```text
Gráficos pesados.
Mapas.
Editores ricos.
Modales poco usados.
Librerías solo del navegador.
```

---

## 16. Eventos en Next.js

Los eventos funcionan en Client Components.

```tsx
"use client";

import type { MouseEvent } from "react";

export function SaveButton() {
    function handleClick(event: MouseEvent<HTMLButtonElement>) {
        event.preventDefault();
        console.log("Guardar");
    }

    return (
        <button type="button" onClick={handleClick}>
            Guardar
        </button>
    );
}
```

No puedes usar `onClick` en un Server Component.

```tsx
// Incorrecto si este archivo no tiene "use client"
export function Button() {
    return (
        <button type="button" onClick={() => console.log("click")}>
            Click
        </button>
    );
}
```

---

## 17. `type MouseEvent`, `type DragEvent` y eventos comunes

```tsx
"use client";

import type {
    ChangeEvent,
    DragEvent,
    FormEvent,
    KeyboardEvent,
    MouseEvent,
} from "react";

export function EventExamples() {
    function handleChange(event: ChangeEvent<HTMLInputElement>) {
        console.log(event.currentTarget.value);
    }

    function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();
    }

    function handleKeyDown(event: KeyboardEvent<HTMLInputElement>) {
        if (event.key === "Enter") {
            console.log("Enter");
        }
    }

    function handleClick(event: MouseEvent<HTMLButtonElement>) {
        console.log(event.currentTarget.dataset.action);
    }

    function handleDrop(event: DragEvent<HTMLDivElement>) {
        event.preventDefault();
        console.log(event.dataTransfer.files);
    }

    return (
        <form onSubmit={handleSubmit}>
            <input onChange={handleChange} onKeyDown={handleKeyDown} />
            <button data-action="save" type="button" onClick={handleClick}>
                Guardar
            </button>
            <div onDragOver={(event) => event.preventDefault()} onDrop={handleDrop}>
                Drop
            </div>
        </form>
    );
}
```

---

## 18. `useState`, `useEffect`, `useMemo` y `useRef`

Estos hooks requieren Client Components.

```tsx
"use client";

import { useEffect, useMemo, useRef, useState } from "react";

type Task = {
    id: string;
    title: string;
    completed: boolean;
};

export function TaskFilter({ tasks }: { tasks: Task[] }) {
    const [query, setQuery] = useState("");
    const inputRef = useRef<HTMLInputElement | null>(null);

    const filteredTasks = useMemo(() => {
        return tasks.filter((task) =>
            task.title.toLowerCase().includes(query.toLowerCase()),
        );
    }, [tasks, query]);

    useEffect(() => {
        inputRef.current?.focus();
    }, []);

    return (
        <section>
            <input
                ref={inputRef}
                value={query}
                onChange={(event) => setQuery(event.currentTarget.value)}
            />

            <ul>
                {filteredTasks.map((task) => (
                    <li key={task.id}>{task.title}</li>
                ))}
            </ul>
        </section>
    );
}
```

Reglas:

```text
useState: estado local.
useEffect: sincronización con APIs del navegador o sistemas externos.
useMemo: cálculos derivados costosos.
useRef: DOM o valores mutables que no disparan render.
```

---

## 19. Listas con `map`

```tsx
type Task = {
    id: string;
    title: string;
};

export function TaskList({ tasks }: { tasks: Task[] }) {
    return (
        <ul>
            {tasks.map((task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

Buenas prácticas:

```text
Usa key estable.
No uses index como key si la lista puede cambiar.
Renderiza estados vacíos.
```

```tsx
if (tasks.length === 0) {
    return <p>No hay tareas.</p>;
}
```

---

## 20. Promesas, `async` y `await`

En Server Components puedes usar `await` directamente.

```tsx
export default async function Page() {
    const profile = await getProfile();

    return <h1>{profile.name}</h1>;
}
```

En Client Components no puedes hacer el componente principal `async`.

```tsx
// No recomendado / inválido para Client Component
"use client";

export default async function Page() {
    const data = await getData();

    return <div>{data.name}</div>;
}
```

En cliente, usa eventos, hooks, TanStack Query o efectos.

```tsx
"use client";

import { useState } from "react";

export function LoadButton() {
    const [message, setMessage] = useState("");

    async function handleClick() {
        const response = await fetch("/api/message");
        const data = await response.json();

        setMessage(data.message);
    }

    return (
        <button type="button" onClick={handleClick}>
            {message || "Cargar"}
        </button>
    );
}
```

---

# Parte II: routing, navegación y layouts

## 21. Routing basado en archivos

Cada carpeta representa un segmento de ruta. Un archivo `page.tsx` hace pública la ruta.

```text
app/
    page.tsx                  /
    tasks/
        page.tsx              /tasks
    tasks/
        new/
            page.tsx          /tasks/new
```

Ejemplo:

```tsx
// app/tasks/new/page.tsx
export default function NewTaskPage() {
    return <h1>Nueva tarea</h1>;
}
```

---

## 22. Rutas dinámicas

Usa carpetas con corchetes.

```text
app/
    tasks/
        [taskId]/
            page.tsx
```

Ruta:

```http
/tasks/abc123
```

En Server Component, los params llegan como prop.

```tsx
type TaskDetailPageProps = {
    params: Promise<{
        taskId: string;
    }>;
};

export default async function TaskDetailPage({
    params,
}: TaskDetailPageProps) {
    const { taskId } = await params;
    const task = await getTask(taskId);

    return <h1>{task.title}</h1>;
}
```

En Client Component usa `useParams`.

```tsx
"use client";

import { useParams } from "next/navigation";

export function TaskClientInfo() {
    const params = useParams<{ taskId: string }>();

    return <p>ID: {params.taskId}</p>;
}
```

---

## 23. Rutas catch-all

Para rutas con varios segmentos:

```text
app/
    docs/
        [...slug]/
            page.tsx
```

Ejemplos:

```http
/docs/react
/docs/react/hooks/use-effect
```

```tsx
type DocsPageProps = {
    params: Promise<{
        slug: string[];
    }>;
};

export default async function DocsPage({ params }: DocsPageProps) {
    const { slug } = await params;

    return <p>{slug.join(" / ")}</p>;
}
```

Opcional catch-all:

```text
app/
    docs/
        [[...slug]]/
            page.tsx
```

Esto también captura `/docs`.

---

## 24. `Link`

Usa `next/link` para navegación interna.

```tsx
import Link from "next/link";

export function MainNav() {
    return (
        <nav>
            <Link href="/">Inicio</Link>
            <Link href="/tasks">Tareas</Link>
            <Link href="/settings">Configuración</Link>
        </nav>
    );
}
```

Ventajas:

```text
Navegación cliente-servidor optimizada.
Prefetch cuando corresponde.
Menos recargas completas.
Mejor integración con App Router.
```

Usa `<a>` normal para sitios externos:

```tsx
<a href="https://nextjs.org" target="_blank" rel="noreferrer">
    Next.js
</a>
```

---

## 25. `useRouter`

En Next.js no se usa `useNavigate`; el equivalente es `useRouter` desde `next/navigation`.

```tsx
"use client";

import { useRouter } from "next/navigation";

export function CreateTaskButton() {
    const router = useRouter();

    function handleClick() {
        router.push("/tasks/new");
    }

    return (
        <button type="button" onClick={handleClick}>
            Nueva tarea
        </button>
    );
}
```

Métodos comunes:

```text
router.push("/tasks")
router.replace("/login")
router.back()
router.forward()
router.refresh()
router.prefetch("/tasks")
```

Regla:

```text
Usa Link para navegación declarativa.
Usa useRouter para navegación programática.
```

---

## 26. `usePathname`

```tsx
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";

export function NavLink({
    href,
    label,
}: {
    href: string;
    label: string;
}) {
    const pathname = usePathname();
    const isActive = pathname === href;

    return (
        <Link
            href={href}
            className={isActive ? "nav-link nav-link--active" : "nav-link"}
        >
            {label}
        </Link>
    );
}
```

Útil para:

```text
Links activos.
Breadcrumbs.
UI dependiente de ruta.
```

---

## 27. Search params en Server Components

La página recibe `searchParams`.

```tsx
type TasksPageProps = {
    searchParams: Promise<{
        query?: string;
        status?: string;
        page?: string;
    }>;
};

export default async function TasksPage({
    searchParams,
}: TasksPageProps) {
    const params = await searchParams;

    const query = params.query ?? "";
    const status = params.status ?? "all";
    const page = Number(params.page ?? "1");

    const tasks = await getTasks({
        query,
        status,
        page,
    });

    return <TaskList tasks={tasks} />;
}
```

Reglas:

```text
Valida search params.
Convierte tipos explícitamente.
No guardes secretos en la URL.
```

---

## 28. `useSearchParams`

En Client Components:

```tsx
"use client";

import { usePathname, useRouter, useSearchParams } from "next/navigation";

export function TaskSearchBox() {
    const router = useRouter();
    const pathname = usePathname();
    const searchParams = useSearchParams();

    const query = searchParams.get("query") ?? "";

    function updateQuery(value: string) {
        const params = new URLSearchParams(searchParams);

        if (value.trim()) {
            params.set("query", value);
        } else {
            params.delete("query");
        }

        router.replace(`${pathname}?${params.toString()}`);
    }

    return (
        <input
            value={query}
            onChange={(event) => updateQuery(event.currentTarget.value)}
            placeholder="Buscar tareas"
        />
    );
}
```

Buenas prácticas:

```text
Guarda búsqueda, filtros, orden y paginación en la URL.
Usa debounce si cada cambio dispara fetch.
Usa router.replace para no llenar historial con cada tecla.
```

---

## 29. Layouts persistentes

```text
app/
    dashboard/
        layout.tsx
        page.tsx
        tasks/
            page.tsx
        settings/
            page.tsx
```

```tsx
// app/dashboard/layout.tsx
import type { ReactNode } from "react";
import Link from "next/link";

type DashboardLayoutProps = {
    children: ReactNode;
};

export default function DashboardLayout({
    children,
}: DashboardLayoutProps) {
    return (
        <div className="dashboard-shell">
            <aside>
                <Link href="/dashboard">Resumen</Link>
                <Link href="/dashboard/tasks">Tareas</Link>
                <Link href="/dashboard/settings">Configuración</Link>
            </aside>

            <main>{children}</main>
        </div>
    );
}
```

Este layout persiste mientras navegas dentro de `/dashboard`.

Ventajas:

```text
Sidebar persistente.
Menos repetición.
Mejor experiencia en dashboards.
Estado visual más estable.
```

---

## 30. Route groups

Los route groups organizan rutas sin afectar la URL.

```text
app/
    (marketing)/
        page.tsx              /
        pricing/
            page.tsx          /pricing
    (dashboard)/
        dashboard/
            page.tsx          /dashboard
```

Sirven para:

```text
Separar layouts.
Organizar features.
Separar marketing, auth y dashboard.
```

Ejemplo:

```text
app/
    (public)/
        layout.tsx
        page.tsx
    (app)/
        layout.tsx
        dashboard/
            page.tsx
```

---

## 31. Loading UI

`loading.tsx` define una UI de carga para una ruta.

```tsx
// app/tasks/loading.tsx
export default function LoadingTasks() {
    return <p>Cargando tareas...</p>;
}
```

Útil con Server Components y streaming.

Recomendación:

```text
Usa skeletons para pantallas conocidas.
Usa mensajes simples para secciones pequeñas.
```

---

## 32. Error UI

`error.tsx` debe ser Client Component.

```tsx
// app/tasks/error.tsx
"use client";

export default function TasksError({
    error,
    reset,
}: {
    error: Error;
    reset: () => void;
}) {
    return (
        <section role="alert">
            <h2>No se pudieron cargar las tareas.</h2>
            <p>{error.message}</p>
            <button type="button" onClick={reset}>
                Reintentar
            </button>
        </section>
    );
}
```

Regla:

```text
Muestra al usuario un error claro.
No expongas detalles internos sensibles en producción.
```

---

## 33. 404 con `not-found.tsx`

```tsx
// app/not-found.tsx
import Link from "next/link";

export default function NotFoundPage() {
    return (
        <main>
            <h1>Página no encontrada</h1>
            <Link href="/">Volver al inicio</Link>
        </main>
    );
}
```

Para disparar 404 desde una página:

```tsx
import { notFound } from "next/navigation";

export default async function TaskDetailPage({
    params,
}: {
    params: Promise<{ taskId: string }>;
}) {
    const { taskId } = await params;
    const task = await getTask(taskId);

    if (!task) {
        notFound();
    }

    return <h1>{task.title}</h1>;
}
```

---

## 34. Metadata

Next.js permite definir metadata por página o layout.

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
    title: "Tareas",
    description: "Administra tus tareas.",
};

export default function TasksPage() {
    return <h1>Tareas</h1>;
}
```

Metadata dinámica:

```tsx
export async function generateMetadata({
    params,
}: {
    params: Promise<{ taskId: string }>;
}): Promise<Metadata> {
    const { taskId } = await params;
    const task = await getTask(taskId);

    return {
        title: task.title,
        description: task.description ?? "Detalle de tarea",
    };
}
```

---

# Parte III: datos, fetch y APIs

## 35. `fetch` en Server Components

Next.js extiende `fetch` en el servidor con opciones de caching y revalidación.

```tsx
async function getTasks() {
    const response = await fetch("https://api.example.com/tasks", {
        next: {
            revalidate: 60,
        },
    });

    if (!response.ok) {
        throw new Error("Could not fetch tasks");
    }

    return response.json();
}
```

Opciones comunes:

```text
cache: "force-cache"      usa cache cuando corresponde.
cache: "no-store"         siempre dinámico.
next.revalidate: 60       revalida cada 60 segundos.
next.tags: ["tasks"]      permite invalidar por tag.
```

Ejemplo dinámico:

```ts
await fetch("https://api.example.com/tasks", {
    cache: "no-store",
});
```

---

## 36. Cuándo usar fetch en servidor o cliente

Servidor:

```text
Datos iniciales de página.
Datos SEO.
Datos que usan secretos.
Datos que no requieren interacción inmediata.
Consultas directas a DB/ORM.
```

Cliente:

```text
Datos que cambian por interacción local.
Búsqueda en vivo.
Infinite scroll.
Mutaciones con feedback inmediato.
Datos dependientes de APIs del navegador.
```

Regla:

```text
Prefiere servidor para carga inicial.
Usa cliente cuando la interacción lo exige.
```

---

## 37. Capa de API interna

Aunque puedes hacer fetch directamente, conviene centralizar lógica.

```text
src/
    lib/
        api/
            http.ts
            tasks.ts
```

```ts
// src/lib/api/http.ts
export class ApiError extends Error {
    status: number;
    payload: unknown;

    constructor(status: number, payload: unknown) {
        super(`API error: ${status}`);
        this.status = status;
        this.payload = payload;
    }
}

export async function apiFetch<T>(
    url: string,
    options: RequestInit = {},
): Promise<T> {
    const response = await fetch(url, options);

    if (!response.ok) {
        let payload: unknown = null;

        try {
            payload = await response.json();
        } catch {
            payload = null;
        }

        throw new ApiError(response.status, payload);
    }

    if (response.status === 204) {
        return undefined as T;
    }

    return response.json() as Promise<T>;
}
```

---

## 38. Route Handlers

Route Handlers permiten crear endpoints dentro de `app/`.

```text
app/
    api/
        tasks/
            route.ts
```

```ts
// app/api/tasks/route.ts
import { NextResponse } from "next/server";

export async function GET() {
    const tasks = await db.task.findMany();

    return NextResponse.json(tasks);
}

export async function POST(request: Request) {
    const body = await request.json();

    const task = await db.task.create({
        data: {
            title: body.title,
        },
    });

    return NextResponse.json(task, {
        status: 201,
    });
}
```

Métodos soportados normalmente:

```text
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
```

Usa Route Handlers para:

```text
APIs internas.
Webhooks.
BFF.
Integraciones.
Descargas.
Endpoints no-HTML.
```

---

## 39. Backend for Frontend

Next.js puede actuar como BFF: un backend diseñado para tu frontend.

```text
Frontend -> Route Handler / Server Function -> API externa / DB
```

Ventajas:

```text
Oculta secretos del navegador.
Adapta datos al frontend.
Centraliza autenticación.
Reduce acoplamiento con APIs externas.
Permite validar y auditar.
```

Ejemplo:

```ts
// app/api/profile/route.ts
import { cookies } from "next/headers";
import { NextResponse } from "next/server";

export async function GET() {
    const cookieStore = await cookies();
    const session = cookieStore.get("__Host-session");

    if (!session) {
        return NextResponse.json(
            { detail: "Unauthorized" },
            { status: 401 },
        );
    }

    const profile = await getProfileFromBackend(session.value);

    return NextResponse.json(profile);
}
```

---

## 40. `cookies()` y `headers()`

En servidor puedes leer cookies y headers.

```tsx
import { cookies, headers } from "next/headers";

export default async function Page() {
    const cookieStore = await cookies();
    const headersList = await headers();

    const theme = cookieStore.get("theme")?.value ?? "light";
    const userAgent = headersList.get("user-agent");

    return (
        <main data-theme={theme}>
            <p>{userAgent}</p>
        </main>
    );
}
```

Para escribir cookies, usa Server Functions o Route Handlers.

```ts
"use server";

import { cookies } from "next/headers";

export async function setTheme(theme: "light" | "dark") {
    const cookieStore = await cookies();

    cookieStore.set("theme", theme, {
        httpOnly: true,
        secure: true,
        sameSite: "lax",
        path: "/",
    });
}
```

---

## 41. Variables de entorno

`.env.local`:

```env
DATABASE_URL=postgresql://...
NEXT_PUBLIC_APP_NAME=Task Manager
```

Uso en servidor:

```ts
const databaseUrl = process.env.DATABASE_URL;
```

Uso en cliente:

```ts
const appName = process.env.NEXT_PUBLIC_APP_NAME;
```

Reglas:

```text
Solo las variables con NEXT_PUBLIC_ se exponen al navegador.
Nunca pongas secretos en variables NEXT_PUBLIC_.
No commitees .env.local.
```

---

# Parte IV: formularios, acciones y mutaciones

## 42. Formularios HTML básicos

```tsx
export function TaskForm() {
    return (
        <form>
            <label htmlFor="title">Título</label>
            <input id="title" name="title" required maxLength={120} />

            <button type="submit">Crear</button>
        </form>
    );
}
```

Ventajas:

```text
Accesibilidad nativa.
Funciona con teclado.
Permite Server Functions.
Menos JavaScript si el caso es simple.
```

---

## 43. Server Functions con `'use server'`

Una Server Function corre en el servidor.

```ts
// app/tasks/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

export async function createTask(formData: FormData) {
    const title = String(formData.get("title") ?? "").trim();

    if (!title) {
        return {
            error: "El título es obligatorio.",
        };
    }

    const task = await db.task.create({
        data: {
            title,
        },
    });

    revalidatePath("/tasks");
    redirect(`/tasks/${task.id}`);
}
```

Uso en formulario:

```tsx
import { createTask } from "./actions";

export function TaskCreateForm() {
    return (
        <form action={createTask}>
            <label htmlFor="title">Título</label>
            <input id="title" name="title" required />

            <button type="submit">Crear</button>
        </form>
    );
}
```

---

## 44. `useActionState`

Para mostrar estado de validación desde una Server Function:

```ts
// app/tasks/actions.ts
"use server";

export type CreateTaskState = {
    error?: string;
    success?: boolean;
};

export async function createTask(
    _previousState: CreateTaskState,
    formData: FormData,
): Promise<CreateTaskState> {
    const title = String(formData.get("title") ?? "").trim();

    if (!title) {
        return {
            error: "El título es obligatorio.",
        };
    }

    await db.task.create({
        data: {
            title,
        },
    });

    return {
        success: true,
    };
}
```

Cliente:

```tsx
"use client";

import { useActionState } from "react";
import { createTask, type CreateTaskState } from "./actions";

const initialState: CreateTaskState = {};

export function TaskCreateForm() {
    const [state, formAction, isPending] = useActionState(
        createTask,
        initialState,
    );

    return (
        <form action={formAction}>
            <input name="title" />

            {state.error && <p role="alert">{state.error}</p>}

            <button type="submit" disabled={isPending}>
                {isPending ? "Guardando..." : "Crear"}
            </button>
        </form>
    );
}
```

---

## 45. `useFormStatus`

`useFormStatus` sirve para componentes dentro de un formulario.

```tsx
"use client";

import { useFormStatus } from "react-dom";

export function SubmitButton() {
    const { pending } = useFormStatus();

    return (
        <button type="submit" disabled={pending}>
            {pending ? "Guardando..." : "Guardar"}
        </button>
    );
}
```

Uso:

```tsx
<form action={createTask}>
    <input name="title" />
    <SubmitButton />
</form>
```

---

## 46. Validaciones de formularios

Valida en servidor aunque también valides en cliente.

Con Zod:

```ts
import { z } from "zod";

export const createTaskSchema = z.object({
    title: z.string().trim().min(1).max(120),
    description: z.string().trim().max(1000).optional(),
});

export type CreateTaskInput = z.infer<typeof createTaskSchema>;
```

Server Function:

```ts
"use server";

import { createTaskSchema } from "@/features/tasks/task.schema";

export async function createTaskAction(formData: FormData) {
    const rawInput = {
        title: formData.get("title"),
        description: formData.get("description"),
    };

    const result = createTaskSchema.safeParse(rawInput);

    if (!result.success) {
        return {
            ok: false,
            errors: result.error.flatten().fieldErrors,
        };
    }

    await db.task.create({
        data: result.data,
    });

    revalidatePath("/tasks");

    return {
        ok: true,
    };
}
```

Regla:

```text
Validación frontend = experiencia.
Validación servidor = seguridad e integridad.
```

---

## 47. Revalidación

Después de mutar datos, invalida lo que quedó obsoleto.

```ts
import { revalidatePath, revalidateTag } from "next/cache";

revalidatePath("/tasks");
revalidateTag("tasks");
```

Uso con fetch tag:

```ts
await fetch("https://api.example.com/tasks", {
    next: {
        tags: ["tasks"],
    },
});
```

Después de crear:

```ts
revalidateTag("tasks");
```

Criterio:

```text
revalidatePath: una ruta específica.
revalidateTag: datos compartidos por varias rutas.
```

---

## 48. `redirect`

```ts
import { redirect } from "next/navigation";

export async function createTaskAction(formData: FormData) {
    const task = await createTask(formData);

    redirect(`/tasks/${task.id}`);
}
```

Criterio:

```text
Usa redirect en servidor después de una acción completada.
Usa useRouter en cliente cuando la navegación nace de interacción local.
```

---

## 49. Transacciones

Cuando una mutación afecta varias tablas, usa transacción en la base de datos.

Ejemplo conceptual con ORM:

```ts
"use server";

export async function completeTaskAction(taskId: string) {
    await db.$transaction(async (tx) => {
        await tx.task.update({
            where: {
                id: taskId,
            },
            data: {
                completed: true,
            },
        });

        await tx.auditLog.create({
            data: {
                entity: "task",
                entityId: taskId,
                action: "completed",
            },
        });
    });

    revalidatePath("/tasks");
}
```

Regla:

```text
Si las operaciones deben completarse juntas, usa transacción.
No simules consistencia crítica solo con cambios de UI.
```

---

# Parte V: CRUD de tareas con Next.js

## 50. Contratos de dominio

```ts
// src/features/tasks/task.types.ts
export type Task = {
    id: string;
    title: string;
    description: string | null;
    completed: boolean;
    createdAt: string;
    updatedAt: string;
};

export type CreateTaskInput = {
    title: string;
    description?: string;
};

export type UpdateTaskInput = {
    title?: string;
    description?: string;
    completed?: boolean;
};
```

---

## 51. Modelo conceptual de datos

```sql
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL CHECK (char_length(title) <= 120),
    description TEXT,
    completed BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Regla:

```text
El frontend valida para UX.
La base de datos protege integridad.
```

---

## 52. Listar tareas en Server Component

```tsx
// app/tasks/page.tsx
import { getTasks } from "@/features/tasks/tasks.data";

export default async function TasksPage() {
    const tasks = await getTasks();

    return (
        <main>
            <h1>Tareas</h1>

            {tasks.length === 0 ? (
                <p>No hay tareas.</p>
            ) : (
                <ul>
                    {tasks.map((task) => (
                        <li key={task.id}>{task.title}</li>
                    ))}
                </ul>
            )}
        </main>
    );
}
```

---

## 53. Crear tarea con Server Function

```ts
// app/tasks/actions.ts
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";
import { createTaskSchema } from "@/features/tasks/task.schema";

export async function createTaskAction(formData: FormData) {
    const result = createTaskSchema.safeParse({
        title: formData.get("title"),
        description: formData.get("description"),
    });

    if (!result.success) {
        return {
            ok: false,
            errors: result.error.flatten().fieldErrors,
        };
    }

    const task = await db.task.create({
        data: result.data,
    });

    revalidatePath("/tasks");
    redirect(`/tasks/${task.id}`);
}
```

---

## 54. Editar tarea

```ts
"use server";

import { revalidatePath } from "next/cache";

export async function updateTaskAction(
    taskId: string,
    formData: FormData,
) {
    const title = String(formData.get("title") ?? "").trim();

    if (!title) {
        return {
            ok: false,
            error: "El título es obligatorio.",
        };
    }

    await db.task.update({
        where: {
            id: taskId,
        },
        data: {
            title,
        },
    });

    revalidatePath("/tasks");
    revalidatePath(`/tasks/${taskId}`);

    return {
        ok: true,
    };
}
```

Uso con `bind`:

```tsx
<form action={updateTaskAction.bind(null, task.id)}>
    <input name="title" defaultValue={task.title} />
    <button type="submit">Guardar</button>
</form>
```

---

## 55. Eliminar tarea

```ts
"use server";

import { revalidatePath } from "next/cache";

export async function deleteTaskAction(taskId: string) {
    await db.task.delete({
        where: {
            id: taskId,
        },
    });

    revalidatePath("/tasks");
}
```

Formulario:

```tsx
<form action={deleteTaskAction.bind(null, task.id)}>
    <button type="submit">Eliminar</button>
</form>
```

Para acciones destructivas, considera confirmación en cliente:

```tsx
"use client";

export function DeleteTaskButton({
    action,
}: {
    action: () => void;
}) {
    function handleClick(event: React.MouseEvent<HTMLButtonElement>) {
        const confirmed = window.confirm("¿Eliminar esta tarea?");

        if (!confirmed) {
            event.preventDefault();
        }
    }

    return (
        <button type="submit" onClick={handleClick}>
            Eliminar
        </button>
    );
}
```

---

## 56. Buscar tareas con search params

```tsx
// app/tasks/page.tsx
type TasksPageProps = {
    searchParams: Promise<{
        query?: string;
    }>;
};

export default async function TasksPage({
    searchParams,
}: TasksPageProps) {
    const { query = "" } = await searchParams;
    const tasks = await searchTasks(query);

    return (
        <main>
            <TaskSearchBox />
            <TaskList tasks={tasks} />
        </main>
    );
}
```

Cliente para modificar URL:

```tsx
"use client";

import { usePathname, useRouter, useSearchParams } from "next/navigation";
import { useTransition } from "react";

export function TaskSearchBox() {
    const router = useRouter();
    const pathname = usePathname();
    const searchParams = useSearchParams();
    const [isPending, startTransition] = useTransition();

    const query = searchParams.get("query") ?? "";

    function handleChange(value: string) {
        const params = new URLSearchParams(searchParams);

        if (value) {
            params.set("query", value);
        } else {
            params.delete("query");
        }

        startTransition(() => {
            router.replace(`${pathname}?${params.toString()}`);
        });
    }

    return (
        <label>
            Buscar
            <input
                value={query}
                onChange={(event) => handleChange(event.currentTarget.value)}
            />
            {isPending && <span>Actualizando...</span>}
        </label>
    );
}
```

---

# Parte VI: TanStack Query en Next.js

## 57. Cuándo usar TanStack Query

En Next.js no siempre necesitas TanStack Query. Server Components y Server Functions cubren muchos casos.

Úsalo cuando necesites:

```text
Interacción cliente intensa.
Búsqueda en vivo.
Paginación infinita.
Mutaciones optimistas.
Cache cliente.
Refetch al enfocar ventana.
Estado remoto muy dinámico.
APIs externas desde cliente.
```

No lo uses por inercia si la página se resuelve bien con Server Components.

---

## 58. Configurar QueryClientProvider

```tsx
// src/app/providers.tsx
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactNode, useState } from "react";

export function Providers({ children }: { children: ReactNode }) {
    const [queryClient] = useState(() => new QueryClient());

    return (
        <QueryClientProvider client={queryClient}>
            {children}
        </QueryClientProvider>
    );
}
```

En layout:

```tsx
// app/layout.tsx
import { Providers } from "./providers";

export default function RootLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="es">
            <body>
                <Providers>{children}</Providers>
            </body>
        </html>
    );
}
```

---

## 59. `useQuery`

```tsx
"use client";

import { useQuery } from "@tanstack/react-query";

async function fetchTasks() {
    const response = await fetch("/api/tasks");

    if (!response.ok) {
        throw new Error("Could not fetch tasks");
    }

    return response.json();
}

export function TaskListClient() {
    const tasksQuery = useQuery({
        queryKey: ["tasks"],
        queryFn: fetchTasks,
    });

    if (tasksQuery.isLoading) {
        return <p>Cargando...</p>;
    }

    if (tasksQuery.isError) {
        return <p role="alert">Error al cargar tareas.</p>;
    }

    return (
        <ul>
            {tasksQuery.data.map((task: Task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

---

## 60. Query keys

```ts
export const taskKeys = {
    all: ["tasks"] as const,
    lists: () => [...taskKeys.all, "list"] as const,
    list: (filters: { query?: string }) =>
        [...taskKeys.lists(), filters] as const,
    details: () => [...taskKeys.all, "detail"] as const,
    detail: (taskId: string) =>
        [...taskKeys.details(), taskId] as const,
};
```

---

## 61. `useMutation` e `invalidateQueries`

```tsx
"use client";

import { useMutation, useQueryClient } from "@tanstack/react-query";
import { taskKeys } from "./taskKeys";

async function createTask(input: CreateTaskInput) {
    const response = await fetch("/api/tasks", {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
        },
        body: JSON.stringify(input),
    });

    if (!response.ok) {
        throw new Error("Could not create task");
    }

    return response.json();
}

export function CreateTaskButton() {
    const queryClient = useQueryClient();

    const createTaskMutation = useMutation({
        mutationFn: createTask,
        onSuccess: () => {
            queryClient.invalidateQueries({
                queryKey: taskKeys.lists(),
            });
        },
    });

    return (
        <button
            type="button"
            disabled={createTaskMutation.isPending}
            onClick={() => createTaskMutation.mutate({ title: "Nueva tarea" })}
        >
            Crear
        </button>
    );
}
```

---

## 62. TanStack Query + Server Components

Puedes prefetchear en servidor e hidratar en cliente, pero no lo hagas sin necesidad.

Uso recomendado:

```text
Server Component:
- Prefetch de datos importantes.
- Render inicial rápido.

Client Component:
- useQuery continúa gestionando cache e interacciones.
```

Estructura conceptual:

```tsx
// app/tasks/page.tsx
import {
    HydrationBoundary,
    QueryClient,
    dehydrate,
} from "@tanstack/react-query";

export default async function TasksPage() {
    const queryClient = new QueryClient();

    await queryClient.prefetchQuery({
        queryKey: taskKeys.list({}),
        queryFn: () => getTasks(),
    });

    return (
        <HydrationBoundary state={dehydrate(queryClient)}>
            <TaskListClient />
        </HydrationBoundary>
    );
}
```

Regla:

```text
Si Server Components ya resuelven el caso, no agregues TanStack Query.
Si necesitas cache cliente e interacción compleja, sí tiene sentido.
```

---

# Parte VII: Supabase con Next.js

## 63. Supabase en Next.js

Supabase puede usarse como:

```text
Base de datos PostgreSQL.
Auth.
Storage.
Realtime.
Backend gestionado.
```

En Next.js, la integración moderna suele usar helpers para SSR y cookies.

---

## 64. Variables de entorno Supabase

```env
NEXT_PUBLIC_SUPABASE_URL=https://project.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=your-publishable-key
```

Reglas:

```text
La publishable key puede estar en cliente.
La service_role key nunca debe estar en cliente.
Activa Row Level Security para datos sensibles.
```

---

## 65. Cliente de Supabase para servidor

Ejemplo conceptual:

```ts
// src/lib/supabase/server.ts
import { cookies } from "next/headers";
import { createServerClient } from "@supabase/ssr";

export async function createClient() {
    const cookieStore = await cookies();

    return createServerClient(
        process.env.NEXT_PUBLIC_SUPABASE_URL!,
        process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!,
        {
            cookies: {
                getAll() {
                    return cookieStore.getAll();
                },
                setAll(cookiesToSet) {
                    cookiesToSet.forEach(({ name, value, options }) => {
                        cookieStore.set(name, value, options);
                    });
                },
            },
        },
    );
}
```

---

## 66. Leer tareas desde Supabase en Server Component

```tsx
import { createClient } from "@/lib/supabase/server";

export default async function TasksPage() {
    const supabase = await createClient();

    const { data, error } = await supabase
        .from("tasks")
        .select("id,title,description,completed,created_at")
        .order("created_at", {
            ascending: false,
        });

    if (error) {
        throw error;
    }

    return (
        <ul>
            {data.map((task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

---

## 67. Crear tarea con Supabase y Server Function

```ts
"use server";

import { revalidatePath } from "next/cache";
import { createClient } from "@/lib/supabase/server";

export async function createSupabaseTaskAction(formData: FormData) {
    const title = String(formData.get("title") ?? "").trim();

    if (!title) {
        return {
            error: "El título es obligatorio.",
        };
    }

    const supabase = await createClient();

    const { error } = await supabase
        .from("tasks")
        .insert({
            title,
        });

    if (error) {
        return {
            error: "No se pudo crear la tarea.",
        };
    }

    revalidatePath("/tasks");

    return {
        ok: true,
    };
}
```

---

## 68. Supabase Auth

Reglas prácticas:

```text
Usa cookies para SSR.
Configura redirect URLs.
Protege rutas en servidor.
No confíes solo en UI cliente.
Usa RLS.
No expongas service_role.
```

Ejemplo de protección en página:

```tsx
import { redirect } from "next/navigation";
import { createClient } from "@/lib/supabase/server";

export default async function PrivatePage() {
    const supabase = await createClient();

    const {
        data: { user },
    } = await supabase.auth.getUser();

    if (!user) {
        redirect("/login");
    }

    return <h1>Privado</h1>;
}
```

---

## 69. RLS

Con Supabase, la autorización debe vivir en la base cuando los datos son sensibles.

Ejemplo conceptual:

```sql
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can read their own tasks"
ON tasks
FOR SELECT
USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own tasks"
ON tasks
FOR INSERT
WITH CHECK (auth.uid() = user_id);
```

Regla:

```text
El frontend no es autoridad.
Server Functions ayudan, pero RLS protege incluso si alguien llama la API directamente.
```

---

# Parte VIII: overlays, inlays y rutas avanzadas

## 70. Overlay común con portal

```tsx
"use client";

import { createPortal } from "react-dom";
import type { ReactNode } from "react";

type ModalProps = {
    title: string;
    children: ReactNode;
    onClose: () => void;
};

export function Modal({ title, children, onClose }: ModalProps) {
    return createPortal(
        <div className="modal-backdrop">
            <section role="dialog" aria-modal="true" className="modal">
                <header>
                    <h2>{title}</h2>
                    <button type="button" onClick={onClose}>
                        Cerrar
                    </button>
                </header>

                {children}
            </section>
        </div>,
        document.body,
    );
}
```

Útil para:

```text
Confirmaciones.
Edición rápida.
Filtros avanzados.
Detalles temporales.
```

---

## 71. Overlay con rutas interceptadas

Next.js permite crear modales que también son rutas. Esto se hace con Parallel Routes e Intercepting Routes.

Uso típico:

```text
/tasks                 lista
/tasks/123             detalle completo
/tasks + modal /123    detalle como overlay sobre lista
```

Estructura conceptual:

```text
app/
    tasks/
        layout.tsx
        page.tsx
        [taskId]/
            page.tsx
        @modal/
            default.tsx
            (.)[taskId]/
                page.tsx
```

Ventajas:

```text
El modal tiene URL compartible.
Al refrescar puede abrir como página o mantener contexto según diseño.
Back cierra el modal.
Forward puede reabrirlo.
```

Úsalo para patrones tipo:

```text
Instagram modal.
Detalle sobre lista.
Edición contextual.
Galerías.
```

---

## 72. Inlay

Un inlay es un panel dentro del layout, no un modal.

```tsx
type TaskInlayProps = {
    task: Task | null;
};

export function TaskInlay({ task }: TaskInlayProps) {
    if (!task) {
        return (
            <aside className="task-inlay">
                Selecciona una tarea.
            </aside>
        );
    }

    return (
        <aside className="task-inlay">
            <h2>{task.title}</h2>
            <p>{task.completed ? "Completada" : "Pendiente"}</p>
        </aside>
    );
}
```

Úsalo cuando:

```text
El usuario necesita contexto permanente.
La lista y el detalle deben verse a la vez.
El flujo no debe interrumpirse.
```

---

## 73. Parallel Routes para paneles

Parallel Routes usan slots.

```text
app/
    dashboard/
        layout.tsx
        @main/
            page.tsx
        @inspector/
            page.tsx
```

Layout:

```tsx
export default function DashboardLayout({
    main,
    inspector,
}: {
    main: React.ReactNode;
    inspector: React.ReactNode;
}) {
    return (
        <div className="dashboard-grid">
            <section>{main}</section>
            <aside>{inspector}</aside>
        </div>
    );
}
```

Útil para:

```text
Dashboards.
Inlays.
Paneles simultáneos.
Vistas maestro-detalle.
```

---

# Parte IX: arquitectura avanzada

## 74. Organización por features

```text
src/
    app/
        layout.tsx
        page.tsx
        tasks/
            page.tsx
            actions.ts
    features/
        tasks/
            components/
                TaskCard.tsx
                TaskForm.tsx
            data/
                tasks.data.ts
            queries/
                taskKeys.ts
                taskQueries.ts
            schemas/
                task.schema.ts
            types/
                task.types.ts
    components/
        ui/
            Button.tsx
            Modal.tsx
    lib/
        db.ts
        env.ts
        supabase/
            server.ts
            client.ts
```

Reglas:

```text
app/ define rutas.
features/ contiene dominio.
components/ui contiene componentes genéricos.
lib/ contiene infraestructura.
```

---

## 75. Contratos de componentes

```tsx
type TaskCardProps = {
    task: Task;
    variant?: "compact" | "detailed";
    onToggle?: (taskId: string) => void;
    onDelete?: (taskId: string) => void;
};

export function TaskCard({
    task,
    variant = "compact",
    onToggle,
    onDelete,
}: TaskCardProps) {
    return (
        <article className={`task-card task-card--${variant}`}>
            <h2>{task.title}</h2>

            {onToggle && (
                <button type="button" onClick={() => onToggle(task.id)}>
                    Cambiar estado
                </button>
            )}

            {onDelete && (
                <button type="button" onClick={() => onDelete(task.id)}>
                    Eliminar
                </button>
            )}
        </article>
    );
}
```

Contrato documentado:

```text
task: dato de lectura.
variant: apariencia.
onToggle: callback opcional de interacción cliente.
onDelete: callback opcional de interacción cliente.
```

---

## 76. Contratos entre Server y Client Components

Props de Client Components deben ser serializables.

Bueno:

```tsx
<TaskClient
    task={{
        id: "1",
        title: "Estudiar Next.js",
        completed: false,
    }}
/>
```

Evita pasar:

```text
Instancias de clases.
Conexiones de DB.
Funciones comunes.
Objetos no serializables.
Secretos.
```

Regla:

```text
El borde Server -> Client debe tratarse como contrato público.
```

---

## 77. Validación runtime

TypeScript no valida respuestas en ejecución.

```ts
import { z } from "zod";

export const taskSchema = z.object({
    id: z.string(),
    title: z.string(),
    description: z.string().nullable(),
    completed: z.boolean(),
    createdAt: z.string(),
    updatedAt: z.string(),
});

export const tasksSchema = z.array(taskSchema);
```

Uso:

```ts
const json = await response.json();
const tasks = tasksSchema.parse(json);
```

Útil cuando:

```text
Consumes APIs externas.
Migras backend.
Los datos son críticos.
Hay errores frecuentes de contrato.
```

---

## 78. Hooks personalizados

Los hooks personalizados solo corren en cliente si usan hooks de React cliente.

```tsx
"use client";

import { useEffect, useState } from "react";

export function useDebouncedValue<T>(value: T, delayMs: number): T {
    const [debouncedValue, setDebouncedValue] = useState(value);

    useEffect(() => {
        const timeoutId = window.setTimeout(() => {
            setDebouncedValue(value);
        }, delayMs);

        return () => {
            window.clearTimeout(timeoutId);
        };
    }, [value, delayMs]);

    return debouncedValue;
}
```

Regla:

```text
Si el hook usa useState/useEffect/window/localStorage, debe vivir en un módulo cliente.
```

---

## 79. Persistencia local

```tsx
"use client";

import { useEffect, useState } from "react";

export function ThemeToggle() {
    const [theme, setTheme] = useState("light");

    useEffect(() => {
        const storedTheme = localStorage.getItem("theme");

        if (storedTheme) {
            setTheme(storedTheme);
        }
    }, []);

    useEffect(() => {
        localStorage.setItem("theme", theme);
        document.documentElement.dataset.theme = theme;
    }, [theme]);

    return (
        <button
            type="button"
            onClick={() => setTheme(theme === "light" ? "dark" : "light")}
        >
            Cambiar tema
        </button>
    );
}
```

Advertencia:

```text
localStorage solo existe en navegador.
No lo leas en Server Components.
No guardes secretos.
```

Para preferencias no sensibles que deben estar disponibles en servidor, considera cookies.

---

## 80. Drag and drop

Debe ser Client Component.

```tsx
"use client";

import type { DragEvent } from "react";

type TaskBoardProps = {
    tasks: Task[];
    onReorder: (sourceId: string, targetId: string) => void;
};

export function TaskBoard({ tasks, onReorder }: TaskBoardProps) {
    function handleDragStart(
        event: DragEvent<HTMLLIElement>,
        taskId: string,
    ) {
        event.dataTransfer.setData("text/task-id", taskId);
        event.dataTransfer.effectAllowed = "move";
    }

    function handleDragOver(event: DragEvent<HTMLLIElement>) {
        event.preventDefault();
    }

    function handleDrop(
        event: DragEvent<HTMLLIElement>,
        targetTaskId: string,
    ) {
        event.preventDefault();

        const sourceTaskId = event.dataTransfer.getData("text/task-id");

        if (!sourceTaskId || sourceTaskId === targetTaskId) {
            return;
        }

        onReorder(sourceTaskId, targetTaskId);
    }

    return (
        <ul>
            {tasks.map((task) => (
                <li
                    key={task.id}
                    draggable
                    onDragStart={(event) => handleDragStart(event, task.id)}
                    onDragOver={handleDragOver}
                    onDrop={(event) => handleDrop(event, task.id)}
                >
                    {task.title}
                </li>
            ))}
        </ul>
    );
}
```

Para guardar reordenamiento:

```text
1. Cambia UI local.
2. Llama Server Function o API.
3. Si falla, revierte o recarga.
4. Revalida lista.
```

---

## 81. Snippets útiles

Snippet para página:

```json
{
    "Next page": {
        "prefix": "nxpage",
        "body": [
            "export default function ${1:PageName}() {",
            "    return (",
            "        <main>",
            "            <h1>${2:Title}</h1>",
            "        </main>",
            "    );",
            "}"
        ]
    }
}
```

Snippet para Client Component:

```json
{
    "Next client component": {
        "prefix": "nxclient",
        "body": [
            "\"use client\";",
            "",
            "type ${1:ComponentName}Props = {",
            "    $2",
            "};",
            "",
            "export function ${1:ComponentName}({}: ${1:ComponentName}Props) {",
            "    return (",
            "        <div>",
            "            $0",
            "        </div>",
            "    );",
            "}"
        ]
    }
}
```

Snippet para Server Function:

```json
{
    "Next server action": {
        "prefix": "nxaction",
        "body": [
            "\"use server\";",
            "",
            "export async function ${1:actionName}(formData: FormData) {",
            "    $0",
            "}"
        ]
    }
}
```

---

# Parte X: seguridad en Next.js

## 82. Backend como autoridad

Aunque Next.js permite lógica full-stack, la regla sigue siendo:

```text
El servidor valida.
El servidor autoriza.
El servidor protege secretos.
La base de datos protege integridad.
El frontend mejora experiencia.
```

No hagas esto:

```tsx
if (user.role === "admin") {
    return <DeleteButton />;
}
```

y luego dejes un endpoint sin autorización.

Cada Server Function y Route Handler sensible debe verificar:

```text
Autenticación.
Autorización.
Ownership.
Tenant.
Validación de datos.
```

---

## 83. Server Functions y seguridad

Server Functions corren en servidor, pero no son mágicamente seguras.

Checklist:

```text
[ ] Validar entrada.
[ ] Verificar usuario.
[ ] Verificar permisos.
[ ] No confiar en IDs del cliente.
[ ] Usar transacciones si corresponde.
[ ] Revalidar datos.
[ ] No filtrar errores internos.
```

Ejemplo:

```ts
"use server";

export async function deleteTaskAction(taskId: string) {
    const user = await requireUser();
    const task = await db.task.findUnique({
        where: {
            id: taskId,
        },
    });

    if (!task || task.userId !== user.id) {
        throw new Error("Forbidden");
    }

    await db.task.delete({
        where: {
            id: taskId,
        },
    });

    revalidatePath("/tasks");
}
```

---

## 84. Cookies seguras

```ts
"use server";

import { cookies } from "next/headers";

export async function setSessionCookie(sessionId: string) {
    const cookieStore = await cookies();

    cookieStore.set("__Host-session", sessionId, {
        httpOnly: true,
        secure: true,
        sameSite: "lax",
        path: "/",
        maxAge: 60 * 60,
    });
}
```

Reglas:

```text
HttpOnly para sesión.
Secure en producción.
SameSite Lax o Strict según flujo.
No guardar tokens sensibles en localStorage si puedes evitarlo.
```

---

## 85. Proxy, antes Middleware

En Next.js moderno, la convención `middleware` fue renombrada a `proxy`. Sirve para ejecutar lógica antes de que una request complete su ciclo.

Ejemplo conceptual:

```ts
// proxy.ts
import { NextResponse, type NextRequest } from "next/server";

export function proxy(request: NextRequest) {
    const session = request.cookies.get("__Host-session");

    if (!session && request.nextUrl.pathname.startsWith("/dashboard")) {
        return NextResponse.redirect(new URL("/login", request.url));
    }

    return NextResponse.next();
}

export const config = {
    matcher: ["/dashboard/:path*"],
};
```

Advertencia:

```text
Proxy ayuda a redirigir o filtrar temprano.
No reemplaza autorización dentro de Server Functions, Route Handlers o consultas.
```

---

## 86. CSP y headers

Configura headers en `next.config.ts` o infraestructura.

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
    async headers() {
        return [
            {
                source: "/(.*)",
                headers: [
                    {
                        key: "X-Content-Type-Options",
                        value: "nosniff",
                    },
                    {
                        key: "Referrer-Policy",
                        value: "strict-origin-when-cross-origin",
                    },
                    {
                        key: "Content-Security-Policy",
                        value: "default-src 'self'; object-src 'none'; frame-ancestors 'none'",
                    },
                ],
            },
        ];
    },
};

export default nextConfig;
```

Ajusta CSP según scripts, estilos, imágenes y proveedores reales.

---

# Parte XI: checklist final

## 87. Checklist de buenas prácticas Next.js

```text
Estructura
[ ] App Router usado como base.
[ ] Rutas organizadas por carpetas.
[ ] Layouts persistentes donde corresponde.
[ ] Route groups para separar áreas.
[ ] loading.tsx, error.tsx y not-found.tsx en rutas importantes.

Server/Client Components
[ ] Server Components por defecto.
[ ] "use client" solo donde se necesita.
[ ] Props serializables hacia cliente.
[ ] Secretos nunca llegan al cliente.
[ ] Hooks cliente no se usan en servidor.

Routing
[ ] Link para navegación interna.
[ ] useRouter solo para navegación programática.
[ ] Rutas dinámicas validadas.
[ ] searchParams usados para filtros compartibles.
[ ] useSearchParams solo en Client Components.
[ ] Layouts anidados claros.

Datos
[ ] fetch en servidor cuando conviene.
[ ] cache/no-store/revalidate definidos con intención.
[ ] Errores de fetch manejados.
[ ] Route Handlers usados para BFF/API.
[ ] Datos externos validados si son críticos.

Formularios y mutaciones
[ ] Server Functions para formularios simples o seguros.
[ ] useActionState para estado de formularios.
[ ] useFormStatus para pending state.
[ ] Validación en servidor.
[ ] revalidatePath o revalidateTag después de mutaciones.
[ ] redirect usado después de acciones completadas cuando corresponde.

TanStack Query
[ ] Solo usado cuando aporta cache cliente/interacción.
[ ] QueryClientProvider en Client Component.
[ ] query keys consistentes.
[ ] useQuery para lectura cliente.
[ ] useMutation para escritura cliente.
[ ] invalidateQueries después de cambios.

Supabase
[ ] Variables NEXT_PUBLIC correctas.
[ ] service_role nunca en cliente.
[ ] Cliente SSR configurado con cookies.
[ ] RLS activado en tablas sensibles.
[ ] Auth validada en servidor.

Seguridad
[ ] Backend como autoridad.
[ ] Server Functions verifican auth y permisos.
[ ] Route Handlers validan entrada.
[ ] Cookies HttpOnly/Secure/SameSite para sesiones.
[ ] Proxy no reemplaza autorización real.
[ ] CSP y headers revisados.
[ ] .env no commiteado.

UX
[ ] next/image usado para imágenes relevantes.
[ ] priority solo para imagen crítica.
[ ] dynamic import para componentes pesados.
[ ] Overlay accesible.
[ ] Inlay para contexto persistente.
[ ] Drag and drop considera accesibilidad.
```

---

## 88. Resumen de reglas principales

```text
1. Next.js no reemplaza React; lo estructura y lo lleva al servidor.
2. Usa Server Components por defecto.
3. Usa Client Components solo para interacción.
4. Usa Link para navegar y useRouter para navegación programática.
5. En App Router, usa next/navigation.
6. Guarda filtros y paginación en search params.
7. Usa layouts persistentes para shells de aplicación.
8. Usa Server Functions para mutaciones de formularios.
9. Usa Route Handlers para APIs, BFF y webhooks.
10. Define cache, no-store o revalidate con intención.
11. Revalida después de mutar.
12. Usa TanStack Query cuando necesites estado remoto cliente.
13. Usa Supabase con RLS y sin exponer claves privilegiadas.
14. Valida datos en servidor.
15. El frontend no es autoridad de permisos.
```

---

# Fuentes de referencia recomendadas

```text
- Next.js Docs: App Router.
- Next.js Docs: Layouts and Pages.
- Next.js Docs: Linking and Navigating.
- Next.js Docs: Server and Client Components.
- Next.js Docs: Fetching Data.
- Next.js Docs: Mutating Data.
- Next.js Docs: Route Handlers.
- Next.js Docs: Image Optimization.
- Next.js Docs: Forms.
- Next.js Docs: Authentication.
- Next.js Docs: Data Security.
- React Docs: Hooks, useActionState, useFormStatus.
- TanStack Query Docs: Advanced Server Rendering, useQuery, useMutation.
- Supabase Docs: Next.js quickstart, SSR Auth, RLS.
```
