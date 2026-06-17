# 03 - Despacho de eventos: `switch` tipado vs `dynamic` (y por qué importa)
> 🎯 **Hacia dónde va:** Reconstruir un agregado exige recorrer su historial de eventos y aplicar cada uno; aquí aprendes cómo se enruta cada evento a su método `Apply` y por qué en producción ese despacho se genera como código (no `dynamic` a mano).

En la arquitectura basada en eventos, constantemente tenemos listas masivas de objetos "crudos". Por ejemplo, un historial de vida puede contener dentro muchos tipos distintos de eventos:

```csharp
List<object> historia = new() 
{
    new PersonaNacida("Jhon"),
    new CumpleañosCelebrado(),
    new PersonaMudada("Bogotá")
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
        else if (ev is PersonaMudada m) 
        { 
            Ciudad = m.NuevaCiudad; 
        }
        // ... imagina 50 eventos más aquí
    }
}
```

Ese `if/else` (o un `switch` equivalente) **funciona y es seguro** —el compilador verifica cada tipo—, pero es verboso y cada nuevo evento te obliga a tocar el motor. ¿Hay formas más limpias? Sí, dos. Y conviene conocer el **tradeoff** de cada una.

## Opción A — `dynamic` dispatch (elegante… pero con costo real)

C# te permite **apagar la verificación de tipos** del compilador y resolver el método en **tiempo de ejecución** con la palabra reservada `dynamic`:

```csharp
public void Load(IEnumerable<object> eventos)
{
    foreach (var ev in eventos)
        ((dynamic)this).Apply((dynamic)ev);   // resuelve en runtime cuál Apply llamar
}

// Y en la clase, un overload de Apply por evento (se ve muy OCP):
public void Apply(PersonaNacida n)       => Nombre = n.Nombre;
public void Apply(CumpleañosCelebrado e) => Edad++;
public void Apply(PersonaMudada m)       => Ciudad = m.NuevaCiudad;
```

Se ve mágico: agregas un evento nuevo añadiendo un overload `Apply`, **sin tocar `Load`**. Pero esa magia tiene un precio que debes conocer:

> [!WARNING]
> 🪤 **Ojo con `public`.** Esos `Apply` están en `public` a propósito. Si `Load` vive en una **clase base** (`AggregateRoot`) y declaras los `Apply` como `protected`/`private` en la clase hija, en runtime obtendrás `RuntimeBinderException: ... is inaccessible due to its protection level`. La razón: `dynamic` respeta la accesibilidad **desde donde está escrita la llamada** (la clase base), y `protected` solo es visible para la clase declarante y sus **subclases** — no para su superclase. Regla: lo que despacha el motor de la base debe ser `public` (o `internal` en el mismo proyecto). *(Con frameworks como Marten es distinto: usan reflexión y sí acceden a métodos no públicos.)*

> [!WARNING]
> **El costo de `dynamic` (según la doc oficial de Microsoft):**
> - *"an object of type `dynamic` **bypasses static type checking**"* — pierdes la red de seguridad del compilador.
> - *"**Overload resolution occurs at run time**"* — resolver qué `Apply` llamar se hace en cada ejecución, vía el **DLR (Dynamic Language Runtime)** → más lento que una llamada directa.
> - Si olvidas escribir el `Apply` de un evento, **no hay error de compilación**: revienta en producción con un `RuntimeBinderException`.
>
> 📎 Referencia: [Using type dynamic — Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/using-type-dynamic).

## Opción B — `switch` con pattern matching (la recomendada para escribir a mano)

Desde C# 8, el `switch` con patrones te da un dispatch **limpio Y con seguridad de compilación** (sin el riesgo de `dynamic`):

```csharp
public void Load(IEnumerable<object> eventos)
{
    foreach (var ev in eventos) Apply(ev);
}

private void Apply(object ev)
{
    switch (ev)
    {
        case PersonaNacida n:       Nombre = n.Nombre; break;
        case CumpleañosCelebrado:   Edad++; break;
        case PersonaMudada m:       Ciudad = m.NuevaCiudad; break;
        // default: podrías ignorar, loguear o lanzar para eventos desconocidos
    }
}
```

Sí, tocas el `switch` al añadir un evento — pero a cambio el **compilador te respalda**, es **más rápido** (sin DLR) y no hay sorpresas en runtime. Para código que escribes a mano, **prefiere esto sobre `dynamic`**.

## En producción: ni una ni otra a mano

Aquí está el remate: las librerías serias (como **Marten**, que verás en el workshop principal) **no usan `dynamic` en la ruta caliente** —por el costo que vimos— ni te hacen escribir el `switch`. **Descubren** tus métodos `Apply` al arrancar y **generan/compilan** el dispatch (código rápido y verificable). Es el concepto *reflexión vs generación de código* (§22 del principal): la "magia" se cambia por **código generado que sí puedes leer**.

> [!TIP]
> Mantén tus métodos `Apply` como `protected`/`private` para no contaminar la API pública del agregado. (Tanto `dynamic` como los frameworks de codegen logran invocarlos aunque estén ocultos.)

---

### Cierre de la Fase 1
Acabamos de pulir las herramientas fundamentales de C# que requiere un arquitecto de Event Sourcing: Inmutabilidad (Records), Moldes de Herencia (Abstract Classes) y Enrutamiento de eventos (`switch` tipado vs `dynamic`, y por qué producción genera código). 

Es hora de saltar a cómo estructuramos el sistema en su totalidad.

---
[⬅️ Volver a la sección anterior](./02-interfaces-vs-abstract-classes.md) | [➡️ Siguiente Fase: El Patrón Comando (Patrones Estructurales)](./04-el-patron-comando.md)
