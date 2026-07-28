# Ejercicio 03 — Login con esperas explícitas y assert

**Nivel:** 🟡 Intermedio
**Requiere haber leído:** Esperas en Selenium, Excepciones comunes

---

## Objetivo

Escribir un test que:
1. Abra la página de login de práctica.
2. Intente iniciar sesión con credenciales **incorrectas** primero, y confirme que aparece un mensaje de error.
3. Luego inicie sesión con las credenciales **correctas**, y confirme que aparece un mensaje de éxito.
4. Cierre sesión, y confirme que se regresó a la página de login.

## App de práctica

**[the-internet.herokuapp.com/login](https://the-internet.herokuapp.com/login)**

Credenciales válidas (publicadas en la propia página):
- Usuario: `tomsmith`
- Contraseña: `SuperSecretPassword!`

## Pistas

<details>
<summary>Pista 1</summary>

El mensaje de error/éxito en esta página vive en el mismo elemento (`#flash`), pero cambia su texto y una clase CSS según el resultado. Puedes usar `WebDriverWait` con `visibilityOfElementLocated` antes de leer el texto.

</details>

<details>
<summary>Pista 2</summary>

No cierres el navegador entre el intento fallido y el exitoso — usa el mismo `driver`, solo limpia los campos (`clear()`) antes de escribir de nuevo.

</details>

<details>
<summary>Pista 3</summary>

El botón de "Logout" solo aparece después de un login exitoso — asegúrate de esperar a que sea clickeable antes de intentar usarlo.

</details>

## Reto extra (opcional)

Encapsula toda la lógica en un `BasePage` con métodos `login(usuario, clave)` y `obtenerMensaje()`, aplicando lo que ya viste en la nota de esperas.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [03-login-con-esperas-solucion.md](./03-login-con-esperas-solucion.md)
