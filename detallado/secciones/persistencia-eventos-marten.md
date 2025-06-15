## Persistencia de eventos con Marten

Ahora que tienes definidos tus eventos, el siguiente paso es guardarlos y consultarlos en una base de datos. Para esto utilizaremos Marten, una biblioteca para .NET que facilita la persistencia de eventos y el manejo de agregados utilizando PostgreSQL.

A continuación, se describen los pasos para instalar Marten en tu proyecto:

### 1. Usando la CLI de .NET

Ejecuta el siguiente comando en la terminal dentro de la carpeta del proyecto:

```
dotnet add package Marten
```

### 2. Usando el archivo .csproj

Agrega la siguiente línea dentro del archivo `.csproj` de tu proyecto:

```xml
<PackageReference Include="Marten" Version="5.*" />
```

### 3. Usando el administrador de paquetes NuGet (Visual Studio)

1. Haz clic derecho en el proyecto > "Administrar paquetes NuGet".
2. Busca "Marten" e instálalo.



## Integra Marten para persistir tus eventos

Ahora que tienes tu modelo de eventos, el siguiente paso es integrar Marten para poder guardar y consultar esos eventos en la base de datos.

### Paso 1: Agrega la configuración de Marten

En tu archivo `Program.cs`, deberás crear una instancia de `DocumentStore` con la cadena de conexión a tu base de datos PostgreSQL. Por ejemplo:

```csharp
using Marten;

var store = DocumentStore.For(opt =>
    opt.Connection("User ID=postgres;Password=Marten123;Host=localhost;Port=5432;Database=marten-demo;Pooling=true;MaxPoolSize=20")
);
```

> **Explicación:**
> - `DocumentStore` es el punto de entrada de Marten y se encarga de la configuración y conexión a la base de datos.
> - Solo necesitas crear una instancia de `DocumentStore` una vez y reutilizarla durante toda la vida de la aplicación.

### Paso 2: Abre una sesión para interactuar con la base de datos

Con el `store` creado, puedes abrir una sesión para guardar o consultar eventos:

```csharp
using var session = store.LightweightSession();
// Aquí podrás guardar eventos, consultar documentos, etc.
```

> **¿Por qué se usa `using` al crear la sesión?**
>
> La instrucción `using` en C# asegura que los recursos asociados a la sesión se liberen correctamente cuando ya no se necesiten. La sesión de Marten implementa la interfaz `IDisposable`, lo que significa que puede contener recursos no administrados, como conexiones a la base de datos. Al usar `using`, se garantiza que la sesión se cierre y libere automáticamente al finalizar el bloque, incluso si ocurre una excepción. Esto evita dejar conexiones abiertas y posibles problemas de rendimiento o bloqueos en la base de datos.

### Nota sobre sesiones asíncronas

Marten soporta operaciones asíncronas para trabajar eficientemente con la base de datos, especialmente útil en aplicaciones que requieren alta concurrencia o no deben bloquear el hilo principal (por ejemplo, aplicaciones web o APIs).

Puedes abrir una sesión y asegurarte de que su liberación (dispose) sea asíncrona usando:

```csharp
await using var session = store.LightweightSession();
```

> **Nota:** El uso de `await using` no significa que la creación de la sesión sea asíncrona, sino que la liberación de los recursos (dispose) de la sesión se realizará de forma asíncrona cuando termine el bloque. Esto es importante para liberar correctamente conexiones y otros recursos en escenarios asíncronos.

Esto te permite aprovechar el patrón `async/await` en todas las operaciones de Marten, como guardar o consultar eventos

### Tipos de sesión en Marten

Marten ofrece diferentes tipos de sesión para interactuar con la base de datos, según el escenario de uso:

- **LightweightSession**: Es la sesión más simple y eficiente. No realiza seguimiento de cambios en los documentos cargados. Ideal para operaciones de solo lectura o cuando no necesitas rastrear modificaciones automáticas.
- **IdentityMapSession**: Realiza seguimiento de los documentos cargados en la sesión. Si cargas el mismo documento varias veces, obtendrás la misma instancia. Útil cuando necesitas consistencia de identidad y rastreo de cambios.
- **DirtyTrackingSession**: Además de la funcionalidad de IdentityMap, detecta automáticamente los cambios realizados en los documentos y los guarda al llamar a `SaveChangesAsync()`. Es útil para escenarios donde quieres que Marten detecte y persista cambios automáticamente.

**¿Cuál usar?**
- Para la mayoría de los escenarios de eventos y operaciones simples, `LightweightSession` es suficiente y más eficiente.
- Si necesitas trabajar con documentos y quieres que Marten rastree los cambios, usa `IdentityMapSession` o `DirtyTrackingSession`.

Puedes crear cada tipo de sesión así:

```csharp
using var session = store.LightweightSession(); // Más común y eficiente
// o
using var session = store.IdentityMapSession();
// o
using var session = store.DirtyTrackingSession();
```

> En la mayoría de los ejemplos de eventos y Event Sourcing, se recomienda usar `LightweightSession`.

---

## Siguiente paso: Crear y guardar un stream de eventos

Ahora que tienes tu sesión abierta, el siguiente paso es crear un identificador único (id) para tu agregado. Este id se usará como identificador del stream de eventos en Marten.

### ¿Qué es un stream?

En Marten, un **stream** es una secuencia de eventos relacionados con un mismo agregado (por ejemplo, una orden de compra). Cada stream tiene un identificador único (id), que normalmente corresponde al id del agregado. Todos los eventos que afectan a ese agregado se almacenan en el mismo stream, permitiendo reconstruir su estado a partir de la secuencia de eventos.

### ¿Cómo se usa el id?

El id que generes (por ejemplo, un string o un Guid) será el identificador del stream y del agregado. Ejemplo:

```csharp
var id = Guid.NewGuid(); // o un string como "1"
```

### ¿Qué es `session.Events`?

`session.Events` es la API de Marten para trabajar con eventos. Permite crear nuevos streams, agregar eventos a streams existentes, consultar eventos, etc.

### ¿Cómo crear un stream de eventos?

Para crear un nuevo stream y guardar los primeros eventos de un agregado, usa el método `StartStream()`:

```csharp
var id = Guid.NewGuid();
var ordenCreada = new OrdenCreada(id.ToString(), "ORD-001");
var productoAgregado = new ProductoAgregado("1", "Laptop", 1500m);

session.Events.StartStream(id, ordenCreada, productoAgregado);
```

- `StartStream(id, eventos...)` crea un nuevo stream para el agregado, con el id dado.

### Agregar eventos adicionales a un stream

Puedes agregar varios eventos a un stream de dos formas:

#### 1. Agregar varios eventos al crear el stream

Puedes pasar todos los eventos iniciales como parámetros a `StartStream`. Por ejemplo:

```csharp
var idOrden = Guid.NewGuid();
var ordenCreada = new OrdenCreada(idOrden, "ORD-001");
var productoAgregado = new ProductoAgregado("pro1", "Computador", 12_000);

// Crea el stream y agrega ambos eventos de una vez
var stream = session.Events.StartStream(idOrden, ordenCreada, productoAgregado);
```

#### 2. Agregar eventos adicionales con `Append`

Si necesitas agregar más eventos después de haber creado el stream, utiliza `Append`:

```csharp
var productoAgregado2 = new ProductoAgregado("pro1", "ComputadorAppend", 12_000);
session.Events.Append(idOrden, productoAgregado2);
```

### Guardar los cambios: el patrón Unit of Work

Una vez que has agregado los eventos a la sesión, debes guardar los cambios en la base de datos. Esto se hace con:

```csharp
await session.SaveChangesAsync();
```

Este método implementa el patrón **Unit of Work**: todas las operaciones realizadas en la sesión (agregar eventos, documentos, etc.) se agrupan y se envían a la base de datos en una sola transacción cuando llamas a `SaveChangesAsync()`. Así, aseguras que los cambios se guarden de forma atómica y consistente.

> Siempre recuerda llamar a `SaveChangesAsync()` después de agregar o modificar eventos para que los cambios se persistan en la base de datos.

> **Nota:** Cuando ejecutas por primera vez la persistencia de eventos con Marten, la librería se encarga automáticamente de crear las tablas y la estructura necesarias en la base de datos PostgreSQL para almacenar los eventos y los streams. No necesitas crear manualmente estas tablas: Marten gestiona la migración y el esquema por ti.

---

## Ejecutar la aplicación y revisar la base de datos

Una vez que hayas implementado la persistencia de eventos y llamado a `await session.SaveChangesAsync();`, puedes ejecutar tu aplicación desde la terminal con:

```bash
dotnet run
```

Esto ejecutará el código y Marten almacenará los eventos en la base de datos PostgreSQL.

### ¿Qué sucede en la base de datos?

- Marten creará automáticamente las tablas necesarias (por ejemplo, `mt_events`, `mt_streams`) si no existen.
- Los eventos que guardaste se almacenarán como filas en la tabla `mt_events`, en formato JSONB.
- Cada stream de eventos tendrá un identificador único y estará representado en la tabla `mt_streams`.

### ¿Cómo revisar los datos?

Puedes inspeccionar la base de datos usando una herramienta como pgAdmin, DBeaver o desde la terminal de PostgreSQL:

```sql
SELECT * FROM mt_events;
SELECT * FROM mt_streams;
```

Verás que los eventos se almacenan como documentos JSON, junto con información sobre el tipo de evento, el stream al que pertenecen y la secuencia.

> Así puedes comprobar que tu aplicación está generando y almacenando correctamente los eventos en la base de datos usando Marten.

### ¿Qué significan las columnas principales en `mt_events`?

Al consultar la tabla `mt_events`, verás varias columnas. Las más relevantes son:

- **id**: Identificador único del evento (UUID generado por Marten).
- **stream_id**: Identificador del stream/agregado al que pertenece el evento (por ejemplo, el id de la orden).
- **version**: Número de versión/secuencia del evento dentro del stream. El primer evento tiene versión 1, el segundo versión 2, etc.
- **timestamp**: Fecha y hora en que se guardó el evento en la base de datos.
- **tenant_id**: Identificador del tenant (multi-tenant), útil si tu aplicación maneja múltiples clientes o contextos. Si no usas multi-tenancy, suele ser null o un valor por defecto.
- **type**: Nombre del tipo de evento almacenado (por ejemplo, `OrdenCreada`).
- **data**: El contenido del evento en formato JSONB, con todos los datos del evento.

Estas columnas te permiten saber qué evento ocurrió, a qué agregado pertenece, cuándo ocurrió, su orden en la secuencia y el contenido exacto del evento.

> Así puedes auditar, depurar y reconstruir el estado de tus agregados a partir de los eventos almacenados.

---

## Sobre la tabla `mt_streams` y el tipado de streams

La tabla `mt_streams` almacena información sobre cada stream de eventos creado en Marten. Cada vez que inicias un nuevo stream (por ejemplo, una nueva orden de compra), Marten agrega una fila en esta tabla.

- **¿Qué guarda `mt_streams`?**
  - El identificador único del stream (por ejemplo, el Guid de la orden).
  - El tipo del stream (si se especifica al crearlo).
  - El estado y metadatos del stream.

- **¿Qué ocurre si se especifica el tipo al crear el stream?**
  - Si usas `StartStream<T>(...)` (por ejemplo, `StartStream<OrdenCompra>(...)`), Marten guarda el nombre del tipo en la columna `type` de la tabla `mt_streams`.
  - Esto permite a Marten asociar el stream directamente con el tipo de agregado, facilitando la reconstrucción automática del agregado y la consulta de streams por tipo.
  - Si no se especifica el tipo, la columna `type` puede quedar vacía o con un valor genérico, y Marten no podrá asociar el stream a un tipo de agregado de forma automática.

- **¿Por qué es importante tipar el stream?**
  - Tipar el stream mejora la trazabilidad y la integridad del modelo de dominio.
  - Permite a Marten optimizar operaciones como la reconstrucción del agregado (`AggregateStreamAsync<T>`) y la consulta de streams por tipo.

> En resumen: la tabla `mt_streams` es el registro central de todos los streams de eventos en tu sistema. Si tipas el stream al crearlo, Marten puede asociar el stream con el tipo de agregado y facilitar operaciones avanzadas sobre los eventos y agregados.

---

[⬅️ Volver a la sección anterior](./definicion-eventos.md)

[➡️ Siguiente sección: Construir el agregado](./construir-agregado.md)