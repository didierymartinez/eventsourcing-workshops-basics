# 09 - Hechos Históricos Inmutables: Eventos de Dominio

¿Qué es un **Evento de Dominio (Domain Event)**?

Alguien que desconoce el Lenguaje Ubicuo (Sección 07) te diría lo siguiente: *"Ah, un evento es cuando se dispara un trigger de que hemos borrado una fila de una tabla para que los cachés se vacíen en la app".*

Falso. Eso es un "Evento de Integración o de Sistema" puro (`UserRowDeletedEvent`). 

En **Domain-Driven Design (DDD)** y vitalmente en Event Sourcing, un **Evento de Dominio** es algo muchísimo más trascendental para el negocio.

Es un **hecho innegable, un hito que ocurrió en el pasado** que los expertos del negocio reconocerían de inmediato como un momento de cambio definitivo dentro de la vida del Agregado (Sección 08). 

## 💎 ¿Cómo reconocer verdaderos Eventos de Dominio?

1. **La Prueba del Tiempo (Pasado Participio):** Nunca debe llamarse como una orden (`EmitInvoice`). Es una orden ejecutada. Debe llamarse siempre en Pasado Participio (`InvoiceEmitted`, `OrdenDespachada`). "Lo hecho, hecho está".
2. **La Prueba Lingüística:** Si imprimes en consola el nombre de tu evento y tu Analista de Negocio o Gerente General exclama: "¡Aha! Necesito saber cuando eso pasa para enviar el paquete rápido en el almacén", ese es un Evento de Dominio. (`FacturaPagada`, `SuscripcionCancelada`, `MaletaDeEquipajeExtraviada`).
3. **Incurabilidad (Inmutabilidad Plena):** Tú puedes crear hoy 20 órdenes y cambiar el precio en BD (Un Error humano corregible). Pero un evento de que *"Se autorizó el despachó con valor $50 dólares porque el gerente pulsó OK a las 3:15 pm"* ocurrió de verdad. Si luego necesitas deshacer esa orden, tu debes generar "otro Evento" (P.ej. `ReembolsoOtorgadoPorCancelacion`). Borrar el primer evento en tu historial es alterar el universo temporal y un delito de auditoría contable. 

Por esto usamos los infalibles `record` enseñados en la lección 01.

## ✉️ Estructura Limpia de un Evento (El Payload)

El "sobre de datos" que contiene un evento (el Payload) debe transportar estrictamente lo mínimo y vital para describir **qué y bajo qué condiciones ocurrió**, con valores primitivos e inmodificables. No adjuntamos colecciones complejas si no son cruciales. No adjuntamos todo un `DbContext` conectado. Solo memoria fotografiada.

Imaginemos un módulo que gestiona Carritos de Compra (El Aggregate Root):

```csharp
// Un Excelente Evento de Dominio: Describe quién, qué, cómo, de forma inmutable
public record ProductoAgregadoAlCarrito(
    Guid ArticuloId,
    string Sku,
    string NombreDelMomento,
    decimal PrecioFijadoDolares
);
```

### ¿Por qué `NombreDelMomento` y no un ID a Base de datos genérica?
En un diseño crud-céntrico, un programador diría: "Si sé el `id`, el cliente hará el JOIN y el front-end buscará el nombre del producto en la tabla "catalogo_productos" actual". 

En la vida real de DDD, el Catálogo de Productos hoy dice: `Manubrio V5`, pero puede ser sobreescribido en años en la BD para que en el futuro el string actual sea `Manubrio Anticuado de Desecho`. Tú necesitas que la fotografía del **Evento Inmutable ocurrido hoy** deje en claro que esta noche al usuario le costó $5 y que ÉL lo compró con el nombre *Manubrio V5*. El Evento retiene los datos que tienen impacto legal, semántico o temporal exacto en el segundo exacto que ocurrió el Hecho en el mundo.

## 🤝 La Relación Pura: Comando -> Agregado -> Evento

Para cerrar con un moño la Arquitectura Clásica, veamos el baile que ocurrirá millones de veces en tu workshop de Event Sourcing.

Recuerda tus lecciones:

1. **[Llega El Comando]** `AgregarProductoAlCarrito` (Un intento del usuario).
2. **[El Comando es Atendido]** El API Web recibe el DTO, y se lo pasa a un Command Handler desacoplado `AgregarProductoHandler`.
3. **[Recuperamos El Objeto del Dominio]** El *Handler* inyectó el Receptorio Lógico y rehidrata (o consulta de SQL) al Aggregate Root (Caja fuerte) cargando a nuestro Agregado: el `CarritoCompras`.
4. **[El Agregado Ejecuta su lógica]** El Handler invoca `carrito.AnexarArticulo(comando.ArticuloId, comando.Precio)`. El Aggregate Root encapsulado revisa sí ese carrito puede pagarse. Como puede, ejecuta sus matemáticas internas protegiendo que nada se rompa.
5. **[El Nacimiento del Hecho]** Como el Agregado finalizó el procedimiento sin lanzar excepciones que anulen la vida, decide que acaba de ocurrir una fotografía histórica: 
   Genera e insta un nuevo `record`:
   `var nuevoEvento = new ProductoAgregadoAlCarrito(...)`.
6. **[Infraestructura]** El Handler toma este recién nacido `record`, llama al *Event Store* y graba ese hecho irrefutable en PostgreSQL en tu bóveda histórica inexpugnable. 

En Event Sourcing puro, ese guardado es la ÚNICA Base de Datos que mantendrás en tu arquitectura de Escritura. ¡Y acabas de dominar al 100% sus requerimientos arquitectónicos!

---
[⬅️ Volver a la sección anterior](./08-aggregate-y-aggregate-root.md) | [➡️ Siguiente Fase: Async / Await en el Mundo Real](./10-async-await-y-concurrencia.md)
