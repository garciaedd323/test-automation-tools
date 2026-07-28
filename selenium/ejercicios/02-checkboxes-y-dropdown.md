# Ejercicio 02 — Checkboxes y dropdown

**Nivel:** 🟢🟡 Básico-intermedio
**Requiere haber leído:** Locators, Interacciones con elementos

---

## Objetivo

Escribir un test que:
1. Abra la página de checkboxes de práctica.
2. Marque el primer checkbox si NO está marcado.
3. Confirme que quedó marcado.
4. Navegue a la página de dropdowns.
5. Seleccione la "Option 2" del dropdown.
6. Confirme que la opción seleccionada es la correcta.

## Apps de práctica

- Checkboxes: **[the-internet.herokuapp.com/checkboxes](https://the-internet.herokuapp.com/checkboxes)**
- Dropdown: **[the-internet.herokuapp.com/dropdown](https://the-internet.herokuapp.com/dropdown)**

## Pistas

<details>
<summary>Pista 1</summary>

Para los checkboxes, recuerda el patrón que ya viste: verificar `is_selected()`/`isSelected()` antes de hacer clic, para no desmarcar algo que ya estaba marcado.

</details>

<details>
<summary>Pista 2</summary>

Para el dropdown, este es un `<select>` HTML nativo — repasa la nota de Interacciones con elementos, sección de `Select`. No necesitas clicks manuales.

</details>

<details>
<summary>Pista 3</summary>

Para confirmar la opción seleccionada, `Select` tiene un método que te devuelve la opción actualmente elegida (`getFirstSelectedOption()` en Java).

</details>

## Reto extra (opcional)

Marca **ambos** checkboxes de la página (hay dos), y verifica que los dos quedaron en estado `true`.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [02-checkboxes-y-dropdown-solucion.md](./02-checkboxes-y-dropdown-solucion.md)
