# 05 - Abstraer el Archivero: El Patrón Repositorio (Repository)

En la sección anterior, vimos al `RegistrarMatrimonioHandler` ir a un cajón llamado `IPersonaRepository` para sacar a Jhon y luego para guardarlo. 

Ese es el **Patrón Repositorio**. Es un escudo arquitectónico cuyo propósito fundamental es que la capa de reglas de negocio **no se entere de que nosotros usamos una tabla en SQL Server, un JSON o colecciones en RAM**.

## 🛑 El Anti-patrón Endémico

Si expones Entity Framework o conexiones a BD directamente a tu Handler (o a tu Controlador), acoplas tu negocio para siempre a la marca de esa base de datos:

```csharp
// ❌ Pésimo: El Handler sabe que existe EntityFramework y maneja SQL!
public void Handle(RegistrarMatrimonio comando)
{
    var persona = _dbContext.Personas.FirstOrDefault(p => p.Id == comando.PersonaId);
    persona.NombrePareja = comando.NombrePareja;
    _dbContext.SaveChanges(); // Si cambias EF por Mongo, esto peta en 10,000 líneas
}
```

## 🛡️ La Solución Clásica: Repositorio Lógico vs Store Físico

En DDD ortodoxo, solucionamos esto partiendo la responsabilidad en dos lugares conceptuales completamente distintos.

### 1. El Repositorio Lógico (El Idealista)
Es una simple y tonta interfaz de métodos que reside en tu capa limpia de core/dominio. Habla únicamente en términos del negocio.

```csharp
// Solo habla nuestro lenguaje: Dame una Persona. Guarda una Persona.
public interface IPersonaRepository
{
    Persona Get(Guid id);
    void Save(Persona entidad);
}
```

Tus Command Handlers solo ven, usan e inyectan esta interfaz pura. Jamás ven a quién trabaja por detrás.

### 2. El Store Físico / Persistencia (El Obrero de Infraestructura)
Aquí reside la clase de código sucio que de verdad implementa SQL, APIs de red o archivos de texto plano. Ocupa la capa externa del sistema ("Infraestructura").

```csharp
// ¡Nadie en el núcleo del sistema sabe que esta clase existe!
public class SqlPersonaRepository : IPersonaRepository
{
    private readonly DbContext _context;

    public Persona Get(Guid id)
    {
        // Lógica sucia y amarrada a SQL Server
        return _context.ExecuteSql("SELECT * FROM..."); 
    }

    public void Save(Persona entidad)
    {
        // Lógica sucia y transaccional 
         _context.ExecuteSql("UPDATE..."); 
    }
}
```

> [!TIP]
> Si mañana el cliente decide cambiar SQL Server por MongoDB, los desarrolladores solo tienen que crear una clase `MongoPersonaRepository : IPersonaRepository`. ¡No tendremos que reescribir ni una coma de nuestros Handlers ni Controladores! A esto se le conoce como **Dipendency Inversion Principle (La "D" de SOLID)**.

### Event Sourcing rompe el paradigma
Es vital que entiendas el Repositorio Clásico porque en el universo moderno de Event Sourcing en .NET, vas a presenciar una **herejía productiva**.

Veremos librerías como `Marten` en el workshop principal que te otorgan una superinterfaz (como `IDocumentSession`) que es al mismo tiempo repositorio lógico ("dame este agregado") y motor de persistencia ("yo me ocupo de generar el SQL nativo contra PostgreSQL subyacente"). 

Muchos puristas detestan inyectar clases de frameworks directamente en los Handlers porque viola el DDD clásico. Pero la agilidad para rehidratar agregados a partir de eventos que entregan herramientas como Marten o EventStoreDB suele justificar eliminar las clases intermedias en muchos escenarios de industria.

---
[⬅️ Volver a la sección anterior](./04-el-patron-comando.md) | [➡️ Siguiente sección: Inyección de Dependencias](./06-inyeccion-de-dependencias.md)
