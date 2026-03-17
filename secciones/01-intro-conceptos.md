# 01 - ¿Por qué Event Sourcing? (Conceptos)

Antes de tocar una sola línea de código, debemos entender el cambio de mentalidad. La mayoría de nosotros estamos acostumbrados al modelo **CRUD** (Create, Read, Update, Delete).

## 1. El modelo CRUD (Estado Actual)

Imagina que estás gestionando una **Orden de Compra**. En un sistema CRUD, tu base de datos guarda el "ahora":

| ID | Numero | Estado | Total |
|----|--------|--------|-------|
| 1  | FAC-001| Pagada | $125  |

Si el cliente cancela la orden, simplemente hacemos un `UPDATE` y cambiamos "Pagada" por "Cancelada".
**El problema:** Hemos perdido la información de que alguna vez estuvo pagada. No sabemos *cuándo* ocurrió ni *quién* lo hizo. Solo sabemos el presente.

---

## 2. El modelo Event Sourcing (La Historia)

En **Event Sourcing**, no guardamos el estado actual. Guardamos la **historia completa** de lo que ha sucedido. Cada cambio es un "Evento" (un hecho inmutable del pasado).

### Analogía: El Libro Mayor (Ledger)
Piensa en un contador. Un contador nunca borra una cifra de su libro. Si cometió un error o hubo un cambio, añade una nueva línea de ajuste.

Para nuestra Orden de Compra, la historia sería:
1. `Orden Creada` (ID: 1, Numero: FAC-001)
2. `Producto Agregado` (Nombre: Laptop, Precio: $1200)
3. `Producto Agregado` (Nombre: Mouse, Precio: $25)
4. `Pago Recibido` (Metodo: Tarjeta)

**¿Cuál es el total?** No está guardado en una columna, pero lo podemos calcular sumando los eventos: `$1225`.

---

## 3. Beneficios inmediatos
- **Auditoría perfecta:** Tienes el rastro de todo lo que pasó.
- **Viaje en el tiempo:** Puedes saber cómo se veía el sistema el martes pasado a las 3:00 PM simplemente procesando los eventos hasta esa fecha.
- **Nuevas preguntas:** Si mañana el jefe pregunta "¿Cuántos clientes agregaron un producto y luego lo quitaron?", con CRUD no podrías responder. Con Event Sourcing, sí.

---

[➡️ Siguiente sección: Creando tu primer proyecto](./02-primer-proyecto.md)
