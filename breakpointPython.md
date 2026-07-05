# Comandos Básicos de `breakpoint()` en Python (Generado por ChatGPT)

Cuando usas `breakpoint()`, se activa el depurador `pdb`, y puedes usar los siguientes comandos:

| Comando | Descripción |
|---------|------------|
| `h` / `help` | Muestra la ayuda con los comandos disponibles. |
| `c` / `continue` | Continúa la ejecución hasta el siguiente breakpoint. |
| `q` / `quit` | Sale del depurador y termina el programa. |
| `n` / `next` | Ejecuta la siguiente línea sin entrar en funciones. |
| `s` / `step` | Avanza una línea, entrando en funciones si es necesario. |
| `l` / `list` | Muestra el código fuente alrededor de la línea actual. |
| `p variable` | Imprime el valor de una variable. |
| `pp variable` | Imprime una variable de manera más legible. |
| `w` / `where` | Muestra la pila de llamadas actual. |
| `b` / `break` | Coloca un breakpoint en una línea específica. |
| `tbreak` | Establece un breakpoint temporal (solo se activará una vez). |
| `cl` / `clear` | Elimina un breakpoint. |
| `r` / `return` | Ejecuta hasta el final de la función actual y muestra su valor de retorno. |
| `u` / `up` | Sube un nivel en la pila de llamadas. |
| `d` / `down` | Baja un nivel en la pila de llamadas. |

> **Nota:** `breakpoint()` está disponible desde Python 3.7 y es equivalente a `pdb.set_trace()`, pero respeta la variable de entorno `PYTHONBREAKPOINT`.

### Ejemplo de Uso:

```python
def suma(a, b):
    resultado = a + b
    breakpoint()  # Se detendrá aquí para depuración
    return resultado

print(suma(3, 5))
```
# Trabajando con Bucles en `breakpoint()`

Cuando usamos `breakpoint()` para depurar bucles, nos enfrentamos a un problema: `pdb` solo permite ejecutar una línea a la vez, por lo que escribir un bucle completo dentro del depurador no es posible. Sin embargo, podemos utilizar diferentes estrategias para analizar la ejecución dentro de un bucle.

---

## 1️⃣ Insertar `breakpoint()` Dentro del Bucle

La forma más sencilla de depurar un bucle es colocar `breakpoint()` dentro de su cuerpo para detener la ejecución en cada iteración.

```python
def procesar_lista(lista):
    for i, elemento in enumerate(lista):
        breakpoint()  # Se detendrá en cada iteración
        print(f"Índice: {i}, Elemento: {elemento}")

mi_lista = [10, 20, 30]
procesar_lista(mi_lista)
```

### Cómo avanzar en el bucle:
* Usa `n` (next) para avanzar línea por línea sin entrar en funciones.
* Usa `c` (continue) para ejecutar hasta la próxima interrupción (`breakpoint()`).
* Usa `p` variable para inspeccionar valores, por ejemplo: `p i`, `p elemento`.

## 2️⃣ Usar until para Saltar a la Siguiente Iteración
Si quieres que el bucle avance automáticamente hasta la siguiente iteración sin tener que presionar `n` varias veces, puedes usar `until <número de línea>`. Esto ejecutará todas las instrucciones hasta que el código vuelva a la línea donde inicia el bucle.

### 🔹 Ejemplo:
```python
def recorrer_numeros():
    for i in range(5):
        breakpoint()  # Se detendrá en cada iteración
        print(f"Valor de i: {i}")

recorrer_numeros()
```
Cuando el código se detenga en `breakpoint()`, usa el comando:
```python
until 5  # Si la línea donde inicia el bucle es la 5
```
Esto hará que el código avance hasta que vuelva a la línea donde inicia el bucle (`for i in range(5)`) en la siguiente iteración.

### 💡 ¿Cuándo usar `until`?
* Para evitar detenerte en cada línea del bucle.
* Para avanzar rápidamente a la siguiente vuelta sin salir del depurador.
* Para verificar cambios en variables después de varias iteraciones.

## 3️⃣ Modificar Variables Durante la Ejecución
Si necesitas cambiar valores en el depurador para probar diferentes escenarios sin detener la ejecución del script, puedes modificar las variables dentro de `pdb` usando `!`.
```python
p i          # Ver el valor actual de i
p elemento   # Ver el elemento actual
!i = 5       # Cambiar i manualmente
!elemento = 100  # Modificar el valor del elemento
```
Luego, continúa con `n` o usa `until` para avanzar en la ejecución.
