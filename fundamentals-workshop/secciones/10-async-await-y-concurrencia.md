# 10 - El Bloqueo Mortal: Async / Await en el Mundo Real
> 🎯 **Hacia dónde va:** Async/await te da la escalabilidad indispensable en Event Sourcing, donde casi todo toca infraestructura (EventStores, buses de mensajes); aprendes a no bloquear hilos al guardar o publicar eventos.
> 📦 **Ejemplo de esta sección:** Hospital / Paciente (guardado del registro).

Hasta ahora en tus tutoriales básicos, cuando quieres guardar datos en la base de datos escribes algo como:

```csharp
// Guardado Sincrónico Clásico
public void Handle(Registro comando)
{
    var paciente = _repo.Get(comando.Id);
    paciente.Nombre = comando.Nombre;
    
    _repo.Save(paciente); // ⚠️ LÍNEA PELIGROSA
    
    // El programa continúa aquí...
}
```

¿Qué hace exactamente el procesador (la CPU) de tu servidor web (Ej. Kestrel de ASP.NET) cuando llega a la línea `_repo.Save(paciente)`?

El Microprocesador le envía una señal de red a la Base de Datos en otra ciudad por cable de fibra óptica, pidiéndole que guarde los datos. Ese viaje por la red, más el tiempo de espera a que el disco duro de la base de datos mecánica gire e inserte los datos, toma unos **100 Milisegundos**.

Para tu CPU, que opera en nanosegundos, 100 Milisegundos es el equivalente a una eternidad. Es como si el cocinero principal de un restaurante enviara al mesero a comprar tomates a España, y **el cocinero, en lugar de seguir picando cebollas para otras mesas, se quedara congelado mirando la puerta del restaurante durante 4 meses esperando a que el mesero regrese con los tomates.**

Ese "Cocinero congelado mirando la puerta" se llama un **Hilo Bloqueado (Thread Blocking)**.

Si el servidor tiene solo 50 hilos (cocineros), y llegan 50 usuarios al mismo tiempo a guardar datos, tus 50 cocineros se quedarán congelados esperando la red. Si llega el usuario número 51, la página web se cae ("Timeout") porque no hay nadie trabajando en el restaurante.

## 🚀 La Solución: Asincronía (Async / Await)

Para evitar que tu servidor colapse, C# introdujo el patrón Asincrónico mediante `async` y `await`.

```csharp
// ✅ Guardado Asincrónico Moderno
public async Task Handle(Registro comando)
{
    var paciente = await _repo.GetAsync(comando.Id);
    paciente.Nombre = comando.Nombre;
    
    await _repo.SaveAsync(paciente); // 🌟 LA MAGIA OCURRE AQUÍ
    
    // ...
}
```

### ¿Qué hace el Await?

1. Cuando la CPU llega a `await _repo.SaveAsync(paciente)`, le envía el paquete de red a la Base de Datos.
2. Inmediatamente, la CPU coloca "un post-it de memoria" en esa línea de código y dice: *"Me avisas cuando los tomates lleguen de España"*.
3. **EL HILO (EL COCINERO) QUEDA LIBRE INSTANTÁNEAMENTE.** Y se va corriendo a atender al Usuario 51 que acaba de entrar a la web.
4. 100 Milisegundos después, la tarjeta de red recibe la respuesta de la Base de datos ("Guardado exitoso!").
5. El sistema busca un hilo libre (cualquier cocinero) y le dice: *"Retoma desde el Post-It"*.

### ¿Por qué obligatoriamente `Task`?

Un `Task` en C# es como darte un **Beep de restaurante de comida rápida**. Cuando llamas al método asíncrono, no te devuelven el objeto `Paciente` de inmediato (porque apenas fuimos a red). Te entregan un `Task` (un aparato vibrador).
La palabra `await` significa *"Haz que este cocinero atienda otras cosas, y pon este código en pausa hasta que el `Task` vibre (se complete)"*. Y cuando vuelva, mágicamente extrae el objeto real.

### Reglas para Escalar (Especialmente en Event Sourcing)

Como en Event Sourcing TODO tu sistema se basa en comunicarse con EventStores en Bases de Datos y lanzar eventos por Bus de Mensajes:

1. Nunca uses `.Result` o `.Wait()` en código asíncrono. Vuelves a **congelar el hilo** esperando — y bajo carga **agotas el pool de hilos** y la app se cae (*thread starvation*). (En apps de UI o ASP.NET clásico, además, causa un **deadlock** clásico por el contexto de sincronización.) La regla: **async hasta arriba**, sin bloquear.
2. Todo lo que toque Infraestructura (Base de datos, Red, Archivos) DEBE llevar la firma `async Task`.
3. **Propaga el `CancellationToken`.** Toda API async seria lo recibe (`Task GuardarAsync(…, CancellationToken ct)`); pásalo siempre para poder abortar trabajo cuando el cliente se va.
4. Todo lo que sea puramente memoria RAM (como los métodos en tu Agregado `Persona.Casar()`) son sincrónicos, no llevan `async` ni `Task`. ¡El CPU es feliz reventando números en memoria en nanosegundos!

> [!NOTE]
> Y desmonta el mito: `async` **no hace tu código más rápido** ni "lo corre en otro hilo". En I/O **no hay ningún hilo esperando**; lo que ganas es **escalabilidad** (atender más peticiones con los mismos hilos).

---
[⬅️ Volver a la sección anterior](./09-eventos-de-dominio.md) | [➡️ Siguiente sección: Fundamentos de CQRS](./11-fundamentos-de-cqrs.md)
