# 12 - El Mensajero Seguro: El Patrón Transaccional Outbox

Supongamos que en la sección anterior aplicamos el CQRS junto al sistema de eventos.

La operación de negocio se ve así:
1. El Agregado valida la orden (Ej. ¡Stock Suficiente!).
2. Lanzamos el evento `OrdenConfirmada`.
3. Ese evento desencadena que se mande un recibo al correo del usuario y que otro microservicio reste inventario.

El instinto del desarrollador sería programar algo como esto:

```csharp
// ❌ EL PELIGRO DEL MENSAJERO INSEGURO
public async Task Handle(CrearOrden comando)
{
    // 1. Ejecutar Lógica y Guardar
    _repo.Save(nuevaOrden); // 💾 Guardado en Base de Datos

    // 2. Avisar a otros sistemas
    var evento = new OrdenConfirmada(nuevaOrden.Id);
    await _rabbitmqBus.PublishAsync(evento); // 📨 Envío a la red para inventario/email
}
```

### 💥 El Doble Fallo Catastrófico Universitario

**Escenario A: La Base de Datos falla, el sistema miente.**
Imagina que `_repo.Save(nuevaOrden)` lanza excepción porque la Base de Datos reinició justo ese milisegundo o porque violó un constraint SQL único. La orden jamás se guardó. Pero, ¡Oh no! Intercambiamos de orden la lógica por descuido, y publicamos a RabbitMQ primero: el cliente recibió un correo confirmando una orden "fantasma" que en Base de Datos no existe, y restamos inventario de un producto que no vendimos.

**Escenario B: El Mensajero muere. El "Estado Fantasma".**
Pusimos el código en el orden "correcto" (Base de Datos primero). Pasamos `_repo.Save(nuevaOrden)` exitosamente. Luego llamamos `_rabbitmqBus.PublishAsync`. Pero justo ese milisegundo el servidor de RabbitMQ arrojó un `TimeoutException`. La orden está confirmada exitosamente en Base de Datos (Nosotros cobramos los $500 y sumamos la venta), pero el sistema nunca se enteró. A inventario nunca se le restó y al cliente jamás le llegó el PDF a su correo. Nunca hay forma de que nuestra app sola recupere e intente reenviar algo que chocó en red. Ocurrió un quiebre de consistencia silencioso y letal.

A esto se le conoce como **Consistencia Dual Flawed (fallida)**. Intentar hacer algo en dos infraestructuras distintas y fallar en la mitad.

---

## 📬 La Solución Definitiva: Transactional Outbox (La Bandeja de Salida)

¿Cómo hace tu jefe contigo para asegurarse de que un memorándum crucial salga para otras empresas pero que además quede una copia en el archivador interno sí o sí al mismo tiempo? Usa una bandeja. 

El Patrón "Transactional Outbox" usa las Transacciones Ácidas atómicas ('Todo o Nada') de SQL garantizado que guardas "Tus datos MÁS tu lista de correos por enviar" **en la misma Base de Datos en un solo hit.**

Funciona en estas dos fases perfectas:

### Fase 1: El Cajón Local (Transacción Única)
En el Handler de Comando, no intentamos comunicarnos con servicios externos de red (ni Rabbit, ni APis de email). En su lugar:

1. Abrimos una única transacción a la Base de Datos (`BEGIN TRANSACTION`).
2. Actualizamos la fila de la Orden (`UPDATE...`).
3. En **la MISMA Base de datos**, insertamos una fila en una tabla especial llamada **Bandeja de Salida (Outbox)** con el JSON del evento ("Enviar email `OrdenConfirmada`").
4. Hacemos `COMMIT`. 
5. Si hubo error en algo, los dos inserts fallan. 

### Fase 2: El Cartero en el Sótano (Relay Background Worker)
Tenemos un pequeño programa escondido (BackgroundWorker) en nuestros servidores que vigila constantemente la tabla `Outbox`.
"Aha, a las 3:15pm me metieron a un evento de ConfirmarOrden".
El Cartero agarra este mensaje de BD, y lo envía por red a RabbitMQ/AWS de forma segura. Si el cartero falla por TimeOut de red, ¡no pasa nada! Repuntará la línea y tratará en 3 segundos de nuevo. Solo cuando logra avisar a los demás, la marca "Enviada".

## Wolverine y Marten al rescate
Construir manualmente la Base de Datos de Outbox, el Hilo en Background, las re-intentativas infinitas y el evitar envíos duplicados, puede tomar semanas de desarrollo para tu equipo. 

En el EventSourcing Workshop, verás que la mágica combinación de **Marten (Base de Datos) + Wolverine (Bus)** trae el Outbox nativo e invisible. Literalmente en config escribes `.AddMartenOutbox()` y todos tus eventos jamás se perderán garantizado a nivel banco.

¡Con este pilar, tu arquitectura es virtualmente Indestructible y Escalable! Ya estás listo para el Workshop Principal de **EventSourcing**.

---
[⬅️ Volver a la sección anterior](./11-fundamentos-de-cqrs.md)
[📚 Volver al inicio: El Diario de Jhon](../01-el-diario-de-jhon.md)
