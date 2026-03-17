# 🏷️ Definición de eventos

En Event Sourcing, los eventos son la fuente de la verdad. Representan hechos inmutables que ya ocurrieron en tu negocio.

## Ejercicio: Modelando tus primeros eventos

En este primer paso, definirás cómo se ven los eventos de una **Orden de Compra** en código C#.

### 1. ¿Qué debes hacer?

Abre el archivo `Program.cs` de tu proyecto `TallerMarten.OrdenCompra` y borra todo su contenido. Luego, escribe las siguientes definiciones de eventos:

```csharp
using System;

// Definición de eventos como registros (records) inmutables
public record OrdenCreada(Guid Id, string NumeroFactura);
public record ProductoAgregado(string Nombre, int Cantidad, decimal Precio);
```

### 2. Reto: ¡Ve un paso más allá!

El dominio de una **Orden de Compra** tiene más situaciones. Intenta definir otros `records` que representen estos cambios de estado:
- ¿Cómo llamarías al evento cuando se elimina un producto? (ej: `ProductoEliminado`)
- ¿Y cuando la orden se cancela o se marca como enviada?

> Escribe al menos dos eventos más por tu cuenta en `Program.cs`.

### 3. ¿Por qué usamos `record` en vez de `class`?

Usar `record` (en lugar de una `class` tradicional) es una de las mejores prácticas en .NET para Event Sourcing por tres razones:

1. **Inmutabilidad**: Un evento es un hecho que **ya ocurrió en el pasado**. El pasado no se puede editar. Los `record` facilitan la definición de objetos que no pueden cambiar una vez creados, evitando errores accidentales en tu lógica.
2. **Igualdad por Valor (Testing)**: A diferencia de las clases, dos registros son considerados **iguales si sus datos son iguales**. Esto es clave para las **Prueas Unitarias**: puedes comparar directamente el evento generado por tu sistema con uno esperado (`Assert.Equal(esperado, generado)`) sin tener que comparar propiedad por propiedad.
3. **Semántica de Dominio**: En Event Sourcing, un evento es una "foto" de un momento. Si tienes dos fotos idénticas, representan el mismo hecho, sin importar si están en distintos lugares de la memoria.

---

> [!TIP]
> **Dato técnico**: Las clases comparan por **referencia** (identidad en memoria), mientras que los records comparan por **valor** (los datos que contienen). Para un evento, lo que importa es el dato, no dónde está guardado en la memoria.


> [!IMPORTANT]
> Los eventos siempre deben nombrarse en **pasado** (`Creada`, `Agregado`, `Enviado`) porque representan algo que ya sucedió.

---

[⬅️ Volver a la sección anterior](./configuracion-inicial.md)

[➡️ Siguiente sección: Persistencia de eventos con Marten](./persistencia-eventos-marten.md)
