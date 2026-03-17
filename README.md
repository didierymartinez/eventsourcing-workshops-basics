# Workshop de Marten con .NET

Bienvenido al workshop práctico para aprender a usar [Marten](https://martendb.io/) con .NET y PostgreSQL.

## 🎯 Objetivo

Aprenderás a modelar eventos, almacenarlos y consultarlos usando Event Sourcing con Marten y PostgreSQL en .NET, siguiendo un enfoque paso a paso y práctico.

## 🧠 Conceptos clave

- **Event Sourcing**: Patrón donde el estado de la aplicación se construye a partir de una secuencia de eventos.
- **Stream**: Secuencia de eventos relacionados a un mismo agregado (por ejemplo, una orden de compra).
- **Record**: Tipo inmutable en C# ideal para modelar eventos.
- **Agregado**: Entidad principal del dominio que agrupa y protege la consistencia de un conjunto de objetos relacionados y sus reglas de negocio. En Event Sourcing, el agregado se reconstruye aplicando la secuencia de eventos de su stream (por ejemplo, una orden de compra con sus productos).
- **Marten**: Librería para .NET que facilita la persistencia de eventos y documentos en PostgreSQL.

## 📚 Secciones del workshop (Hoja de Ruta)

1. [01 - 🧠 ¿Por qué Event Sourcing? (Conceptos)](./secciones/01-intro-conceptos.md)
2. [02 - 🚀 Tu primer proyecto .NET](./secciones/02-primer-proyecto.md)
3. [03 - 📝 Event Sourcing en Memoria](./secciones/03-es-en-memoria.md)
4. [04 - ⚠️ El gran problema: La Volatilidad](./secciones/04-el-problema.md)
5. [05 - 🗄️ Preparando la Persistencia (Docker)](./secciones/05-preparando-persistencia.md)
6. [06 - 💾 Marten: Tu Event Store profesional](./secciones/06-marten-event-store.md)
7. [07 - 📊 Consultas y Proyecciones](./secciones/07-consultas-proyecciones.md)

---

Sigue el orden de las secciones para avanzar paso a paso en el workshop.

