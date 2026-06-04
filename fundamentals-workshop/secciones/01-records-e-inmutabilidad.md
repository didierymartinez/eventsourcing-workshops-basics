# 01 - El Peligro del Transporte: Records e Inmutabilidad

Si has programado en C# clásico o en arquitecturas de tres capas, estás acostumbrado a usar **Clases** (`class`) para todo: para conectarte a la base de datos, para representar un Botón, y también para transportar datos entre métodos (los famosos DTOs).

Sin embargo, cuando empezamos a separar la aplicación en partes puras (como en Event Sourcing, DDD o Microservicios), usar una `class` para transportar intenciones o hechos históricos es extremadamente peligroso.

## 🚨 El Problema: Datos Mutables en Tránsito

Imaginemos que enviamos una orden al sistema usando una clase tradicional:

```csharp
public class SolicitarMudanza
{
    public Guid PersonaId { get; set; }
    public string NuevaCiudad { get; set; }
}

// En el origen (Ej. un Controlador API)
var orden = new SolicitarMudanza { PersonaId = miUsuario, NuevaCiudad = "Madrid" };

// ... el mensajero se lleva la orden...
// ¿Qué pasa si algún interceptor, middleware o desarrollador despistado hace esto en medio del camino?
EnrutarOrden(orden); 

void EnrutarOrden(SolicitarMudanza req)
{
    // Accidentalmente o por "lógica sucia", alguien muta la orden en tránsito.
    req.NuevaCiudad = "Murcia"; 
    
    ValidarYGuardar(req);
}
```

La intención original del usuario fue alterada. Al llegar a su destino, el empleado que procesa la mudanza ve "Murcia" y cree que es la verdad.

En el mundo físico, cuando llenas un formulario de aduanas con un bolígrafo, se sella. Si el mensajero lo borra con típex y escribe otra cosa, es un delito. **En arquitectura, tus "Formularios de Intención" (Comandos) y tus "Hechos Ocurridos" (Eventos) deben ser inmutables.** 

## 🛡️ La Solución en C#: Los Records

Desde C# 9, se introdujo la palabra reservada `record`. Están diseñados específicamente para ser inmutables y comportarse como puros contenedores de valor.

Así es como reescribimos nuestra orden:

```csharp
// La sintaxis corta de un Record
public record SolicitarMudanza(Guid PersonaId, string NuevaCiudad);
```

### Beneficio 1: Inmutabilidad por defecto

Si un desarrollador intenta alterar este formulario en medio de la aplicación, el compilador lo detendrá inmediatamente:

```csharp
var orden = new SolicitarMudanza(miUsuario, "Madrid");

// ERROR DE COMPILACIÓN: Init-only property or indexer 'SolicitarMudanza.NuevaCiudad' 
// cannot be assigned to -- it is read-only.
orden.NuevaCiudad = "Murcia"; 
```

### Beneficio 2: Igualdad por Valor (Structural Equality)

Con las clases normales (Referencia), dos objetos son diferentes aunque tengan los mismos datos, porque viven en espacios de memoria distintos.
Con los Records (Valor), dos registros son iguales si sus datos son idénticos.

```csharp
var orden1 = new SolicitarMudanza(id, "Madrid");
var orden2 = new SolicitarMudanza(id, "Madrid");

// Si fueran clases, esto sería false (distinta referencia de memoria)
// Al ser records, esto es TRUE (sus datos son exactamente iguales)
bool sonIguales = orden1 == orden2; 
```

Esto es brutalmente útil en Pruebas Unitarias para verificar si "este evento generado es igual al evento esperado".

### Beneficio 3: Creación No-Destructiva (`with`)

Si por alguna razón legítima necesitas una versión modificada del formulario, no puedes alterarlo. Tienes que **fotocopiarlo** y cambiar un dato en la copia. C# nos da la expresión `with` para esto.

```csharp
var formularioBase = new SolicitarMudanza(id, "Madrid");

// Crea un NUEVO objeto, copiando todo, pero actualizando la ciudad
var nuevoFormulario = formularioBase with { NuevaCiudad = "Barcelona" };
```

---

## 🎯 Regla de Oro del Arquitecto

A partir de este momento, adopta la siguiente heurística estructural estricta:

1. **Usa `class`** para entidades que tienen comportamiento, ciclo de vida continuo y cambian con el tiempo (Ej. Tu Repositorio, tu `AggregateRoot`, tu Gestor de Conexiones).
2. **Usa `record`** para simples "sobres de datos" inmutables que viajan por el sistema (Comandos, Eventos de Dominio, DTOs de Entrada/Salida).

---

[➡️ Siguiente sección: Interfaces vs Clases Abstractas](./02-interfaces-vs-abstract-classes.md)
