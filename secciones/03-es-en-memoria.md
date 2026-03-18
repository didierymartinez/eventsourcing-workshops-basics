# 03 - Guardando la historia en el código

Ahora que tenemos nuestro lienzo en blanco, vamos a empezar a anotar la historia de una vida usando C#.

## 🎯 El Objetivo
Imagina que quieres saber exactamente qué edad tenía Juan en el año 2005. 
**No puedes guardar una variable fija llamada `Edad = 34`**, porque esa variable solo te dice el "ahora". Si la sobrescribes cada año, destruyes el pasado.

¿Cómo podrías calcular su edad en cualquier punto del tiempo sin guardar el número final?

---

## 1. El Protagonista e Identidad
En Event Sourcing, todo gira en torno a una identidad única.

```csharp
// El ancla de nuestra historia
var idPersona = Guid.NewGuid();
```

Esto representa la **Identidad**. Es como tu huella digital o tu número de pasaporte: no cambia nunca, sin importar cuántos hechos ocurran en tu vida.

---

## 2. Definiendo los Hitos (Hechos)

Antes de escribir código, debemos decidir qué momentos de la vida de Juan afectan su "estado" (como su edad o su ubicación). En el mundo profesional, se usa una técnica llamada **Event Storming** para identificar qué eventos son realmente importantes para el negocio.

### ¿Record o Clase?
Para representar estos hechos, en C# usamos **Records** en lugar de clases tradicionales. ¿Por qué?

1.  **Inmutabilidad**: Un hecho del pasado (como nacer) no puede cambiarse. Un `record` es inmutable por defecto.
2.  **Igualdad por valor**: Dos hechos son iguales si sus datos son iguales, no si son la misma instancia en memoria.

```csharp
// La vida empieza con un hecho fundacional
public record PersonaNacida(Guid Id, string Nombre, DateTime FechaNacimiento);

// Y continúa con hitos que cambian su estado
public record CumpleañosCelebrado(Guid Id, DateTime Fecha);
```

> [!TIP]
> Solo anotamos los eventos que **importan**. Que Juan desayunó hoy quizás no sea relevante para nuestro sistema, pero que cumplió años sí, porque afecta su edad.

---

## 3. La Biografía (El Stream)

¿Cómo guardamos estos hitos para no perder el orden? Usamos una **Lista**.

```csharp
// Nuestra biografía es una secuencia cronológica
var biografia = new List<object>();

biografia.Add(new PersonaNacida(idPersona, "Juan", new DateTime(1990, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona, new DateTime(1991, 5, 10)));
biografia.Add(new CumpleañosCelebrado(idPersona, new DateTime(1992, 5, 10)));
```

**¿Por qué una lista?** Porque un diario solo tiene sentido si se lee en orden. A esta secuencia ininterrumpida de hechos la llamamos **Stream** (Flujo). El Stream es la "Fuente de la Verdad": si quieres saber quién es Juan, lees su Stream.

---

## 4. Resolviendo el desafío (El Agregado)

Volvamos al objetivo: ¿Cómo sabemos su edad actual? 
No la leemos de una propiedad; la **reconstruimos** procesando su biografía.

```csharp
int edadCalculada = 0;

foreach (var hito in biografia)
{
    if (hito is CumpleañosCelebrado)
    {
        edadCalculada++;
    }
}

Console.WriteLine($"Juan tiene {edadCalculada} años.");
```

### El Descubrimiento: Aggregate Root y Aggregate

Acabas de crear un sistema que protege la historia. En la teoría de DDD, esto se divide así:

1.  **Aggregate Root (La Raíz)**: Es el "Jefe". En nuestro caso, es la clase o el concepto de **Persona** que posee el ID. Es a quien le enviamos las órdenes.
2.  **Aggregate (El Agregado)**: Es el conjunto completo. Es **Juan (Root) + Su Biografía (Stream) + La lógica de conteo**. 

> [!IMPORTANT]
> El **Agregado** es la frontera que garantiza que los datos sean coherentes. Por ejemplo, la lógica del Agregado no debería permitir que Juan celebre un cumpleaños ANTES de haber nacido. Esa regla de negocio la protege el Agregado.

---

[⬅️ Volver a la sección anterior](./02-primer-proyecto.md)

[➡️ Siguiente sección: El gran problema](./04-el-problema.md)
