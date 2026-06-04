# 03 - La Magia del Enrutamiento: Polimorfismo y Dynamic Dispatching

En la arquitectura basada en eventos, constantemente tenemos listas masivas de objetos "crudos". Por ejemplo, un historial de vida puede contener dentro muchos tipos distintos de eventos:

```csharp
List<object> historia = new() 
{
    new PersonaNacida("Jhon"),
    new CumpleañosCelebrado(),
    new MudanzaRegistrada("Bogotá")
};
```

Cuando queremos procesar cada uno de estos eventos para actualizar el estado actual, el primer instinto de todo programador es armar un bloque gigante de comprobaciones usando `if` o `switch`:

## 🍝 El Código Espagueti (Acoplamiento Alto)

```csharp
public void Load(IEnumerable<object> eventos)
{
    foreach (var ev in eventos)
    {
        // ❌ Esto viola el principio Abierto/Cerrado (OCP)
        if (ev is PersonaNacida n) 
        { 
            Nombre = n.Nombre; 
        }
        else if (ev is CumpleañosCelebrado) 
        { 
            Edad++; 
        }
        else if (ev is MudanzaRegistrada m) 
        { 
            Ciudad = m.NuevaCiudad; 
        }
        // ... imagina 50 eventos más aquí
    }
}
```

Cada vez que el negocio invente un nuevo evento, tendrás que abrir el motor nuclear (`Load`) y modificar esa cadena interminable. 

## 🎩 La Magia: Dynamic Dispatching

C# nos ofrece una forma de "engañar" al estricto compilador de tipos para enrutar los métodos automáticamente en **tiempo de ejecución (Runtime)** en lugar de obligarlo a resolverlos en **tiempo de compilación**. 

Hacemos esto rompiendo las reglas limpiamente con la palabra reservada `dynamic`.

```csharp
// El motor base limpio que JAMÁS cambia
public void Load(IEnumerable<object> eventos)
{
    foreach (var ev in eventos)
    {
        ((dynamic)this).Apply((dynamic)ev);
    }
}
```

### ¿Cómo funciona bajo el capó?

1. El bucle toma el primer objeto (ej: `PersonaNacida`) que ante los ojos del compilador era un genérico `object`.
2. Al castearlo a `(dynamic)ev`, le decimos a C#: *"Apaga tus validaciones tipadas en este momento"*.
3. El programa sigue ejecutándose, examina la memoria real y dice: *"Aha! Este objeto es realmente del tipo Exacto `PersonaNacida`. Voy a buscar si existe, en cualquier parte de LA CLASE ACTUAL (`this`), un método que se llame `Apply` y reciba exactamente un `PersonaNacida`."*
4. Si lo encuentra, lo llama de forma transparente. ¡Sin condicionales!

### El Resultado en tus Clases (Limpio y Aislado)

Ahora cada Agregado (`Persona`) añade nuevos comportamientos de forma aislada e infinita sin alterar para nada el motor base:

```csharp
public class Persona : AggregateRoot
{
    // ... propiedades
    
    // ✅ Cumple el principio Abierto/Cerrado (OCP de código Limpio)
    // C# lo encontrará automáticamente por el tipo de parámetro.
    protected void Apply(PersonaNacida n) => Nombre = n.Nombre;
    protected void Apply(CumpleañosCelebrado e) => Edad++;
    protected void Apply(MudanzaRegistrada m) => Ciudad = m.NuevaCiudad;
}
```

> [!TIP]
> Mantener estos métodos `Apply` marcados como **`protected`** o **`private`** es una excelente práctica. Evitamos contaminar la "API pública" de la clase ante el mundo exterior. `dynamic` tiene poderes especiales y logrará invocarlos aunque estén ocultos, protegiendo así el encapsulamiento de nuestro dominio.

---

### Cierre de la Fase 1
Acabamos de pulir las herramientas fundamentales de C# que requiere un arquitecto de Event Sourcing: Inmutabilidad (Records), Moldes de Herencia (Abstract Classes) y Enrutamiento Eficiente (Dynamic). 

Es hora de saltar a cómo estructuramos el sistema en su totalidad.

---
[⬅️ Volver a la sección anterior](./02-interfaces-vs-abstract-classes.md) | [➡️ Siguiente Fase: El Patrón Comando (Patrones Estructurales)](./04-el-patron-comando.md)
