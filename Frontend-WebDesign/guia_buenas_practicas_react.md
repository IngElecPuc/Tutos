# Guía de buenas prácticas para React

## Objetivo

Esta guía resume buenas prácticas para construir aplicaciones React modernas, mantenibles y seguras. Está orientada a desarrollo frontend con TypeScript, consumo de APIs, rutas, formularios, estado, persistencia y manejo de datos remotos.

Está organizada en tres niveles:

```text
1. Básico: componentes, JSX, props, exports, eventos, estilos, imágenes, listas, promises, async/await y fetch.
2. Intermedio: hooks, useEffect, formularios, validaciones, React Router, rutas anidadas/dinámicas, layouts, APIs y CRUD.
3. Avanzado: TanStack Query, useQuery, useMutation, invalidateQueries, hooks personalizados, Supabase, persistencia, contratos, overlays, drag and drop y patrones escalables.
```

La guía usa TypeScript porque ayuda a definir contratos claros entre componentes, APIs y formularios.

---

# Parte I: fundamentos básicos

## 1. Qué es React

React es una librería para construir interfaces de usuario a partir de componentes.

Un componente es una función que retorna UI:

```tsx
function Greeting() {
    return <h1>Hola React</h1>;
}

export default Greeting;
```

React permite dividir una pantalla completa en piezas pequeñas:

```text
App
├── Header
├── Sidebar
├── TaskList
│   ├── TaskCard
│   └── TaskCard
└── Footer
```

Regla práctica:

```text
Un componente debería tener una responsabilidad clara.
Si un componente hace demasiado, probablemente debe dividirse.
```

---

## 2. Crear un proyecto React con Vite

Vite es una opción común para crear aplicaciones React modernas.

```bash
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install
npm run dev
```

Estructura inicial típica:

```text
my-react-app/
    src/
        App.tsx
        main.tsx
        index.css
    package.json
    tsconfig.json
    vite.config.ts
```

Punto de entrada:

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import App from "./App";

createRoot(document.getElementById("root")!).render(
    <StrictMode>
        <App />
    </StrictMode>,
);
```

---

## 3. JSX

JSX permite escribir estructura visual dentro de JavaScript/TypeScript.

```tsx
function App() {
    const appName = "Task Manager";

    return (
        <main>
            <h1>{appName}</h1>
            <p>Administra tus tareas.</p>
        </main>
    );
}
```

Reglas importantes:

```text
Usa className en vez de class.
Usa htmlFor en vez de for.
Todo componente debe retornar un solo nodo raíz.
Las expresiones JS van entre llaves.
```

Ejemplo:

```tsx
<label htmlFor="task-title">Título</label>
<input id="task-title" className="field" />
```

---

## 4. Componentes

Un componente debería ser una función pura respecto a sus props: para las mismas props, debería producir la misma UI.

```tsx
type TaskCardProps = {
    title: string;
    completed: boolean;
};

function TaskCard({ title, completed }: TaskCardProps) {
    return (
        <article className="task-card">
            <h2>{title}</h2>
            <p>{completed ? "Completada" : "Pendiente"}</p>
        </article>
    );
}
```

Uso:

```tsx
<TaskCard title="Estudiar React" completed={false} />
```

Buenas prácticas:

```text
Nombra componentes con PascalCase.
Nombra props con camelCase.
Evita componentes enormes.
Extrae componentes cuando haya repetición o una responsabilidad clara.
```

---

## 5. Props

Las props son datos que un componente recibe desde su padre.

```tsx
type ButtonProps = {
    label: string;
    disabled?: boolean;
    onClick: () => void;
};

function Button({ label, disabled = false, onClick }: ButtonProps) {
    return (
        <button type="button" disabled={disabled} onClick={onClick}>
            {label}
        </button>
    );
}
```

Uso:

```tsx
<Button
    label="Guardar"
    onClick={() => {
        console.log("Guardando");
    }}
/>
```

Regla:

```text
Las props son de solo lectura.
Si necesitas cambiar algo, usa estado o emite un evento hacia el padre.
```

---

## 6. Children

`children` permite pasar contenido interno al componente.

```tsx
import type { ReactNode } from "react";

type CardProps = {
    title: string;
    children: ReactNode;
};

function Card({ title, children }: CardProps) {
    return (
        <section className="card">
            <h2>{title}</h2>
            <div>{children}</div>
        </section>
    );
}
```

Uso:

```tsx
<Card title="Resumen">
    <p>Tienes 3 tareas pendientes.</p>
</Card>
```

---

## 7. Exports e imports

React usa módulos JavaScript.

### 7.1 Export default

```tsx
// components/Header.tsx
function Header() {
    return <header>Mi app</header>;
}

export default Header;
```

Importación:

```tsx
import Header from "./components/Header";
```

Ventaja:

```text
Útil cuando un archivo expone un componente principal.
```

Riesgo:

```text
El import puede cambiar de nombre y reducir consistencia.
```

---

### 7.2 Named export

```tsx
// components/Button.tsx
export function Button() {
    return <button type="button">Aceptar</button>;
}
```

Importación:

```tsx
import { Button } from "./components/Button";
```

Ventaja:

```text
Mejor para autocompletado.
Evita renombres accidentales.
Permite varios exports por archivo.
```

Recomendación práctica:

```text
Usa named exports para componentes reutilizables.
Usa default export en páginas o entrypoints si tu equipo lo prefiere.
Mantén una convención única.
```

---

## 8. Barrel files

Un barrel file centraliza exports.

```tsx
// components/index.ts
export { Button } from "./Button";
export { Card } from "./Card";
export { TaskCard } from "./TaskCard";
```

Uso:

```tsx
import { Button, Card, TaskCard } from "./components";
```

Úsalo con criterio. En proyectos grandes puede ocultar dependencias o crear ciclos si se abusa.

---

## 9. Estilos básicos en React

Opciones comunes:

```text
CSS global.
CSS Modules.
CSS-in-JS.
Tailwind CSS.
Sass.
Estilos inline.
```

Para guías generales, CSS Modules o CSS normal bien organizado suelen ser suficientes.

### 9.1 CSS global

```tsx
import "./App.css";

function App() {
    return <main className="app">Hola</main>;
}
```

```css
.app {
    min-height: 100dvh;
    padding: 2rem;
}
```

### 9.2 CSS Modules

```tsx
import styles from "./TaskCard.module.css";

function TaskCard() {
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

Ventaja:

```text
Reduce choques de nombres entre componentes.
```

---

## 10. `type CSSProperties`

Usa `CSSProperties` cuando necesitas estilos inline tipados.

```tsx
import type { CSSProperties } from "react";

type BoxProps = {
    color: string;
};

function Box({ color }: BoxProps) {
    const style: CSSProperties = {
        backgroundColor: color,
        padding: "1rem",
        borderRadius: "0.75rem",
    };

    return <div style={style}>Caja</div>;
}
```

Recomendación:

```text
Usa estilos inline para valores dinámicos pequeños.
No reemplaces todo tu CSS con style={{ ... }}.
```

Ejemplo útil con variable CSS:

```tsx
import type { CSSProperties } from "react";

type ProgressProps = {
    value: number;
};

function Progress({ value }: ProgressProps) {
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

---

## 11. Eventos en React

React normaliza eventos del navegador. Los handlers usan camelCase.

```tsx
function SaveButton() {
    function handleClick() {
        console.log("Click");
    }

    return (
        <button type="button" onClick={handleClick}>
            Guardar
        </button>
    );
}
```

Convención:

```text
Prop recibida: onSave, onDelete, onSubmit.
Función interna: handleSave, handleDelete, handleSubmit.
```

Ejemplo:

```tsx
type TaskCardProps = {
    onDelete: () => void;
};

function TaskCard({ onDelete }: TaskCardProps) {
    function handleDeleteClick() {
        onDelete();
    }

    return (
        <button type="button" onClick={handleDeleteClick}>
            Eliminar
        </button>
    );
}
```

---

## 12. `type MouseEvent`

```tsx
import type { MouseEvent } from "react";

function ButtonExample() {
    function handleClick(event: MouseEvent<HTMLButtonElement>) {
        event.preventDefault();
        console.log(event.currentTarget.name);
    }

    return (
        <button name="save" type="button" onClick={handleClick}>
            Guardar
        </button>
    );
}
```

Regla:

```text
Usa event.currentTarget cuando quieres el elemento que tiene el handler.
Usa event.target cuando quieres el elemento exacto que disparó el evento.
```

---

## 13. `type DragEvent`

```tsx
import type { DragEvent } from "react";

function DropZone() {
    function handleDragOver(event: DragEvent<HTMLDivElement>) {
        event.preventDefault();
    }

    function handleDrop(event: DragEvent<HTMLDivElement>) {
        event.preventDefault();

        const files = Array.from(event.dataTransfer.files);
        console.log(files);
    }

    return (
        <div
            className="dropzone"
            onDragOver={handleDragOver}
            onDrop={handleDrop}
        >
            Arrastra archivos aquí
        </div>
    );
}
```

Importante:

```text
En dragover normalmente debes llamar event.preventDefault()
para permitir que el drop ocurra.
```

---

## 14. Eventos frecuentes

```tsx
import type {
    ChangeEvent,
    FormEvent,
    KeyboardEvent,
    MouseEvent,
    DragEvent,
} from "react";
```

Ejemplos:

```tsx
function EventExamples() {
    function handleChange(event: ChangeEvent<HTMLInputElement>) {
        console.log(event.currentTarget.value);
    }

    function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();
        console.log("submit");
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
            <div onDrop={handleDrop}>Drop</div>
        </form>
    );
}
```

---

## 15. Estado con `useState`

`useState` guarda estado local de un componente.

```tsx
import { useState } from "react";

function Counter() {
    const [count, setCount] = useState(0);

    return (
        <section>
            <p>Contador: {count}</p>
            <button type="button" onClick={() => setCount(count + 1)}>
                Sumar
            </button>
        </section>
    );
}
```

Si el nuevo estado depende del estado anterior, usa función:

```tsx
setCount((currentCount) => currentCount + 1);
```

Esto evita errores cuando React agrupa actualizaciones.

---

## 16. Estado con tipos

```tsx
type Task = {
    id: string;
    title: string;
    completed: boolean;
};

function TaskList() {
    const [tasks, setTasks] = useState<Task[]>([]);

    function addTask(title: string) {
        const newTask: Task = {
            id: crypto.randomUUID(),
            title,
            completed: false,
        };

        setTasks((currentTasks) => [...currentTasks, newTask]);
    }

    return (
        <button type="button" onClick={() => addTask("Nueva tarea")}>
            Agregar
        </button>
    );
}
```

Evita mutar arrays directamente:

```tsx
// Mala práctica
tasks.push(newTask);
setTasks(tasks);
```

Mejor:

```tsx
setTasks((currentTasks) => [...currentTasks, newTask]);
```

---

## 17. Renderizado condicional

```tsx
function TaskStatus({ completed }: { completed: boolean }) {
    if (completed) {
        return <span>Completada</span>;
    }

    return <span>Pendiente</span>;
}
```

También puedes usar ternario:

```tsx
<p>{completed ? "Completada" : "Pendiente"}</p>
```

Y renderizado condicional:

```tsx
{error && <p className="error">{error}</p>}
```

Cuidado:

```tsx
{tasks.length && <p>Hay tareas</p>}
```

Si `tasks.length` es `0`, React puede renderizar `0`.

Mejor:

```tsx
{tasks.length > 0 && <p>Hay tareas</p>}
```

---

## 18. Listas con `map`

```tsx
type Task = {
    id: string;
    title: string;
};

function TaskList({ tasks }: { tasks: Task[] }) {
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
No uses index como key si la lista se ordena, filtra, inserta o elimina.
Incluye IDs en tus datos.
```

Menos recomendable:

```tsx
{tasks.map((task, index) => (
    <li key={index}>{task.title}</li>
))}
```

---

## 19. Imágenes locales

Con Vite, puedes importar imágenes desde `src`.

```tsx
import logoUrl from "./assets/logo.svg";

function Header() {
    return (
        <img src={logoUrl} alt="Logo de la aplicación" />
    );
}
```

Para imágenes públicas estáticas, puedes usar `public/`.

```text
public/
    images/
        hero.jpg
```

Uso:

```tsx
<img src="/images/hero.jpg" alt="Hero" />
```

Reglas:

```text
Usa import para assets que forman parte del bundle.
Usa public para archivos referenciados por URL fija.
Siempre agrega alt apropiado.
No uses imágenes enormes sin optimizar.
```

---

## 20. Lazy loading de imágenes

El navegador soporta carga diferida:

```tsx
<img
    src={coverUrl}
    alt="Portada del libro"
    loading="lazy"
/>
```

También puedes reservar proporción para evitar saltos visuales:

```tsx
<img
    className="cover"
    src={coverUrl}
    alt="Portada"
    loading="lazy"
/>
```

```css
.cover {
    width: 100%;
    aspect-ratio: 16 / 9;
    object-fit: cover;
}
```

---

## 21. Lazy loading de componentes

Para cargar componentes bajo demanda:

```tsx
import { Suspense, lazy } from "react";

const ReportsPage = lazy(() => import("./pages/ReportsPage"));

function App() {
    return (
        <Suspense fallback={<p>Cargando...</p>}>
            <ReportsPage />
        </Suspense>
    );
}
```

Útil para:

```text
Páginas pesadas.
Módulos de administración.
Gráficos.
Editores ricos.
Rutas poco visitadas.
```

No abuses de lazy loading en componentes pequeños.

---

## 22. Promesas

Una promesa representa una operación asíncrona.

```ts
function wait(ms: number): Promise<void> {
    return new Promise((resolve) => {
        window.setTimeout(resolve, ms);
    });
}
```

Uso con `.then()`:

```ts
wait(1000).then(() => {
    console.log("Pasó un segundo");
});
```

Uso con `async/await`:

```ts
async function run() {
    await wait(1000);
    console.log("Pasó un segundo");
}
```

Regla:

```text
async/await no elimina la asincronía.
Solo hace el código más legible.
```

---

## 23. `async` y `await`

```ts
async function getTasks() {
    const response = await fetch("/api/tasks");

    if (!response.ok) {
        throw new Error("Error loading tasks");
    }

    return response.json();
}
```

Manejo de errores:

```ts
async function loadTasks() {
    try {
        const tasks = await getTasks();
        console.log(tasks);
    } catch (error) {
        console.error(error);
    }
}
```

Reglas:

```text
Usa try/catch para errores esperados.
Valida response.ok.
No asumas que response.json() siempre tendrá la forma correcta.
```

---

## 24. Fetch básico

```ts
type Task = {
    id: string;
    title: string;
    completed: boolean;
};

async function fetchTasks(): Promise<Task[]> {
    const response = await fetch("https://api.example.com/tasks");

    if (!response.ok) {
        throw new Error("Could not fetch tasks");
    }

    const data = await response.json();

    return data as Task[];
}
```

Mejor práctica:

```text
Centraliza llamadas a APIs.
No repitas fetch crudo en todos los componentes.
Valida datos de entrada con Zod, Valibot o una capa propia si el backend no es 100% confiable.
```

---

## 25. Snippets útiles

Los snippets ayudan a crear código repetitivo.

Ejemplo de snippet conceptual para VS Code:

```json
{
    "React component": {
        "prefix": "rfc",
        "body": [
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

Snippets recomendados:

```text
Componente con props.
Hook personalizado.
useQuery.
useMutation.
Formulario con submit.
Route object.
Test básico.
```

---

# Parte II: nivel intermedio

## 26. `useEffect`

`useEffect` sincroniza un componente con un sistema externo.

Sistemas externos típicos:

```text
API remota.
WebSocket.
Intervalo.
Evento global del navegador.
Storage.
Librería externa.
Mapa.
Reproductor.
```

Ejemplo:

```tsx
import { useEffect, useState } from "react";

type Task = {
    id: string;
    title: string;
};

function TasksPage() {
    const [tasks, setTasks] = useState<Task[]>([]);
    const [isLoading, setIsLoading] = useState(false);

    useEffect(() => {
        async function loadTasks() {
            setIsLoading(true);

            try {
                const response = await fetch("/api/tasks");
                const data = await response.json();

                setTasks(data);
            } finally {
                setIsLoading(false);
            }
        }

        loadTasks();
    }, []);

    if (isLoading) {
        return <p>Cargando...</p>;
    }

    return (
        <ul>
            {tasks.map((task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

Reglas:

```text
No llames hooks dentro de if, loops o funciones anidadas.
Incluye dependencias correctas.
Si no sincronizas con un sistema externo, quizá no necesitas useEffect.
Para datos de servidor, TanStack Query suele ser mejor que useEffect manual.
```

---

## 27. Cleanup en `useEffect`

Si registras algo, normalmente debes limpiarlo.

```tsx
import { useEffect, useState } from "react";

function WindowWidth() {
    const [width, setWidth] = useState(window.innerWidth);

    useEffect(() => {
        function handleResize() {
            setWidth(window.innerWidth);
        }

        window.addEventListener("resize", handleResize);

        return () => {
            window.removeEventListener("resize", handleResize);
        };
    }, []);

    return <p>Ancho: {width}</p>;
}
```

Ejemplos que requieren cleanup:

```text
setInterval.
addEventListener.
WebSocket.
Subscriptions.
AbortController.
```

---

## 28. Fetch con AbortController

Evita actualizar estado si una request ya no importa.

```tsx
useEffect(() => {
    const controller = new AbortController();

    async function loadTasks() {
        const response = await fetch("/api/tasks", {
            signal: controller.signal,
        });

        const data = await response.json();
        setTasks(data);
    }

    loadTasks().catch((error) => {
        if (error.name !== "AbortError") {
            console.error(error);
        }
    });

    return () => {
        controller.abort();
    };
}, []);
```

Esto es especialmente útil cuando:

```text
El usuario cambia filtros rápido.
El componente se desmonta.
Hay búsquedas en vivo.
```

---

## 29. `useMemo`

`useMemo` memoriza un cálculo derivado.

```tsx
import { useMemo, useState } from "react";

type Task = {
    id: string;
    title: string;
    completed: boolean;
};

function TaskSummary({ tasks }: { tasks: Task[] }) {
    const completedCount = useMemo(() => {
        return tasks.filter((task) => task.completed).length;
    }, [tasks]);

    return <p>Completadas: {completedCount}</p>;
}
```

Úsalo cuando:

```text
El cálculo es costoso.
El valor derivado se pasa a componentes memoizados.
Quieres evitar recrear estructuras innecesariamente.
```

No lo uses para todo. `useMemo` agrega complejidad.

---

## 30. `useRef`

`useRef` guarda una referencia mutable que no dispara render al cambiar.

Uso típico con DOM:

```tsx
import { useRef } from "react";

function SearchInput() {
    const inputRef = useRef<HTMLInputElement | null>(null);

    function focusInput() {
        inputRef.current?.focus();
    }

    return (
        <section>
            <input ref={inputRef} />
            <button type="button" onClick={focusInput}>
                Enfocar
            </button>
        </section>
    );
}
```

Uso típico para valor mutable:

```tsx
const lastRequestIdRef = useRef<string | null>(null);
```

Regla:

```text
Usa useRef para valores que no afectan directamente el render.
Usa useState para valores que sí deben reflejarse en la UI.
```

---

## 31. Formularios controlados

Un formulario controlado guarda el valor en estado React.

```tsx
import { FormEvent, useState } from "react";

function TaskForm() {
    const [title, setTitle] = useState("");

    function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();

        if (title.trim().length === 0) {
            return;
        }

        console.log({ title });
        setTitle("");
    }

    return (
        <form onSubmit={handleSubmit}>
            <label htmlFor="title">Título</label>
            <input
                id="title"
                value={title}
                onChange={(event) => setTitle(event.currentTarget.value)}
            />

            <button type="submit">Crear</button>
        </form>
    );
}
```

Ventaja:

```text
Control total del valor.
Validación inmediata.
UI dependiente del estado.
```

Desventaja:

```text
Más renders y más código en formularios grandes.
```

---

## 32. Formularios no controlados

Un formulario no controlado obtiene datos desde el DOM al enviar.

```tsx
import { FormEvent } from "react";

function TaskForm() {
    function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();

        const formData = new FormData(event.currentTarget);
        const title = String(formData.get("title") ?? "");

        console.log(title);
    }

    return (
        <form onSubmit={handleSubmit}>
            <label htmlFor="title">Título</label>
            <input id="title" name="title" />

            <button type="submit">Crear</button>
        </form>
    );
}
```

Útil cuando:

```text
El formulario es simple.
No necesitas validar mientras se escribe.
Quieres menos estado local.
```

---

## 33. Validaciones de formularios

Puedes validar manualmente:

```tsx
function validateTaskTitle(title: string): string | null {
    if (title.trim().length === 0) {
        return "El título es obligatorio.";
    }

    if (title.length > 120) {
        return "El título no puede superar 120 caracteres.";
    }

    return null;
}
```

Uso:

```tsx
const error = validateTaskTitle(title);

if (error) {
    setError(error);
    return;
}
```

Para aplicaciones medianas o grandes, considera:

```text
React Hook Form.
Zod.
Valibot.
Yup.
Conform.
```

Ejemplo con Zod:

```ts
import { z } from "zod";

const taskSchema = z.object({
    title: z.string().trim().min(1).max(120),
    description: z.string().max(1000).optional(),
});

type TaskFormValues = z.infer<typeof taskSchema>;
```

Validar:

```ts
const result = taskSchema.safeParse({
    title,
    description,
});

if (!result.success) {
    console.log(result.error.flatten());
}
```

Regla:

```text
Valida en frontend para UX.
Valida en backend para seguridad e integridad.
```

---

## 34. Contratos de datos

Un contrato define la forma esperada de datos entre frontend, backend y componentes.

```ts
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

export type UpdateTaskInput = Partial<CreateTaskInput> & {
    completed?: boolean;
};
```

Buenas prácticas:

```text
Separa tipos de lectura y escritura.
No uses el mismo tipo para todo.
No expongas campos internos del backend si no son necesarios.
Valida datos recibidos de APIs externas.
```

---

## 35. Contratos de componentes

Ejemplo de contrato para un componente `TaskCard`:

```ts
type TaskCardProps = {
    task: Task;
    variant?: "compact" | "detailed";
    onToggle: (taskId: string) => void;
    onDelete: (taskId: string) => void;
};
```

Implementación:

```tsx
function TaskCard({
    task,
    variant = "compact",
    onToggle,
    onDelete,
}: TaskCardProps) {
    return (
        <article className={`task-card task-card--${variant}`}>
            <h2>{task.title}</h2>

            <button type="button" onClick={() => onToggle(task.id)}>
                {task.completed ? "Marcar pendiente" : "Completar"}
            </button>

            <button type="button" onClick={() => onDelete(task.id)}>
                Eliminar
            </button>
        </article>
    );
}
```

Regla:

```text
El componente no debería saber cómo se actualiza la base de datos.
Solo comunica intención mediante callbacks.
```

---

## 36. Consumo de APIs: capa `api`

Centraliza llamadas HTTP.

```text
src/
    api/
        http.ts
        tasks.api.ts
    types/
        task.ts
```

`http.ts`:

```ts
const API_URL = import.meta.env.VITE_API_URL;

export async function apiFetch<T>(
    path: string,
    options: RequestInit = {},
): Promise<T> {
    const response = await fetch(`${API_URL}${path}`, {
        ...options,
        headers: {
            "Content-Type": "application/json",
            ...options.headers,
        },
    });

    if (!response.ok) {
        throw new Error(`API error: ${response.status}`);
    }

    return response.json() as Promise<T>;
}
```

`tasks.api.ts`:

```ts
import { apiFetch } from "./http";
import type { CreateTaskInput, Task, UpdateTaskInput } from "../types/task";

export function getTasks(): Promise<Task[]> {
    return apiFetch<Task[]>("/tasks");
}

export function createTask(input: CreateTaskInput): Promise<Task> {
    return apiFetch<Task>("/tasks", {
        method: "POST",
        body: JSON.stringify(input),
    });
}

export function updateTask(id: string, input: UpdateTaskInput): Promise<Task> {
    return apiFetch<Task>(`/tasks/${id}`, {
        method: "PATCH",
        body: JSON.stringify(input),
    });
}

export function deleteTask(id: string): Promise<void> {
    return apiFetch<void>(`/tasks/${id}`, {
        method: "DELETE",
    });
}
```

---

## 37. Manejo de errores de API

Mejor que lanzar solo `Error`.

```ts
export class ApiError extends Error {
    status: number;
    payload: unknown;

    constructor(message: string, status: number, payload: unknown) {
        super(message);
        this.status = status;
        this.payload = payload;
    }
}
```

Uso:

```ts
export async function apiFetch<T>(
    path: string,
    options: RequestInit = {},
): Promise<T> {
    const response = await fetch(`${API_URL}${path}`, options);

    if (!response.ok) {
        let payload: unknown = null;

        try {
            payload = await response.json();
        } catch {
            payload = null;
        }

        throw new ApiError("API request failed", response.status, payload);
    }

    if (response.status === 204) {
        return undefined as T;
    }

    return response.json() as Promise<T>;
}
```

---

# Parte III: rutas con React Router

## 38. Instalación

```bash
npm install react-router
```

En muchos proyectos existentes también verás `react-router-dom`, especialmente con versiones anteriores. En React Router moderno, la documentación oficial muestra imports desde `react-router`.

---

## 39. Rutas básicas

```tsx
import { BrowserRouter, Route, Routes } from "react-router";

import { HomePage } from "./pages/HomePage";
import { TasksPage } from "./pages/TasksPage";
import { NotFoundPage } from "./pages/NotFoundPage";

export function AppRouter() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<HomePage />} />
                <Route path="/tasks" element={<TasksPage />} />
                <Route path="*" element={<NotFoundPage />} />
            </Routes>
        </BrowserRouter>
    );
}
```

Reglas:

```text
Usa rutas orientadas a recursos.
Define una ruta 404.
Evita duplicar rutas equivalentes.
```

---

## 40. `Link`

`Link` navega sin recargar la página.

```tsx
import { Link } from "react-router";

function Header() {
    return (
        <nav>
            <Link to="/">Inicio</Link>
            <Link to="/tasks">Tareas</Link>
        </nav>
    );
}
```

Evita usar `<a href="/tasks">` para navegación interna de una SPA, porque puede recargar toda la aplicación.

Usa `<a>` para URLs externas:

```tsx
<a href="https://react.dev" target="_blank" rel="noreferrer">
    React Docs
</a>
```

---

## 41. `NavLink`

`NavLink` permite aplicar estilo activo.

```tsx
import { NavLink } from "react-router";

function MainNav() {
    return (
        <nav>
            <NavLink
                to="/tasks"
                className={({ isActive }) =>
                    isActive ? "nav-link nav-link--active" : "nav-link"
                }
            >
                Tareas
            </NavLink>
        </nav>
    );
}
```

---

## 42. `useNavigate`

`useNavigate` permite navegar desde código.

```tsx
import { useNavigate } from "react-router";

function CreateTaskButton() {
    const navigate = useNavigate();

    function handleClick() {
        navigate("/tasks/new");
    }

    return (
        <button type="button" onClick={handleClick}>
            Nueva tarea
        </button>
    );
}
```

Usos típicos:

```text
Después de crear un recurso.
Después de login.
Después de logout.
Al cancelar un formulario.
Al cerrar un flujo.
```

No uses `useNavigate` cuando un simple `Link` es suficiente.

---

## 43. Rutas dinámicas

```tsx
<Route path="/tasks/:taskId" element={<TaskDetailPage />} />
```

Leer parámetro:

```tsx
import { useParams } from "react-router";

function TaskDetailPage() {
    const { taskId } = useParams<{ taskId: string }>();

    return <p>ID de tarea: {taskId}</p>;
}
```

Regla:

```text
Los params de ruta son strings.
Valida y convierte si esperas números u otro formato.
```

---

## 44. Rutas anidadas

Las rutas anidadas permiten compartir layout.

```tsx
import { Outlet } from "react-router";

function TasksLayout() {
    return (
        <section>
            <h1>Tareas</h1>
            <nav>
                <Link to="/tasks">Lista</Link>
                <Link to="/tasks/new">Nueva</Link>
            </nav>

            <Outlet />
        </section>
    );
}
```

Rutas:

```tsx
<Routes>
    <Route path="/tasks" element={<TasksLayout />}>
        <Route index element={<TaskListPage />} />
        <Route path="new" element={<TaskCreatePage />} />
        <Route path=":taskId" element={<TaskDetailPage />} />
        <Route path=":taskId/edit" element={<TaskEditPage />} />
    </Route>
</Routes>
```

`Outlet` marca dónde se renderiza la ruta hija.

---

## 45. Layouts persistentes

Un layout persistente no se desmonta al cambiar rutas hijas.

Ejemplo:

```tsx
function AppLayout() {
    return (
        <div className="app-shell">
            <Sidebar />
            <main>
                <Outlet />
            </main>
        </div>
    );
}
```

Rutas:

```tsx
<Routes>
    <Route element={<AppLayout />}>
        <Route path="/" element={<HomePage />} />
        <Route path="/tasks" element={<TaskListPage />} />
        <Route path="/settings" element={<SettingsPage />} />
    </Route>
</Routes>
```

Ventajas:

```text
Mantiene sidebar, header o filtros globales.
Evita repetir estructura.
Preserva parte del estado visual.
```

---

## 46. Search params

Los search params son útiles para filtros, búsqueda, paginación y ordenamiento.

URL:

```http
/tasks?query=react&status=pending&page=2
```

Uso:

```tsx
import { useSearchParams } from "react-router";

function TaskSearchPage() {
    const [searchParams, setSearchParams] = useSearchParams();

    const query = searchParams.get("query") ?? "";
    const status = searchParams.get("status") ?? "all";

    function updateQuery(value: string) {
        setSearchParams((currentParams) => {
            const nextParams = new URLSearchParams(currentParams);

            if (value.trim()) {
                nextParams.set("query", value);
            } else {
                nextParams.delete("query");
            }

            return nextParams;
        });
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
Guarda en la URL filtros compartibles.
No guardes secretos en search params.
Valida valores recibidos desde la URL.
```

---

## 47. Rutas protegidas

Ejemplo simple:

```tsx
import { Navigate, Outlet } from "react-router";

function ProtectedRoute({ isAuthenticated }: { isAuthenticated: boolean }) {
    if (!isAuthenticated) {
        return <Navigate to="/login" replace />;
    }

    return <Outlet />;
}
```

Uso:

```tsx
<Route element={<ProtectedRoute isAuthenticated={isAuthenticated} />}>
    <Route path="/tasks" element={<TaskListPage />} />
    <Route path="/settings" element={<SettingsPage />} />
</Route>
```

Advertencia:

```text
Una ruta protegida en frontend mejora UX.
No reemplaza autorización backend.
El backend debe rechazar requests no autorizadas.
```

---

## 48. Overlay e inlay

### 48.1 Overlay

Un overlay aparece encima de la pantalla: modal, drawer, popover.

```tsx
import { createPortal } from "react-dom";
import type { ReactNode } from "react";

type ModalProps = {
    title: string;
    children: ReactNode;
    onClose: () => void;
};

function Modal({ title, children, onClose }: ModalProps) {
    return createPortal(
        <div className="modal-backdrop" role="presentation">
            <section className="modal" role="dialog" aria-modal="true">
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

Uso:

```tsx
{isOpen && (
    <Modal title="Editar tarea" onClose={() => setIsOpen(false)}>
        <TaskForm />
    </Modal>
)}
```

### 48.2 Inlay

Un inlay se muestra dentro del flujo de la página: panel lateral interno, detalle embebido, inspector.

```tsx
function TaskInlay({ task }: { task: Task | null }) {
    if (!task) {
        return <aside className="inlay">Selecciona una tarea.</aside>;
    }

    return (
        <aside className="inlay">
            <h2>{task.title}</h2>
            <p>{task.completed ? "Completada" : "Pendiente"}</p>
        </aside>
    );
}
```

Criterio:

```text
Usa overlay para interrupciones o edición focalizada.
Usa inlay para contexto persistente junto a la lista.
```

---

# Parte IV: CRUD de tareas sin librerías externas

## 49. Tipos del dominio

```ts
export type Task = {
    id: string;
    title: string;
    description: string | null;
    completed: boolean;
    createdAt: string;
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

## 50. API de tareas

```ts
const API_URL = import.meta.env.VITE_API_URL;

async function request<T>(
    path: string,
    options: RequestInit = {},
): Promise<T> {
    const response = await fetch(`${API_URL}${path}`, {
        ...options,
        headers: {
            "Content-Type": "application/json",
            ...options.headers,
        },
    });

    if (!response.ok) {
        throw new Error(`Request failed: ${response.status}`);
    }

    if (response.status === 204) {
        return undefined as T;
    }

    return response.json() as Promise<T>;
}

export function listTasks(query?: string): Promise<Task[]> {
    const params = new URLSearchParams();

    if (query) {
        params.set("query", query);
    }

    const suffix = params.toString() ? `?${params.toString()}` : "";

    return request<Task[]>(`/tasks${suffix}`);
}

export function getTask(taskId: string): Promise<Task> {
    return request<Task>(`/tasks/${taskId}`);
}

export function createTask(input: CreateTaskInput): Promise<Task> {
    return request<Task>("/tasks", {
        method: "POST",
        body: JSON.stringify(input),
    });
}

export function updateTask(
    taskId: string,
    input: UpdateTaskInput,
): Promise<Task> {
    return request<Task>(`/tasks/${taskId}`, {
        method: "PATCH",
        body: JSON.stringify(input),
    });
}

export function removeTask(taskId: string): Promise<void> {
    return request<void>(`/tasks/${taskId}`, {
        method: "DELETE",
    });
}
```

---

## 51. Listar tareas con `useEffect`

```tsx
import { useEffect, useState } from "react";
import { listTasks } from "../api/tasks";
import type { Task } from "../types/task";

export function TaskListPage() {
    const [tasks, setTasks] = useState<Task[]>([]);
    const [error, setError] = useState<string | null>(null);
    const [isLoading, setIsLoading] = useState(true);

    useEffect(() => {
        async function loadTasks() {
            try {
                setIsLoading(true);
                setError(null);

                const data = await listTasks();
                setTasks(data);
            } catch {
                setError("No se pudieron cargar las tareas.");
            } finally {
                setIsLoading(false);
            }
        }

        loadTasks();
    }, []);

    if (isLoading) {
        return <p>Cargando...</p>;
    }

    if (error) {
        return <p role="alert">{error}</p>;
    }

    return (
        <ul>
            {tasks.map((task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

Esto funciona, pero en apps reales TanStack Query suele resolver mejor:

```text
Cache.
Refetch.
Estados de carga.
Reintentos.
Invalidaciones.
Mutaciones.
Sincronización con foco de ventana.
```

---

## 52. Crear tarea

```tsx
import { FormEvent, useState } from "react";
import { createTask } from "../api/tasks";

export function TaskCreatePage() {
    const [title, setTitle] = useState("");
    const [isSaving, setIsSaving] = useState(false);
    const [error, setError] = useState<string | null>(null);

    async function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();

        if (!title.trim()) {
            setError("El título es obligatorio.");
            return;
        }

        try {
            setIsSaving(true);
            setError(null);

            await createTask({
                title,
            });

            setTitle("");
        } catch {
            setError("No se pudo crear la tarea.");
        } finally {
            setIsSaving(false);
        }
    }

    return (
        <form onSubmit={handleSubmit}>
            <input
                value={title}
                onChange={(event) => setTitle(event.currentTarget.value)}
                placeholder="Título"
            />

            {error && <p role="alert">{error}</p>}

            <button type="submit" disabled={isSaving}>
                {isSaving ? "Guardando..." : "Crear"}
            </button>
        </form>
    );
}
```

---

## 53. Búsqueda de tareas

Con estado local:

```tsx
import { useEffect, useState } from "react";

export function TaskSearchPage() {
    const [query, setQuery] = useState("");
    const [tasks, setTasks] = useState<Task[]>([]);

    useEffect(() => {
        const controller = new AbortController();

        async function searchTasks() {
            const params = new URLSearchParams();

            if (query.trim()) {
                params.set("query", query);
            }

            const response = await fetch(`/api/tasks?${params.toString()}`, {
                signal: controller.signal,
            });

            const data = await response.json();
            setTasks(data);
        }

        searchTasks().catch((error) => {
            if (error.name !== "AbortError") {
                console.error(error);
            }
        });

        return () => {
            controller.abort();
        };
    }, [query]);

    return (
        <section>
            <input
                value={query}
                onChange={(event) => setQuery(event.currentTarget.value)}
                placeholder="Buscar..."
            />

            <ul>
                {tasks.map((task) => (
                    <li key={task.id}>{task.title}</li>
                ))}
            </ul>
        </section>
    );
}
```

Mejoras posibles:

```text
Debounce.
Search params en URL.
TanStack Query.
Validación de longitud mínima.
Estado vacío.
```

---

## 54. Persistencia en `localStorage`

Útil para preferencias locales, no para datos sensibles.

```tsx
import { useEffect, useState } from "react";

function ThemeToggle() {
    const [theme, setTheme] = useState(() => {
        return localStorage.getItem("theme") ?? "light";
    });

    useEffect(() => {
        localStorage.setItem("theme", theme);
        document.documentElement.dataset.theme = theme;
    }, [theme]);

    return (
        <button
            type="button"
            onClick={() => setTheme((current) => (
                current === "light" ? "dark" : "light"
            ))}
        >
            Cambiar tema
        </button>
    );
}
```

No guardes:

```text
Contraseñas.
Refresh tokens.
Secretos.
Datos personales sensibles.
Información que debe estar protegida por backend.
```

---

## 55. Hook personalizado para localStorage

```tsx
import { useEffect, useState } from "react";

export function useLocalStorageState<T>(
    key: string,
    initialValue: T,
): [T, React.Dispatch<React.SetStateAction<T>>] {
    const [value, setValue] = useState<T>(() => {
        const rawValue = localStorage.getItem(key);

        if (!rawValue) {
            return initialValue;
        }

        try {
            return JSON.parse(rawValue) as T;
        } catch {
            return initialValue;
        }
    });

    useEffect(() => {
        localStorage.setItem(key, JSON.stringify(value));
    }, [key, value]);

    return [value, setValue];
}
```

Uso:

```tsx
const [theme, setTheme] = useLocalStorageState("theme", "light");
```

---

# Parte V: TanStack Query

## 56. Cuándo usar TanStack Query

TanStack Query gestiona estado de servidor.

Estado de servidor:

```text
Vive fuera de React.
Puede cambiar sin que el usuario lo sepa.
Requiere fetch, cache, reintentos, sincronización e invalidación.
```

Ejemplos:

```text
Tareas desde API.
Perfil de usuario.
Resultados de búsqueda.
Permisos.
Notificaciones.
Reportes.
```

No lo uses para:

```text
Estado de un input.
Modal abierto/cerrado.
Tema visual local.
Paso actual de un wizard simple.
```

---

## 57. Instalación

```bash
npm install @tanstack/react-query
```

Configurar proveedor:

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactNode } from "react";

const queryClient = new QueryClient();

export function AppProviders({ children }: { children: ReactNode }) {
    return (
        <QueryClientProvider client={queryClient}>
            {children}
        </QueryClientProvider>
    );
}
```

Uso en `main.tsx`:

```tsx
createRoot(document.getElementById("root")!).render(
    <StrictMode>
        <AppProviders>
            <App />
        </AppProviders>
    </StrictMode>,
);
```

---

## 58. `useQuery`

```tsx
import { useQuery } from "@tanstack/react-query";
import { listTasks } from "../api/tasks";

export function TaskListPage() {
    const tasksQuery = useQuery({
        queryKey: ["tasks"],
        queryFn: () => listTasks(),
    });

    if (tasksQuery.isLoading) {
        return <p>Cargando...</p>;
    }

    if (tasksQuery.isError) {
        return <p role="alert">No se pudieron cargar las tareas.</p>;
    }

    return (
        <ul>
            {tasksQuery.data.map((task) => (
                <li key={task.id}>{task.title}</li>
            ))}
        </ul>
    );
}
```

Reglas:

```text
queryKey debe identificar la consulta.
queryFn debe retornar una promesa.
Incluye filtros dentro de queryKey.
```

Ejemplo con búsqueda:

```tsx
const tasksQuery = useQuery({
    queryKey: ["tasks", { query }],
    queryFn: () => listTasks(query),
});
```

---

## 59. `useMutation`

`useMutation` sirve para crear, actualizar o eliminar datos.

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { createTask } from "../api/tasks";

export function TaskCreateForm() {
    const queryClient = useQueryClient();

    const createTaskMutation = useMutation({
        mutationFn: createTask,
        onSuccess: () => {
            queryClient.invalidateQueries({
                queryKey: ["tasks"],
            });
        },
    });

    function handleSubmit(title: string) {
        createTaskMutation.mutate({
            title,
        });
    }

    return (
        <button
            type="button"
            disabled={createTaskMutation.isPending}
            onClick={() => handleSubmit("Nueva tarea")}
        >
            Crear tarea
        </button>
    );
}
```

---

## 60. `invalidateQueries`

Cuando una mutación cambia datos, invalida consultas relacionadas.

```tsx
queryClient.invalidateQueries({
    queryKey: ["tasks"],
});
```

Esto indica:

```text
Estos datos ya no son confiables.
Refetch si la query está activa.
```

Ejemplos:

```tsx
// Crear tarea: invalida lista
queryClient.invalidateQueries({ queryKey: ["tasks"] });

// Editar tarea: invalida detalle y lista
queryClient.invalidateQueries({ queryKey: ["tasks"] });
queryClient.invalidateQueries({ queryKey: ["task", taskId] });

// Eliminar tarea: invalida lista
queryClient.invalidateQueries({ queryKey: ["tasks"] });
```

Regla:

```text
Diseña query keys desde el inicio.
Sin query keys consistentes, las invalidaciones se vuelven caóticas.
```

---

## 61. Query keys recomendadas

```ts
export const taskKeys = {
    all: ["tasks"] as const,
    lists: () => [...taskKeys.all, "list"] as const,
    list: (filters: { query?: string }) => (
        [...taskKeys.lists(), filters] as const
    ),
    details: () => [...taskKeys.all, "detail"] as const,
    detail: (taskId: string) => (
        [...taskKeys.details(), taskId] as const
    ),
};
```

Uso:

```tsx
const tasksQuery = useQuery({
    queryKey: taskKeys.list({ query }),
    queryFn: () => listTasks(query),
});
```

Invalidar:

```tsx
queryClient.invalidateQueries({
    queryKey: taskKeys.lists(),
});
```

---

## 62. CRUD de tareas con TanStack Query

Hook personalizado:

```tsx
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import {
    createTask,
    listTasks,
    removeTask,
    updateTask,
} from "../api/tasks";
import { taskKeys } from "./taskKeys";

export function useTasks(query: string) {
    return useQuery({
        queryKey: taskKeys.list({ query }),
        queryFn: () => listTasks(query),
    });
}

export function useCreateTask() {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: createTask,
        onSuccess: () => {
            queryClient.invalidateQueries({
                queryKey: taskKeys.lists(),
            });
        },
    });
}

export function useUpdateTask() {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: ({
            taskId,
            input,
        }: {
            taskId: string;
            input: UpdateTaskInput;
        }) => updateTask(taskId, input),
        onSuccess: (updatedTask) => {
            queryClient.invalidateQueries({
                queryKey: taskKeys.lists(),
            });

            queryClient.invalidateQueries({
                queryKey: taskKeys.detail(updatedTask.id),
            });
        },
    });
}

export function useDeleteTask() {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: removeTask,
        onSuccess: () => {
            queryClient.invalidateQueries({
                queryKey: taskKeys.lists(),
            });
        },
    });
}
```

Ventaja:

```text
La UI no conoce detalles de invalidación.
Los componentes consumen hooks de dominio.
```

---

## 63. Optimistic updates

Permiten actualizar la UI antes de que el servidor confirme.

```tsx
export function useToggleTask() {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: ({
            taskId,
            completed,
        }: {
            taskId: string;
            completed: boolean;
        }) => updateTask(taskId, { completed }),
        onMutate: async ({ taskId, completed }) => {
            await queryClient.cancelQueries({
                queryKey: taskKeys.lists(),
            });

            const previousTasks = queryClient.getQueryData<Task[]>(
                taskKeys.list({}),
            );

            queryClient.setQueryData<Task[]>(
                taskKeys.list({}),
                (currentTasks = []) =>
                    currentTasks.map((task) =>
                        task.id === taskId
                            ? { ...task, completed }
                            : task,
                    ),
            );

            return {
                previousTasks,
            };
        },
        onError: (_error, _variables, context) => {
            queryClient.setQueryData(
                taskKeys.list({}),
                context?.previousTasks,
            );
        },
        onSettled: () => {
            queryClient.invalidateQueries({
                queryKey: taskKeys.lists(),
            });
        },
    });
}
```

Úsalo cuando:

```text
La operación suele ser exitosa.
El cambio es reversible.
La UX se beneficia claramente.
```

---

# Parte VI: Supabase

## 64. Qué es Supabase en React

Supabase entrega servicios como:

```text
PostgreSQL.
Auth.
Storage.
Realtime.
Edge Functions.
APIs generadas.
```

En React suele usarse mediante `@supabase/supabase-js`.

---

## 65. Instalación y cliente

```bash
npm install @supabase/supabase-js
```

Variables:

```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
```

Cliente:

```ts
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

Nota:

```text
La anon key puede estar en frontend.
No pongas service_role key en frontend.
Protege datos con Row Level Security.
```

---

## 66. Leer tareas desde Supabase

```ts
import { supabase } from "./supabaseClient";
import type { Task } from "../types/task";

export async function listTasksFromSupabase(): Promise<Task[]> {
    const { data, error } = await supabase
        .from("tasks")
        .select("id,title,description,completed,created_at")
        .order("created_at", { ascending: false });

    if (error) {
        throw error;
    }

    return data.map((row) => ({
        id: row.id,
        title: row.title,
        description: row.description,
        completed: row.completed,
        createdAt: row.created_at,
    }));
}
```

---

## 67. Insertar tareas con Supabase

```ts
export async function createTaskInSupabase(input: CreateTaskInput): Promise<Task> {
    const { data, error } = await supabase
        .from("tasks")
        .insert({
            title: input.title,
            description: input.description ?? null,
        })
        .select("id,title,description,completed,created_at")
        .single();

    if (error) {
        throw error;
    }

    return {
        id: data.id,
        title: data.title,
        description: data.description,
        completed: data.completed,
        createdAt: data.created_at,
    };
}
```

---

## 68. Supabase + TanStack Query

```tsx
export function useSupabaseTasks() {
    return useQuery({
        queryKey: ["supabase-tasks"],
        queryFn: listTasksFromSupabase,
    });
}

export function useCreateSupabaseTask() {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: createTaskInSupabase,
        onSuccess: () => {
            queryClient.invalidateQueries({
                queryKey: ["supabase-tasks"],
            });
        },
    });
}
```

Regla:

```text
Aunque Supabase simplifique backend, el frontend no debe ser autoridad de permisos.
Usa RLS, políticas y validaciones de base de datos.
```

---

# Parte VII: patrones avanzados

## 69. Estado: local, servidor, URL y persistente

Distingue dónde vive cada estado.

```text
Estado local:
Modal abierto, input actual, pestaña activa.

Estado de servidor:
Tareas, usuario, permisos, reportes.

Estado en URL:
Filtros, búsqueda, paginación, ordenamiento.

Estado persistente local:
Tema, preferencia de vista, sidebar colapsado.
```

Regla:

```text
No metas todo en useState.
Elige la fuente de verdad correcta.
```

Ejemplo:

```text
query de búsqueda:
Mejor en search params si debe poder compartirse.

lista de tareas:
Mejor en TanStack Query.

input mientras escribes:
Mejor en useState.

tema:
Mejor en localStorage o preferencia global.
```

---

## 70. Hooks personalizados

Un hook personalizado encapsula lógica reutilizable.

```tsx
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

Uso:

```tsx
const [query, setQuery] = useState("");
const debouncedQuery = useDebouncedValue(query, 300);

const tasksQuery = useTasks(debouncedQuery);
```

Reglas:

```text
Un hook debe empezar con use.
Un hook puede llamar otros hooks.
No llames hooks condicionalmente.
```

---

## 71. Búsqueda de tareas con debounce, URL y Query

```tsx
import { useSearchParams } from "react-router";
import { useDebouncedValue } from "../hooks/useDebouncedValue";
import { useTasks } from "../queries/tasks";

export function TaskSearchPage() {
    const [searchParams, setSearchParams] = useSearchParams();

    const query = searchParams.get("query") ?? "";
    const debouncedQuery = useDebouncedValue(query, 300);

    const tasksQuery = useTasks(debouncedQuery);

    function handleQueryChange(value: string) {
        setSearchParams((currentParams) => {
            const nextParams = new URLSearchParams(currentParams);

            if (value.trim()) {
                nextParams.set("query", value);
            } else {
                nextParams.delete("query");
            }

            return nextParams;
        });
    }

    return (
        <section>
            <input
                value={query}
                onChange={(event) => handleQueryChange(event.currentTarget.value)}
                placeholder="Buscar tareas"
            />

            {tasksQuery.isLoading && <p>Cargando...</p>}

            {tasksQuery.isError && (
                <p role="alert">No se pudo buscar.</p>
            )}

            {tasksQuery.data && (
                <ul>
                    {tasksQuery.data.map((task) => (
                        <li key={task.id}>{task.title}</li>
                    ))}
                </ul>
            )}
        </section>
    );
}
```

Este patrón combina:

```text
URL como fuente de filtros.
Debounce para evitar requests excesivas.
TanStack Query para cache y estado de servidor.
```

---

## 72. Separación por capas

Estructura recomendada:

```text
src/
    app/
        App.tsx
        AppRouter.tsx
        AppProviders.tsx
    api/
        http.ts
    features/
        tasks/
            api/
                tasks.api.ts
            components/
                TaskCard.tsx
                TaskForm.tsx
            hooks/
                useDebouncedValue.ts
            pages/
                TaskListPage.tsx
                TaskDetailPage.tsx
            queries/
                taskKeys.ts
                taskQueries.ts
            types/
                task.types.ts
    shared/
        components/
            Button.tsx
            Modal.tsx
        utils/
            formatDate.ts
```

Ventajas:

```text
El código de tareas está junto.
Los componentes compartidos no dependen de features.
Las APIs se centralizan.
Las queries se reutilizan.
```

---

## 73. Componentes presentacionales y contenedores

Presentacional:

```tsx
type TaskListProps = {
    tasks: Task[];
    onToggle: (taskId: string) => void;
};

export function TaskList({ tasks, onToggle }: TaskListProps) {
    return (
        <ul>
            {tasks.map((task) => (
                <li key={task.id}>
                    <button type="button" onClick={() => onToggle(task.id)}>
                        {task.title}
                    </button>
                </li>
            ))}
        </ul>
    );
}
```

Contenedor:

```tsx
export function TaskListContainer() {
    const tasksQuery = useTasks("");
    const toggleTaskMutation = useToggleTask();

    if (!tasksQuery.data) {
        return null;
    }

    return (
        <TaskList
            tasks={tasksQuery.data}
            onToggle={(taskId) => {
                const task = tasksQuery.data.find((item) => item.id === taskId);

                if (!task) {
                    return;
                }

                toggleTaskMutation.mutate({
                    taskId,
                    completed: !task.completed,
                });
            }}
        />
    );
}
```

Regla:

```text
Los componentes presentacionales son fáciles de probar y reutilizar.
Los contenedores conocen datos, rutas y mutaciones.
```

---

## 74. `useTransition`

`useTransition` permite marcar actualizaciones no urgentes.

```tsx
import { useState, useTransition } from "react";

function FilterLargeList({ items }: { items: string[] }) {
    const [query, setQuery] = useState("");
    const [filteredItems, setFilteredItems] = useState(items);
    const [isPending, startTransition] = useTransition();

    function handleChange(value: string) {
        setQuery(value);

        startTransition(() => {
            setFilteredItems(
                items.filter((item) =>
                    item.toLowerCase().includes(value.toLowerCase()),
                ),
            );
        });
    }

    return (
        <section>
            <input
                value={query}
                onChange={(event) => handleChange(event.currentTarget.value)}
            />

            {isPending && <p>Actualizando...</p>}

            <ul>
                {filteredItems.map((item) => (
                    <li key={item}>{item}</li>
                ))}
            </ul>
        </section>
    );
}
```

Úsalo para:

```text
Actualizaciones de UI costosas.
Filtrados grandes.
Cambios de vista no urgentes.
```

No lo confundas con transacciones de base de datos.

---

## 75. Transacciones de UI y consistencia

En frontend, a veces se habla informalmente de “transacciones” para agrupar cambios visuales o mantener consistencia durante una operación.

Ejemplo: crear tarea.

```text
1. Usuario envía formulario.
2. Botón pasa a disabled.
3. Se llama API.
4. Si éxito: limpiar formulario, invalidar lista, navegar.
5. Si error: mostrar mensaje y mantener datos.
```

Implementación:

```tsx
const navigate = useNavigate();
const queryClient = useQueryClient();

const mutation = useMutation({
    mutationFn: createTask,
    onSuccess: async (task) => {
        await queryClient.invalidateQueries({
            queryKey: taskKeys.lists(),
        });

        navigate(`/tasks/${task.id}`);
    },
});
```

Buenas prácticas:

```text
No limpies el formulario antes de confirmar si perder datos sería grave.
Deshabilita acciones duplicadas mientras mutation está pending.
Maneja rollback si haces optimistic update.
```

---

## 76. Drag and drop de tareas

HTML/React:

```tsx
import type { DragEvent } from "react";

type Task = {
    id: string;
    title: string;
};

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
        <ul className="task-board">
            {tasks.map((task) => (
                <li
                    key={task.id}
                    className="task-board__item"
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

CSS:

```css
.task-board {
    display: grid;
    gap: 0.75rem;
    list-style: none;
    padding: 0;
}

.task-board__item {
    padding: 1rem;
    border: 1px solid #d0d7de;
    border-radius: 0.75rem;
    background: white;
    cursor: grab;
}

.task-board__item:active {
    cursor: grabbing;
}
```

Advertencia:

```text
El drag and drop nativo requiere trabajo adicional para accesibilidad con teclado y dispositivos touch.
Para tableros complejos, evalúa librerías especializadas.
```

---

## 77. Validación runtime de respuestas

TypeScript valida en compilación, no en runtime. Si una API responde mal, TypeScript no lo detecta automáticamente.

Con Zod:

```ts
import { z } from "zod";

const taskSchema = z.object({
    id: z.string(),
    title: z.string(),
    description: z.string().nullable(),
    completed: z.boolean(),
    createdAt: z.string(),
});

const tasksSchema = z.array(taskSchema);

export async function listTasks(): Promise<Task[]> {
    const response = await fetch("/api/tasks");
    const json = await response.json();

    return tasksSchema.parse(json);
}
```

Úsalo cuando:

```text
Consumes APIs externas.
El backend cambia frecuentemente.
Hay datos críticos.
Quieres fallar temprano.
```

---

## 78. Manejo de autenticación en frontend

Frontend:

```text
Muestra u oculta UI según sesión.
Redirige si falta sesión.
Agrega token/cookie según estrategia.
Renueva estado de usuario.
```

Backend:

```text
Valida sesión/token.
Autoriza cada recurso.
Aplica permisos.
Rechaza requests no autorizadas.
```

Ejemplo con cookies HttpOnly:

```ts
fetch("/api/me", {
    credentials: "include",
});
```

Ejemplo con bearer token:

```ts
fetch("/api/me", {
    headers: {
        Authorization: `Bearer ${accessToken}`,
    },
});
```

Recomendación:

```text
Evita guardar tokens sensibles en localStorage si puedes usar una arquitectura con cookies HttpOnly o BFF.
```

---

## 79. Error boundaries

Un error boundary captura errores de renderizado en una parte del árbol.

En React moderno puedes usar librerías como `react-error-boundary`.

Ejemplo conceptual:

```tsx
import { ErrorBoundary } from "react-error-boundary";

function ErrorFallback() {
    return (
        <section role="alert">
            <h2>Algo salió mal</h2>
            <p>Intenta recargar la página.</p>
        </section>
    );
}

function App() {
    return (
        <ErrorBoundary FallbackComponent={ErrorFallback}>
            <AppRouter />
        </ErrorBoundary>
    );
}
```

No reemplaza manejo de errores de API. Sirve para fallos inesperados de UI.

---

## 80. Testing básico

Herramientas comunes:

```text
Vitest.
React Testing Library.
Playwright.
Cypress.
MSW para mock de APIs.
```

Test de componente:

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { TaskCard } from "./TaskCard";

test("calls onToggle when button is clicked", async () => {
    const user = userEvent.setup();
    const onToggle = vi.fn();

    render(
        <TaskCard
            task={{
                id: "1",
                title: "Estudiar React",
                completed: false,
            }}
            onToggle={onToggle}
            onDelete={vi.fn()}
        />,
    );

    await user.click(screen.getByRole("button", {
        name: /completar/i,
    }));

    expect(onToggle).toHaveBeenCalledWith("1");
});
```

Buenas prácticas:

```text
Prueba comportamiento, no implementación.
Busca por roles y texto visible.
No dependas de clases CSS salvo que sea necesario.
```

---

# Parte VIII: checklist final

## 81. Checklist de buenas prácticas React

```text
Componentes
[ ] Componentes pequeños y con responsabilidad clara.
[ ] Props tipadas.
[ ] Children tipado con ReactNode cuando corresponde.
[ ] Exports consistentes.
[ ] No hay lógica de API duplicada en componentes.

Estado
[ ] useState para estado local.
[ ] useRef para valores mutables que no renderizan.
[ ] useMemo solo para cálculos relevantes.
[ ] useEffect solo para sincronizar con sistemas externos.
[ ] Estado de servidor gestionado con TanStack Query cuando corresponde.

Rutas
[ ] Link para navegación interna.
[ ] useNavigate solo para navegación programática.
[ ] Rutas dinámicas validadas.
[ ] Rutas anidadas usan Outlet.
[ ] Layouts persistentes definidos.
[ ] Search params para filtros compartibles.
[ ] Rutas protegidas no reemplazan seguridad backend.

APIs
[ ] Fetch centralizado.
[ ] response.ok validado.
[ ] Errores manejados.
[ ] Contratos de datos definidos.
[ ] Respuestas críticas validadas en runtime.
[ ] Credenciales y tokens tratados con cuidado.

Formularios
[ ] Validación frontend para UX.
[ ] Validación backend para seguridad.
[ ] Mensajes de error claros.
[ ] Botones deshabilitados durante envío.
[ ] No se pierden datos en errores evitables.

TanStack Query
[ ] Query keys consistentes.
[ ] useQuery para lectura.
[ ] useMutation para escritura.
[ ] invalidateQueries después de cambios.
[ ] Optimistic updates solo cuando tiene sentido.
[ ] Hooks de dominio encapsulan queries y mutations.

Supabase
[ ] Anon key en frontend, nunca service_role.
[ ] RLS activado para tablas sensibles.
[ ] Políticas revisadas.
[ ] Tipos generados o contratos definidos.
[ ] Mutaciones invalidan queries relacionadas.

UX avanzada
[ ] Imágenes con alt.
[ ] Imágenes pesadas con loading="lazy".
[ ] Overlays accesibles.
[ ] Inlays cuando se necesita contexto persistente.
[ ] Drag and drop considera teclado/touch si es crítico.

Testing
[ ] Componentes críticos probados.
[ ] Flujos CRUD probados.
[ ] Estados loading/error/empty probados.
[ ] Rutas principales probadas.
```

---

## 82. Resumen de reglas principales

```text
1. React construye UI con componentes.
2. TypeScript define contratos, pero no valida datos en runtime.
3. Usa map con keys estables.
4. Usa Link para navegar y useNavigate para navegación programática.
5. Usa search params para filtros compartibles.
6. Usa layouts persistentes con rutas anidadas y Outlet.
7. Usa useEffect para sincronización externa, no para toda lógica.
8. Usa TanStack Query para estado de servidor.
9. Usa useMutation e invalidateQueries para mantener datos frescos.
10. Centraliza APIs y contratos.
11. Valida formularios en frontend y backend.
12. Guarda preferencias en localStorage, no secretos.
13. Supabase requiere RLS y políticas, no solo lógica frontend.
14. Diseña hooks personalizados por dominio.
15. Separa UI presentacional de lógica de datos cuando el proyecto crece.
```

---

# Fuentes de referencia recomendadas

```text
- React Docs: Components, Hooks, TypeScript, Lists, Events.
- React Router Docs: Routing, Link, useNavigate, nested routes, search params.
- TanStack Query Docs: useQuery, useMutation, query invalidation.
- Supabase Docs: React quickstart, Auth, Database, RLS.
- MDN Web Docs: Fetch API, Promises, DragEvent, MouseEvent, Web Storage.
- React TypeScript Cheatsheet.
```
