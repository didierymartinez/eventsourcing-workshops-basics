# 11 - Automatizando las biografías

Ya tenemos el baúl (Docker/Postgres). Pero en la sección 03 vimos que manejar listas de `object` y hacer bucles manualmente para contar cumpleaños es mucho trabajo y es fácil cometer errores.

## 🎯 El Objetivo
¿Y si existiera una herramienta que se encargue de guardar los hechos en el baúl y de reconstruir a Jhon por nosotros automáticamente?

Esa herramienta existe y se llama **Marten**.

---

## 1. Instalando el asistente
Marten es una librería que convierte a PostgreSQL en un **Event Store** de primera clase para .NET.

Abre tu terminal en la carpeta del proyecto y ejecuta:

```bash
dotnet add package Marten
```

## 2. Conectando el diario al baúl
Ahora, vamos a configurar Marten en nuestro `Program.cs`. Reemplaza el contenido de tu archivo con lo siguiente (manteniendo tus `record` de la sección 03):

```csharp
using Marten;
using Taller.HistoriaVida;

// 1. Configuramos la conexión al baúl
var store = DocumentStore.For(opts =>
{
    opts.Connection("Host=localhost;Database=historias_vida;Username=postgres;Password=mysecretpassword");
});

// 2. Abrimos una "sesión" para escribir en el diario
using var session = store.LightweightSession();

var idPersona = Guid.NewGuid();

// 3. ¡Anclamos nuevos hechos! 
// Marten se encarga de saber a qué Stream pertenecen
session.Events.StartStream<Persona>(idPersona, 
    new PersonaNacida(idPersona, "Jhon", new DateTime(1990, 5, 10)),
    new CumpleañosCelebrado(idPersona, new DateTime(1991, 5, 10))
);

await session.SaveChangesAsync();

Console.WriteLine("¡Biografía guardada en el Event Store!");
```

---

## 3. El Descubrimiento: Automatización y Stream Id
¿Notaste que ya no creamos una `List<object>` manual? 

*   **Marten es el bibliotecario**: Él recibe tus hechos y los guarda en el baúl digital.
*   **Stream Id**: Cuando usamos `idPersona`, Marten lo usa como el ID del flujo. Él sabe que todos esos eventos pertenecen a la misma "pestaña" del archivador.

> [!NOTE]
> ¿Te acuerdas de la duda sobre el ID dentro del hecho? Marten guarda el `idPersona` en una columna especial llamada `stream_id`. Así que incluso si el hecho no lo tuviera, Marten sabría de quién es.

---

Ya sabemos guardar. Pero, ¿cómo recuperamos a Jhon del baúl y sabemos cuántos años tiene sin volver a hacer bucles manuales?

[⬅️ Volver a la sección anterior](./10-el-baul-de-historias.md)

[➡️ Siguiente sección: Vistas inteligentes](./12-vistas-inteligentes.md)
