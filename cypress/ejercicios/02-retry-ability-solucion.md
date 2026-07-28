# Solución — Ejercicio 02: Retry-ability con contenido dinámico

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto.

```javascript
describe('Dynamic Loading', () => {
  it('Example 1: espera a que el elemento se renderice y aparezca', () => {
    cy.visit('https://the-internet.herokuapp.com/dynamic_loading/1');

    cy.get('#start button').click();
    cy.get('#loading').should('be.visible');

    // Retry-ability real: sin cy.wait(numero), solo la condición
    cy.get('#finish h4').should('be.visible').and('contain', 'Hello World!');
  });

  it('Example 2: espera a que el elemento oculto se vuelva visible', () => {
    cy.visit('https://the-internet.herokuapp.com/dynamic_loading/2');

    cy.get('#start button').click();
    cy.get('#loading').should('be.visible');

    cy.get('#finish h4').should('be.visible').and('contain', 'Hello World!');
  });
});
```

## Puntos clave a revisar en tu solución

- ¿Tu test NO tiene ningún `cy.wait(numero)`? Si lo tiene, revisa si de verdad era necesario o si el `.should()` ya lo resolvía solo.
- ¿Verificaste el loader `#loading` como paso intermedio, no solo el resultado final?
- Si hiciste el reto extra: ¿confirmaste que con un timeout muy bajo el test falla, y que subiéndolo vuelve a pasar? Eso demuestra que el retry-ability estaba haciendo el trabajo real.

## La diferencia sutil entre Example 1 y Example 2

Aunque el test se ve casi idéntico, internamente son distintos: en Example 1 el elemento **no existe en el DOM** hasta que termina de cargar (aparece y desaparece del árbol). En Example 2, el elemento **ya existe pero está oculto** (`display: none`) y solo cambia su visibilidad. Cypress maneja ambos casos con el mismo `.should('be.visible')`, pero es un buen ejemplo de por qué, en Selenium, se necesitaban dos `ExpectedConditions` distintos (`presence_of_element_located` vs `visibility_of_element_located`) para diferenciar estos casos — aquí el retry-ability lo abstrae por completo.

## Errores comunes al hacer este ejercicio

- Agregar `cy.wait(3000)` "para estar seguro" — es precisamente el antipatrón que esta nota advierte evitar.
- Usar `cy.get('#finish h4').should('exist')` en vez de `'be.visible'` — en Example 2 el elemento "existe" desde el principio (solo está oculto), así que `'exist'` pasaría de inmediato sin esperar realmente a que aparezca.
