# 10 - El Recepcionista: Inyección de Dependencias

Con nuestro motor Event Sourcing preparado para el mundo asíncrono, estamos a un paso de conectar todo a una base de datos real.

Pero si abrimos nuestro punto de entrada (el lugar donde arranca la aplicación, como `Program.cs`), encontraremos un problema de diseño que hará colapsar la Agencia de Biografías si crece un poco más.

## 🚀 El Problema: El jefe hace todo el trabajo manual

Hasta ahora, para que nuestro sistema funcione, nosotros (como programadores en la raíz de la app) tenemos que instanciar y ensamblar las piezas manualmente:

```csharp
// ❌ Acoplamiento manual (Creacional)
// Program.cs

// 1. Instanciamos el almacén
IEventStore miAlmacen = new InMemoryEventStore();

// 2. Instanciamos el Handler pasándole el almacén manualmente
var handlerBoda = new MatrimonioSolicitadoHandler(miAlmacen);
var handlerMudanza = new MudanzaSolicitadaHandler(miAlmacen);

// 3. Ya podemos usar los handlers...
```

¿Qué pasará mañana cuando el `EventStore` necesite una conexión a PostgreSQL (`new PostgresConnection()`) y un formateador de JSON (`new JsonSerializer()`) para funcionar?

Tendríamos que escribir el infierno de los "new":

```csharp
var conexion = new PostgresConnection("Host=localhost;...");
var json = new MiJsonMaker();
var almacen = new PostgresEventStore(conexion, json);
var handlerBoda = new MatrimonioSolicitadoHandler(almacen);
```

Si tenemos 50 Handlers, nuestro `Program.cs` será una pesadilla de configuración. El "Jefe" (nuestra app) está perdiendo el tiempo armando los escritorios de los empleados en lugar de atender clientes.

---

## 🛎️ La Solución de C#: El Recepcionista (Contenedor DI)

Para resolver esto, la industria adoptó el patrón de **Inyección de Dependencias (DI)**. 
Imagina que contratamos a un Recepcionista súper eficiente para la Agencia.

1. **El Registro (`IServiceCollection`)**: Al inicio del día, le decimos al Recepcionista cómo se fabrica cada empleado o herramienta. *"Señor Recepcionista, cuando alguien pida un `IEventStore`, entréguele un `InMemoryEventStore`"*.
2. **El Suministro (`IServiceProvider`)**: El Recepcionista se queda en la puerta. Cuando llega un Comando de Boda, el sistema le dice al Recepcionista: *"Necesito un `MatrimonioSolicitadoHandler`"*. El Recepcionista lee el constructor del Handler, ve que necesita un `IEventStore`, fabrica el Almacén él mismo, se lo inyecta al Handler, y te entrega el Handler listo para usar.

> [!TIP]
> A este concepto se le llama **Inversión de Control (IoC)**. El Handler ya no asume el control de crear o buscar su almacén (`new InMemoryEventStore()`). En su lugar, simplemente *exige* un `IEventStore` en su constructor, y confía en que el sistema se lo proveerá (se lo inyectará) por arte de magia.

---

## 🛠️ Implementando DI en .NET

Modernamente, ASP.NET Core y las aplicaciones Worker en C# traen este "Recepcionista" incorporado.

Así se configura nuestro registro moderno:

```csharp
using Microsoft.Extensions.DependencyInjection;

// 1. Contratamos al Recepcionista (Crear la Colección)
var services = new ServiceCollection();

// 2. INSTRUCCIONES: Le enseñamos qué hacer.
// "AddSingleton" significa: Crea UNO SOLO para toda la vida de la app.
services.AddSingleton<IEventStore, InMemoryEventStore>();

// "AddTransient" significa: Crea UNO NUEVO cada vez que alguien te lo pida.
services.AddTransient<MatrimonioSolicitadoHandler>();

// 3. Abrimos la oficina (Construir el Proveedor)
var proveedor = services.BuildServiceProvider();
```

Y así se usa en el resto de la aplicación (en tus Controladores Web o APIs):

```csharp
// ... llega una petición web ...

// MAGIA: No usamos 'new'. Le pedimos al recepcionista que nos dé el Handler.
// Él automáticamente lee el constructor, fabrica el EventStore (o usa el que ya tiene), 
// lo inyecta al Handler, y nos lo entrega listo.
var handlerBoda = proveedor.GetRequiredService<MatrimonioSolicitadoHandler>();

await handlerBoda.HandleAsync(comandoBoda);
```

---

### El Descubrimiento Crítico

La Inyección de Dependencias no es solo una comodidad, es el **puente que conecta el dominio con la infraestructura**. 

Nuestros Handlers de la Fase 1 (`MatrimonioSolicitadoHandler`) exigen en su constructor un `IEventStore`. Ellos **no tienen idea** si la persistencia ocurre en RAM, en Postgres o en un archivo de texto. A ellos no les importa; el Recepcionista se encarga del trabajo sucio.

¿Por qué es vital aprender esto ahora? Porque en la siguiente fase (Persistencia Real), descubriremos que la famosa librería **Marten** no es más que una caja llena de instrucciones para nuestro Recepcionista.

Cuando escribamos la línea mágica `services.AddMarten(...)`, Marten le enseñará a nuestro Recepcionista cómo conectarse a la base de datos y cómo proveernos interfaces profesionales para leer y guardar eventos, borrando de un plumazo todas las clases manuales que escribimos para simular el almacenamiento.

**¡Es hora de conectar Postgres!**

---

[⬅️ Volver a la sección anterior](./09-el-riesgo-de-olvidar.md)

[➡️ Siguiente Fase: El Baúl Incombustible (Docker y PostgreSQL)](./11-docker-postgres.md)
