# 08 - Fronteras de Consistencia: El Agregado y su Raíz (Aggregate Root)
> 🎯 **Hacia dónde va:** El Aggregate Root es el guardián de las reglas de negocio y la frontera de consistencia donde, en Event Sourcing, se validan los comandos y se emiten los eventos que reconstruyen su estado.
> 📦 **Ejemplo de esta sección:** Carrito de compras (con factura e ítems).

Si abres casi cualquier proyecto de software web genérico hoy en día, verás que todas las propiedades de las clases tienen getters y setters públicos (`public string Estado { get; set; }`). Cualquier parte mínima del programa puede modificar en silencio la fecha límite de un pedido, alterar el estado de un carrito de compras o cancelar una transacción, todo directamente tocando la fila de la base de datos a través de sentencias ORM sueltas.

Esto se llama **"Modelo Anémico"**. En un modelo anémico los objetos no dictan reglas vitales de sí mismos; solo son baldes tontos de datos mientras las verdaderas reglas del negocio están dispersas en Services kilométricos. 

En **Domain-Driven Design (DDD)**, esta debilidad es la causante principal del colapso de lógicas de negocio complejas en producción. Para solucionar el caos, DDD impone una muralla inexpugnable: **Los Agregados**.

## 🧩 ¿Qué es un Agregado (Aggregate)?

Visualízalo en tu cabeza: No todas las clases en el modelo del negocio importan de manera independiente. Si imaginas a una persona que entra al hospital, esa persona tiene un historial médico. **Un Historial Médico no existe por sí solo, sólo tiene sentido si existe la Persona.** 

Un *Agregado* es exactamente eso en el software: un conjunto semántico de objetos tan dependientes entre sí para ser consistentes, que merecen ser encerrados y tratados estadísticamente como si fuesen UNA sola súper-entidad. 

## 🌳 El Guardaespaldas Absoluto: El Aggregate Root (Raíz)

Dentro de cada "Agregado", una (y solo una) de las entidades es elegida como el **Aggregate Root**. Es el guardia de seguridad, el jefe de operaciones exclusivo, la "puerta".

Las Reglas Reales: Todo sucede A TRAVÉS del Aggregate Root.

1.  **Impenetrabilidad:** Ninguna parte externa del sistema puede ir por detrás y tratar de interactuar, añadir o borrar objetos *dentro* de la caja fuerte saltándose a la Raíz. Si un cajero quiere añadir un ítem a la factura, no llama al `Insert` en la base de datos de Ítems; debe pedirle explícitamente a la Factura (Root) que procese el nuevo ítem `Factura.AnadirItem("Manzanas")`.
2.  **Consistencia Inmediata:** La Raíz es el lugar ideal para programar todas las validaciones complejas. Si llamas a `Factura.AnadirItem()`, el código dentro de la raíz puede reaccionar y validar cosas antes siquiera de permitir ese cambio. Ej: *"Si la factura ya superó los 10 items, no dejo añadir otro y detengo aquí mismo la transacción"*.
3.  **Vida o Muerte a la Vez:** Cuando el repositorio guarda cambios, debe guardar (o emitir eventos de) TODA LA CAJA fuerte (el Agregado entero). No guardamos un sub-ítem suelto. 

### En Código (Un Caso Real): Modelo Anémico vs Aggregate Root

#### ❌ El Error del Modelo Anémico (Puertas Abiertas)
Cualquier desarrollador en cualquier parte de la App podría hacer esto y causar desastres financieros con nuestro Carrito de compras:

```csharp
// Un Service externo altera "por las malas" las variables de los objetos:
var carrito = db.Carritos.Find(cartId);
var productoExterno = new Item { Precio = 50 };

// ⚠️ El Service altera la lista saltándose la autorización! 
carrito.Items.Add(productoExterno);
// ⚠️ El total matemático se daña porque el carrito ignoró este update directo.
db.SaveChanges(); // Consistencia corrompida.
```

#### ✅ La Solución: El Control Centralizado por La Raíz (Aggregate Root)

Para curarlo, blindamos las propiedades (Solo lectura pública) y obligamos a que el sistema "hable las intenciones" (Comandos) con los métodos de comportamiento.

```csharp
// Un Aggregate Root perfecto
public class CarritoDeCompras : AggregateRoot 
{
    public Guid Id { get; private set; } // Nadie lo modifica desde afuera.
    public decimal TotalDolares { get; private set; }

    // El negocio exige que la lista esté encapsulada (ReadOnlyCollection)
    // porque ni el mundo exterior ni EntityFramework tienen derecho de manipularla.
    private readonly List<OrdenItem> _items = new();
    public IReadOnlyCollection<OrdenItem> Items => _items.AsReadOnly();

    public CarritoDeCompras(Guid id) => Id = id;

    // EL COMPORTAMIENTO: (La forma oficial de que alteren la Root)
    public void AñadirAlCarrito(string nombre, decimal precioDolares)
    {
        // 1. REGLAS Y CONSISTENCIA: Si el carrito fue cerrado ayer, lanzo error:
        if (TotalDolares > 1000) 
            throw new Exception("Límite de la canasta de compras excedido bajo reglas antifraude.");

        // 2. MODIFICACIÓN AUTORIZADA: Agrego y modifico el total. 
        // Solo el Carrito deCompras sabe cómo hacerlo.
        _items.Add(new OrdenItem(nombre, precioDolares));
        TotalDolares += precioDolares;
        
        // 3. (Event Sourcing): Emitimos el evento de que esto ocurrió.
        // Emit(new ProductoAgregadoAlCarrito(nombre, precio));
    }
}
```

Así el desarrollador no interactúa con variables, sino con un experto que sabe las reglas de su propio terreno.

### Event Sourcing: Dependiente de un Aggregate Root

Todo lo que enseñamos durante este manual cobra sentido en **Event Sourcing**, donde ni siquiera almacenaremos el "Estado Físico Actual" del Agregado en una tabla plana. Como la única forma de que un Agregado recupere su memoria (`TotalDolares` y los iteradores) y pueda aplicar reglas es sabiendo qué le había pasado antes, guardaremos todos los `ProductoAgregadoAlCarrito` en su propio Archivero Histórico Exclusivo y los recargaremos en orden antes de intentar validar si podemos o no agregar otro producto.

> [!IMPORTANT]
> **Dos reglas de oro del Agregado que casi nunca se enseñan:**
> 1. **Una transacción = un agregado.** Cada operación modifica **un solo** agregado de forma atómica. Si una acción "necesita" cambiar dos agregados a la vez, casi siempre el diseño está mal: divídela en dos comandos y coordínalos con eventos/consistencia eventual (o una *saga*).
> 2. **Dimensiona el agregado por sus invariantes, no por sus datos.** Hazlo **pequeño**: solo lo que debe mantenerse consistente *junto* en la misma transacción. El error clásico es el "God aggregate" (un Carrito con 10.000 ítems) que serializa toda la concurrencia y se vuelve lento. Si dos partes no comparten una regla, probablemente son **dos** agregados.

---
[⬅️ Volver a la sección anterior](./07-lenguaje-ubicuo.md) | [➡️ Siguiente sección: Eventos de Dominio](./09-eventos-de-dominio.md)
