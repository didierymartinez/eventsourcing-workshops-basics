# 02 - El Contrato vs El Molde: Interfaces vs Clases Abstractas
> 🎯 **Hacia dónde va:** Saber cuándo usar `interface` (contrato para desacoplar Handlers y EventStores) o `abstract class` (motor reutilizable del `AggregateRoot`) es lo que te permitirá estructurar el polimorfismo de tu sistema de Event Sourcing.

En C#, tenemos dos herramientas principales para estandarizar objetos: las **Interfaces** (`interface`) y las **Clases Abstractas** (`abstract class`).

Muchos tutoriales básicos enseñan que "son casi lo mismo, pero la interfaz no tiene código". En la arquitectura moderna y en Event Sourcing, la diferencia es profunda y define cómo estructuramos el polimorfismo.

## 📜 La Interfaz: El Contrato Puro

Una interfaz (`interface`) es como un requerimiento legal de contratación. Dice **QUÉ** se debe hacer, pero no le importa **CÓMO** se hace.

```csharp
public interface ICommandHandler<in TCommand>
{
    void Handle(TCommand command);
}
```

El modificador `in` de `TCommand` indica **contravarianza**: permite usar un handler de un tipo de comando base donde se espera uno de un tipo derivado (la flexibilidad fluye "hacia adentro", de lo general a lo específico).

- **Uso Estructural:** La usamos para lograr **Desacoplamiento (Loose Coupling)**. 
- **El escenario ideal:** Cuando diferentes clases hacen el mismo trabajo conceptual pero su implementación física es drásticamente distinta. Por ejemplo, `InMemoryEventStore` guarda datos en un diccionario, mientras que `PostgresEventStore` guarda datos haciendo llamados de red a un motor SQL. Comparten el contrato (`IEventStore`), pero no comparten ni una sola línea de código útil entre ellos.

> [!NOTE]
> **Cuidado con el mito "la interfaz no tiene código".** Desde **C# 8** las interfaces **sí pueden** traer implementación (*default interface methods*). Entonces, ¿cuál es la diferencia *real* con una clase abstracta? Dos cosas: (1) una clase abstracta puede tener **estado** (campos, propiedades con backing field), una interfaz no; (2) solo puedes heredar de **una** clase base, pero implementar **muchas** interfaces. La elección no es "código sí/no", es **estado + herencia única (abstracta)** vs **contrato múltiple (interfaz)**.

## 🏗️ La Clase Abstracta: El Molde (Reuso de Comportamiento)

Una clase abstracta (`abstract class`) es un **molde a medio terminar**. Dice **QUÉ** hay que hacer, pero también te regala el código de **CÓMO** resolver la mitad de las cosas genéricas.

```csharp
public abstract class AggregateRoot
{
    public Guid Id { get; protected set; }

    // El motor genérico (Esto es oro puro, código real reutilizable)
    public void Load(IEnumerable<object> eventos)
    {
        foreach (var ev in eventos)
        {
            ((dynamic)this).Apply((dynamic)ev);
        }
    }

    // Parte incompleta: El hijo debe obligatoriamente saber aplicar sus eventos
    // (A través de métodos ocultos vía convención, como vimos)
}
```

- **Uso Estructural:** La usamos primordialmente para **Reuso de Código Base (Inheritance)** y para aplicar el **Patrón Template Method**.
- **El escenario ideal:** En Event Sourcing, todas nuestras entidades (Persona, CuentaBancaria, Vehículo) van a necesitar recorrer eventos uno por uno para reconstruirse. ¿Te imaginas copiar y pegar ese `foreach` 100 veces por toda tu app usando una `interface IAggregateRoot`? Sería una pesadilla de mantenimiento.

### ⚠️ El Límite de las Clases Abstractas
C# **no soporta herencia múltiple**. Una clase puede firmar 50 contratos (implementar múltiples interfaces), pero solo puede tener un único padre biológico (heredar de una sola clase base).

Por eso, `abstract class` es una herramienta de uso muy calculada. Resérvala solo para cuando el comportamiento a heredar es el "motor fundamental" de la clase hija (En nuestro caso: Un `AggregateRoot`).

---

## 🎯 Regla de Oro del Arquitecto

1. **Usa `interface`** para inyectar dependencias y ocultar la implementación (Ej. Puertos y Adaptadores, Infraestructura, Handlers, Servicios Externos). Todo lo que vaya en el parámetro de un constructor debería ser una interfaz.
2. **Usa `abstract class`** exclusivamente en la capa de Dominio, cuando necesitas estandarizar el "motor interno" de un grupo de familias de objetos y evitar el Copy/Paste de algoritmos core.

---

[⬅️ Volver a la sección anterior](./01-records-e-inmutabilidad.md) | [➡️ Siguiente sección: Genéricos y restricciones (`where T`)](./02b-genericos-y-restricciones.md)
