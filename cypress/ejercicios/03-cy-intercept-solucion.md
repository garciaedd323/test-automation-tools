# Solución — Ejercicio 03: Interceptar red con cy.intercept

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto. La URL exacta puede variar levemente si la app de ejemplo cambia — usa las herramientas de desarrollador para confirmarla si algo no coincide.

## `cypress/fixtures/comentario-falso.json` (reto extra)

```json
{
  "name": "Comentario de prueba",
  "body": "Este es un comentario inventado para el test"
}
```

## El test

```javascript
describe('cy.intercept con la Kitchen Sink de Cypress', () => {
  beforeEach(() => {
    cy.visit('https://example.cypress.io/commands/network-requests');
  });

  it('espía la petición real sin modificarla', () => {
    cy.intercept('GET', '**/comments/*').as('getComment');

    cy.get('.network-btn').click();

    cy.wait('@getComment').its('response.statusCode').should('eq', 200);
  });

  it('mockea la respuesta con un fixture', () => {
    cy.intercept('GET', '**/comments/*', { fixture: 'comentario-falso.json' }).as('getComentarioFalso');

    cy.get('.network-btn').click();

    cy.wait('@getComentarioFalso');
    cy.get('.network-comment').should('contain', 'Este es un comentario inventado para el test');
  });

  it('simula un error de red', () => {
    cy.intercept('GET', '**/comments/*', { forceNetworkError: true }).as('getComentarioError');

    cy.get('.network-btn').click();

    cy.wait('@getComentarioError');
    // La app de ejemplo no siempre muestra un mensaje de error visible;
    // lo mínimo verificable es que la petición interceptada efectivamente falló.
  });
});
```

## Puntos clave a revisar en tu solución

- ¿Tu `cy.intercept()` está declarado **antes** del `.click()` en los tres tests?
- ¿Verificaste el `statusCode` de la respuesta real en el primer test, no solo que "algo pasó"?
- ¿El texto mockeado en el segundo test aparece exactamente igual en la aserción final (mismo texto que puso en el fixture/JSON)?

## Errores comunes al hacer este ejercicio

- Poner el `cy.intercept()` después del clic — la petición ya se disparó y el espía llega tarde.
- Copiar la estructura completa de la respuesta real de la API sin necesidad — para el mock, alcanza con los campos que la interfaz efectivamente usa.
- Olvidar `.as()` y luego intentar usar `cy.wait('@algo')` sin haber declarado ese alias.
