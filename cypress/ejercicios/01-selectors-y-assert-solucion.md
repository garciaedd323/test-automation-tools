# Solución — Ejercicio 01: Tu primer test con selectors y assert

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto.

```javascript
describe('Checkboxes y dropdown', () => {
  it('marca el primer checkbox y confirma que quedó marcado', () => {
    cy.visit('https://the-internet.herokuapp.com/checkboxes');

    cy.get('#checkboxes input').should('have.length', 2); // reto extra

    cy.get('#checkboxes input').first().check();
    cy.get('#checkboxes input').first().should('be.checked');
  });

  it('selecciona Option 2 en el dropdown', () => {
    cy.visit('https://the-internet.herokuapp.com/dropdown');

    cy.get('#dropdown').select('Option 2');
    cy.get('#dropdown').should('have.value', '2');
  });
});
```

## Puntos clave a revisar en tu solución

- ¿Usaste `.check()` en vez de `.click()`? Cypress lo tiene como comando dedicado justo para este caso.
- ¿Usaste `.select('Option 2')` con el texto visible, en vez de intentar hacer clics manuales sobre las opciones?
- ¿Tu assert de "quedó marcado" usa `.should('be.checked')`, que ya trae retry-ability incorporado?

## Errores comunes al hacer este ejercicio

- Intentar usar `.click()` en el checkbox y luego verificar manualmente `is(':checked')` — funciona, pero `.check()` + `.should('be.checked')` es la forma idiomática en Cypress.
- Seleccionar el dropdown por el `value` interno (`'2'`) en vez del texto visible (`'Option 2'`) al usar `.select()` — ambos funcionan, pero el texto visible es más legible y menos frágil si el backend cambia los `value` internos.
