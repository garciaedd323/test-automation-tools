# Ejercicio 03 — Swipe para eliminar del carrito

**Nivel:** 🟡🔴 Intermedio-avanzado
**Requiere haber leído:** Gestos táctiles

---

## Objetivo

Escribir un test que:
1. Agregue un producto al carrito (reutiliza la lógica del ejercicio 02).
2. Vaya a la pantalla del carrito.
3. Haga un **swipe horizontal** sobre el producto para eliminarlo (la app soporta "swipe to delete" en el carrito).
4. Confirme que el carrito quedó vacío (verificando un mensaje de "carrito vacío", o que la lista de productos del carrito tiene 0 elementos).

## App de práctica

Misma app de los ejercicios anteriores — Sauce Labs "My Demo App" (Android).

## Pistas

<details>
<summary>Pista 1</summary>

Antes de programar el swipe, usa el Inspector para anotar las coordenadas aproximadas (o el tamaño) del elemento del carrito que vas a deslizar — necesitas un punto de inicio y uno de destino.

</details>

<details>
<summary>Pista 2</summary>

Repasa la sección de swipe de la nota de gestos táctiles: la duración del gesto (`Duration.ofMillis(...)`) importa — un swipe demasiado rápido puede no ser reconocido como un gesto de "deslizar", sino como un tap fallido.

</details>

<details>
<summary>Pista 3</summary>

Si el swipe no elimina el producto al primer intento, prueba aumentar la distancia del gesto (que cubra más del ancho de la pantalla) o ajustar la duración.

</details>

## Reto extra (opcional)

Agrega **dos** productos al carrito antes de eliminar solo uno, y confirma que el que queda es el correcto (no solo que "el contador bajó a 1").

---

**¿Ya lo intentaste?** Compara tu solución aquí: [03-swipe-carrito-solucion.md](./03-swipe-carrito-solucion.md)
