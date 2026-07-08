# Tipos de pruebas de software: mapa general

## 1. La gran división: funcionales vs no funcionales

Antes de ver cada tipo específico, hay que entender esta distinción de base:

- **Pruebas funcionales**: verifican **qué hace** el sistema. ¿El botón de "comprar" agrega el producto al carrito? ¿El login funciona con credenciales correctas e incorrectas? Responden a la pregunta: *"¿el sistema hace lo que debería hacer?"*

- **Pruebas no funcionales**: verifican **cómo lo hace**. ¿Es rápido? ¿Aguanta muchos usuarios a la vez? ¿Es seguro? ¿Lo puede usar una persona con discapacidad visual? Responden a la pregunta: *"¿el sistema lo hace bien, de forma segura, rápida y accesible?"*

**Analogía cotidiana**: pensá en un restaurante.
- Funcional: ¿la cocina prepara el plato que pediste? ¿te lo traen a la mesa correcta?
- No funcional: ¿cuánto tardan en traerlo (rendimiento)? ¿aguanta el restaurante un sábado a la noche lleno (carga)? ¿la cocina tiene higiene y seguridad (seguridad)? ¿el menú tiene opciones en braille o el mozo puede atender a alguien con dificultad auditiva (accesibilidad)?

---

## 2. Pruebas funcionales

### 2.1 Pruebas unitarias

Prueban una pieza aislada de código (una función, un método) sin depender de otras partes del sistema.

**Analogía cotidiana**: antes de armar un mueble de IKEA, revisás que cada tornillo, cada tabla y cada bisagra individual no esté rota o defectuosa, sin haber armado nada todavía.

**Ejemplo**: probar que una función `calcularDescuento(precio, porcentaje)` devuelve el valor correcto para distintos casos (0%, 50%, 100%, valores negativos).

### 2.2 Pruebas de integración

Verifican que varios módulos o componentes funcionen bien **en conjunto**.

**Analogía cotidiana**: ya armaste el cajón del mueble (con sus tablas y bisagras) y ahora comprobás que el cajón encaje bien en el mueble y que el riel deslice sin trabarse.

**Ejemplo**: probar que cuando el backend recibe un pedido, lo guarda correctamente en la base de datos y descuenta el stock del inventario.

### 2.3 Pruebas end-to-end (E2E)

Simulan el flujo completo de un usuario real, de principio a fin, a través de toda la aplicación.

**Analogía cotidiana**: el mueble ya está armado completo. Ahora simulás el uso real: abrís la puerta, sacás un cajón, guardás algo, cerrás todo. Verificás la experiencia completa, no una pieza suelta.

**Ejemplo**: un usuario entra al sitio, busca un producto, lo agrega al carrito, paga, y recibe el email de confirmación.

### 2.4 Pruebas de regresión

Verifican que cambios nuevos en el código **no hayan roto** funcionalidades que ya funcionaban antes.

**Analogía cotidiana**: le agregaste una repisa nueva al mueble. Antes de dar por terminado el trabajo, volvés a revisar que los cajones de siempre sigan abriendo bien y que la puerta no haya quedado desalineada por el cambio.

**Ejemplo**: después de agregar un nuevo método de pago, volver a probar que el login, el carrito y los métodos de pago anteriores sigan funcionando igual.

### 2.5 Pruebas de humo (smoke testing)

Un chequeo rápido y superficial para verificar que **lo más básico funciona**, antes de invertir tiempo en pruebas más profundas.

**Analogía cotidiana**: antes de arrancar un auto para un viaje largo, girás la llave para confirmar que prende, revisás que tenga combustible y que las luces enciendan. No revisás el motor a fondo, solo lo esencial para saber si vale la pena seguir.

**Ejemplo**: después de un despliegue nuevo, verificar rápidamente que la página carga, que el login funciona y que no hay una pantalla en blanco. Si algo básico falla, ni se sigue probando el resto.

---

## 3. Pruebas no funcionales

### 3.1 Pruebas de rendimiento (performance)

Miden qué tan rápido responde el sistema bajo condiciones normales de uso.

**Analogía cotidiana**: cronometrar cuánto tarda un mozo en traerte el café en un día normal, con la cantidad habitual de clientes.

**Ejemplo**: medir cuánto tarda en cargar la página de resultados de búsqueda con 100 usuarios navegando el sitio al mismo tiempo.

### 3.2 Pruebas de carga (load testing)

Evalúan cómo se comporta el sistema bajo una cantidad de usuarios o transacciones **esperada o elevada**, para ver si mantiene el rendimiento.

**Analogía cotidiana**: ver si el restaurante puede atender bien a 80 comensales un sábado a la noche, sin que el servicio se vuelva lento o se cometan errores en los pedidos.

**Ejemplo**: simular 10,000 usuarios comprando entradas para un recital al mismo tiempo, apenas se abre la venta.

*(Nota relacionada: las pruebas de estrés (stress testing) son similares pero llevan el sistema más allá de su capacidad esperada, para ver en qué punto se rompe y cómo se recupera.)*

### 3.3 Pruebas de seguridad

Buscan vulnerabilidades que un atacante podría explotar: accesos no autorizados, fugas de datos, inyecciones de código malicioso, etc.

**Analogía cotidiana**: contratar a alguien para que intente entrar a tu casa por todas las ventanas y puertas posibles, para encontrar los puntos débiles antes de que lo haga un ladrón de verdad.

**Ejemplo**: verificar que un usuario no pueda ver los datos de otro usuario simplemente cambiando un número en la URL, o que la app no sea vulnerable a inyección SQL en el formulario de login.

### 3.4 Pruebas de accesibilidad

Verifican que la aplicación pueda ser usada por personas con distintas capacidades: visuales, auditivas, motrices o cognitivas.

**Analogía cotidiana**: revisar que un edificio tenga rampas para sillas de ruedas, señalización en braille, y puertas lo suficientemente anchas, no solo escaleras y puertas pensadas para una única forma de moverse.

**Ejemplo**: comprobar que un sitio se pueda navegar completo usando solo el teclado (sin mouse), y que un lector de pantalla pueda leer correctamente cada botón e imagen.

### 3.5 Pruebas de usabilidad

Evalúan qué tan fácil e intuitivo es usar el sistema para una persona real, sin conocimientos técnicos previos.

**Analogía cotidiana**: darle un electrodoméstico nuevo a alguien sin manual de instrucciones y observar si logra usarlo sin frustrarse.

**Ejemplo**: sentar a 5 usuarios reales frente a una nueva app y observar si logran completar una compra sin ayuda ni confusión.

### 3.6 Pruebas de compatibilidad

Verifican que el sistema funcione correctamente en distintos entornos: navegadores, sistemas operativos, dispositivos, resoluciones de pantalla.

**Analogía cotidiana**: probar que una llave abra la puerta sin importar si la cerradura es de una marca u otra, vieja o nueva.

**Ejemplo**: comprobar que un sitio web se vea y funcione bien tanto en Chrome como en Safari, en una PC y en un celular.

---

## 4. Tabla resumen

| Tipo | Categoría | Pregunta que responde | Ejemplo cotidiano |
|---|---|---|---|
| Unitaria | Funcional | ¿Esta pieza aislada funciona? | Revisar cada tornillo antes de armar el mueble |
| Integración | Funcional | ¿Las piezas funcionan juntas? | El cajón encaja y desliza en el mueble |
| End-to-end | Funcional | ¿Funciona el flujo completo? | Usar el mueble armado como se usaría en la vida real |
| Regresión | Funcional | ¿Se rompió algo que antes andaba? | Revisar que los cajones viejos sigan bien tras agregar una repisa |
| Humo (smoke) | Funcional | ¿Lo básico arranca? | Girar la llave del auto antes de un viaje largo |
| Rendimiento | No funcional | ¿Es rápido en condiciones normales? | Cronometrar al mozo en un día normal |
| Carga | No funcional | ¿Aguanta muchos usuarios a la vez? | El restaurante lleno un sábado a la noche |
| Seguridad | No funcional | ¿Es vulnerable a ataques? | Buscar puertas y ventanas sin cerrar en una casa |
| Accesibilidad | No funcional | ¿Lo puede usar cualquier persona? | Rampas y braille en un edificio |
| Usabilidad | No funcional | ¿Es fácil e intuitivo de usar? | Usar un electrodoméstico sin manual |
| Compatibilidad | No funcional | ¿Funciona en distintos entornos? | Una llave que abre distintas cerraduras |

---

## 5. Cómo se relaciona esto con la pirámide de testing

La pirámide de testing (unitarias, integración, E2E, manual/exploratorio) describe principalmente **pruebas funcionales** y su nivel de automatización. Las pruebas no funcionales (rendimiento, seguridad, accesibilidad, etc.) son una **capa adicional**, transversal, que se suele automatizar con herramientas especializadas y ejecutar en momentos puntuales del ciclo de desarrollo (antes de un release grande, de forma periódica, o continuamente en CI/CD según el caso).
