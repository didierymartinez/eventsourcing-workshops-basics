# 06 - El flujo de vida: Manejando múltiples biografías

Hasta ahora, nuestro sistema es perfecto... si Jhon fuera la única persona en el mundo. Pero la realidad es que una aplicación real maneja miles de biografías al mismo tiempo.

## 🚀 El Problema: La oficina crece

Imagina que ahora no solo tenemos a Jhon, sino también a **Pedro** y **María**. Todos comparten el mismo "baúl" (la lista de eventos), pero cada uno tiene su propia historia.

Si intentamos usar lo que sabemos hasta ahora, nuestro `Program.cs` se vería así:

```csharp
// El baúl lleno de hitos de TODOS
var baulGeneral = new List<object>();

// 1. Queremos cargar a Pedro (ID: 123)
var eventosDePedro = baulGeneral.Where(e => ((dynamic)e).PersonaId == Guid.Parse("123")); 
var pedro = new Persona(eventosDePedro);

// 2. Queremos cargar a María (ID: 456)
var eventosDeMaria = baulGeneral.Where(e => ((dynamic)e).PersonaId == Guid.Parse("456"));
var maria = new Persona(eventosDeMaria);

// 3. ¡Qué desorden! 
// Cada vez que necesitemos a alguien, tenemos que filtrar manualmente por ID.
```

Este enfoque tiene dos problemas graves:
1. **Fugas de lógica**: El código que sabe "cómo se busca a alguien" está desparramado por todo el programa.
2. **Riesgo**: Es muy fácil equivocarse y cargar los eventos de Pedro en el objeto de Jhon.

---

## 🌊 La Solución: El EventStream

Para solucionar esto, creamos una herramienta que se encargue de "gestionar el flujo" de una sola persona a la vez. A esta herramienta la llamamos **EventStream**.

El **EventStream** es como un **archivador personal**. Tú le dices qué ID quieres manejar, y él se encarga de buscar las páginas correctas y entregarte al protagonista ya reconstruido.

### 🛠️ Paso 1: Creando el archivador genérico

Vamos a crear una clase que pueda manejar cualquier tipo de Agregado:

```csharp
public class EventStream<T> where T : AggregateRoot, new()
{
    private readonly List<object> _baul; 
    private readonly Guid _aggregateId;

    public EventStream(List<object> baul, Guid aggregateId)
    {
        _baul = baul;
        _aggregateId = aggregateId;
    }

    // La magia de la reconstrucción automática
    public T Get()
    {
        var entidad = new T();
        
        // El stream sabe filtrar por ID (simplificado por ahora)
        var eventos = _baul; 

        entidad.Load(eventos);
        return entidad;
    }

    // El stream sabe dónde guardar
    public void Append(object nuevoEvento)
    {
        _baul.Add(nuevoEvento);
    }
}
```

## 🎯 El resultado: Orden Total

Mira cómo cambia tu código ahora que tienes un bibliotecario personal:

```csharp
// 1. Creamos el flujo específico para Jhon
var flujoJhon = new EventStream<Persona>(baul, idJhon);

// 2. Traemos a Jhon a la vida en UN solo paso
var jhon = flujoJhon.Get();

// 3. Jhon decide y el stream guarda
var hito = jhon.Decidir(unComando);
if (hito != null) flujoJhon.Append(hito);
```

### ¿Qué hemos ganado?
- **Escalabilidad**: Ahora puedes tener 10.000 personas y el código para cargar a cualquiera de ellas es siempre el mismo.
- **Seguridad**: El stream garantiza que no mezcles biografías.
- **Simplicidad**: Tu lógica de negocio (el handler o el program) solo se preocupa de pedir a la persona y dejarla actuar.

---

[⬅️ Volver a la sección anterior](./05-decidir-el-futuro.md)

[➡️ Siguiente sección: El riesgo de olvidar](./07-el-riesgo-de-olvidar.md)
