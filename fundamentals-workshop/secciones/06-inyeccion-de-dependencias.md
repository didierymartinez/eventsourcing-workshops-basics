# 06 - Ensamblando el Sistema: Inyección de Dependencias (DI)
> 🎯 **Hacia dónde va:** La DI es el motor que ensambla automáticamente Handlers, Repositorios y EventStores con sus ciclos de vida correctos; dominarla evita las trampas de scopes (captive dependency) que verás en Cosmos en el workshop principal.

En las Secciones 04 y 05 construimos un `RegistrarMatrimonioHandler` que no sabe conectarse a la base de datos y solo exige una interfaz `IPersonaRepository` en su constructor.

Pero la cruda realidad de programación orientada a objetos es que no puedes compilar `new RegistrarMatrimonioHandler()` si no le pasas un objeto real en el paréntesis. Alguien, en algún lugar oscuro y central de la aplicación, tiene que saber crear las conexiones físicas y ensamblar todo el lego:

```csharp
// Acoplamiento Físico Manual (El infierno de instanciar)
var dbContext = new PostgresContext("host=localhost;...");
var dirtyRepo = new SqlPersonaRepository(dbContext);
var cleanHandler = new RegistrarMatrimonioHandler(dirtyRepo);

// Ah, y si el repositorio ahora necesita un FileLogger... prepara los pañuelos para refactorizar.
```

## 🛎️ El Recepcionista (El Contenedor IoC)

Para resolver ese caos de ensamblaje masivo, los frameworks modernos como .NET Core traen integrado el motor de **Inyección de Dependencias (DI)**. Nosotros lo visualizamos como un Recepcionista ultra-eficiente. 

Funciona de la siguiente forma:

### Fase 1: El Registro (Enseñar al Recepcionista)
Apenas arranca tu aplicación en el `Program.cs`, abres "El Libro" (`IServiceCollection`) y creas manuales de instrucciones sobre cómo y con qué clase real se resuelvan las interfaces.

```csharp
var services = new ServiceCollection();

// "Estimado Recepcionista: cada vez que le pidan un IPersonaRepository... 
// por debajo entrégueles y fabrique una instancia sucia de SqlPersonaRepository"
services.AddTransient<IPersonaRepository, SqlPersonaRepository>();

// "Ah, y registre la existencia de mis Handlers también"
services.AddTransient<RegistrarMatrimonioHandler>();
```

### Fase 2: Construir el Proveedor (Abrir la Oficina)
```csharp
var proveedor = services.BuildServiceProvider(); // ¡La oficina abre sus puertas!
```

### Fase 3: La Resolución Mágica Automática (Suministro)
Cuando un Controlador de API recibe un request web de un usuario pidiendo "Registrar Matrimonio", el framework web le pide al recepcionista que consiga al manejador:

```csharp
// Le pedimos la clase limpia.
var handlerMagico = proveedor.GetRequiredService<RegistrarMatrimonioHandler>();
```

**Aquí sucede la magia absoluta de C#:** 
El Recepcionista lee el constructor del `Handler`. Ve que necesita `IPersonaRepository`. El Recepcionista busca en su libro de maestría, y ve que tiene las instrucciones para armar el `SqlPersonaRepository`. Si ese SQL a su vez necesitara un SqlConnection, el recepcionista baja tres niveles de profundidad para instanciar todos los legos anidados, inyectarlos de abajo hacia arriba, y nos entrega un `handlerMagico` completo listo para presionar play.

Nosotros NUNCA tocamos un `new`.

## ⏳ La Tercera Dimensión: Los Ciclos de Vida (Scopes)

Cuando el Recepcionista fabrica los objetos de sus libros, necesita saber "cuánto tiempo debe dejarlos vivir en la memoria de la RAM" y "cada cuánto debe hacer una fotocopia nueva".

Hay 3 Scopes dorados en .NET Core:

1. **Transient (`AddTransient`)**: El recepcionista fabrica un clon completamente nuevo *cada vez que se lo pidas*. Ideal para objetos ligeros, como nuestros Command Handlers abstractos. No tienen estado persistente.
2. **Singleton (`AddSingleton`)**: El recepcionista crea un solo objeto la *primera* vez, lo congela y entrega temporalmente en la RAM compartiéndolo para el resto de la eternidad con toda la aplicación y a todos los que llamen de la red. Ideal para Cachés Globales de configuración. 💣 **PELIGRO:** Nunca pongas una conexión a Base de Datos abierta en Singleton (condición de carrera y desborde).
3. **Scoped (`AddScoped`)**: El equilibrio de facto web. El recepcionista crea un objeto nuevo *cada vez que recibes un Request HTTP de la red*, lo comparte entre todo el procesamiento de ESA SOLA SOLICITUD de usuario y lo destruye para siempre apenas mandas la respuesta (Response HTTP) final al navegador. Todo lo relacionado a Bases de datos y Repositorios cae aquí por default.

---

> [!WARNING]
> **Dos trampas clásicas de los scopes/DI (que casi nadie te cuenta):**
> 1. **Captive dependency:** si registras un servicio `Scoped` (o `Transient`) y lo inyectas dentro de un `Singleton`, el singleton **captura la primera instancia para siempre** → el `Scoped` deja de ser "por request" y arrastra estado viejo o conexiones muertas. (En Cosmos, este bug se manifiesta como un `TenantId` por defecto — lo veremos en el principal.)
> 2. **Service Locator (anti-patrón):** pedir `proveedor.GetRequiredService<X>()` *por todo el código* esconde las dependencias reales y rompe la testabilidad. La regla: las dependencias se piden **por constructor**; el `GetRequiredService` solo es válido en el **Composition Root** (el único lugar donde se arma el grafo, normalmente el arranque).

### Cierre de la Fase 2
Has masterizado la separación estructural pura. Repositorio = Escudo contra DBs. Comandos = Sobres Inmutables de intención. DI = Nuestro Robot automático de Ensamblaje. 

En la siguiente etapa, soltaremos la estructura técnica para sumergirnos profundamente en el mindset del analista de negocios. Entramos a la sala de Domain Driven Design.

---
[⬅️ Volver a la sección anterior](./05-el-patron-repositorio.md) | [➡️ Siguiente Fase: Lenguaje Ubicuo (DDD)](./07-lenguaje-ubicuo.md)
