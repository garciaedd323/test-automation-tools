# Ejercicio 04 — Login con Page Object Model

**Nivel:** 🔴 Avanzado
**Requiere haber leído:** El primer test funcional completo (Appium)

---

## Objetivo

Construir una suite completa (no solo un método `main`) que:
1. Abra el menú lateral (drawer) de la app.
2. Navegue a la pantalla de "Login".
3. Inicie sesión con las credenciales de prueba que la propia app expone en pantalla (usuario y contraseña de ejemplo visibles en el formulario).
4. Confirme que aparece un mensaje o pantalla de bienvenida tras el login exitoso.

Todo esto usando **Page Object Model** — el test final no debería tener ningún `driver.findElement()` directo, solo llamadas a métodos de tus Page Objects.

## App de práctica

Misma app de los ejercicios anteriores — Sauce Labs "My Demo App" (Android).

## Pistas

<details>
<summary>Pista 1</summary>

Vas a necesitar al menos 3 Page Objects: `MenuPage` (para abrir el drawer y navegar), `LoginPage`, y quizás una clase simple para verificar el estado post-login. Reutiliza la estructura de `BasePage` que ya construiste con Selenium/Appium.

</details>

<details>
<summary>Pista 2</summary>

Usa el Inspector para confirmar cómo se abre el menú lateral — normalmente es un ícono de "hamburguesa" en la esquina superior con un `accessibility id` propio.

</details>

<details>
<summary>Pista 3</summary>

Las credenciales de prueba suelen estar impresas directamente en la pantalla de login de esta app (como ayuda visual) — no necesitas inventarlas ni buscarlas en documentación externa.

</details>

<details>
<summary>Pista 4</summary>

Estructura el test con JUnit 5: `@BeforeEach` para arrancar la sesión de Appium con las capabilities correctas, y el método `@Test` solo debería llamar a métodos de alto nivel como `menuPage.irALogin()` y `loginPage.loginExitoso()`.

</details>

## Reto extra (opcional)

Agrega un segundo `@Test` que intente el login con una contraseña incorrecta, y confirme que aparece un mensaje de error — reutilizando los mismos Page Objects sin duplicar locators.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [04-login-page-object-solucion.md](./04-login-page-object-solucion.md)
