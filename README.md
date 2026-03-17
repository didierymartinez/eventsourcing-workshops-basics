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

## 📚 Secciones del workshop

1. [01 - 🧠 Fundamentos de Event Sourcing](./secciones/1-fundamentos.md)
2. [02 - ⚙️ Configuración inicial y requisitos previos](./secciones/2-configuracion.md)
3. [03 - 🏷️ Modelado de eventos](./secciones/3-modelado-eventos.md)
4. [04 - 🧩 Construir el Agregado (Puro C#)](./secciones/4-agregado-puro.md)
5. [05 - 💾 Persistencia de eventos con Marten](./secciones/5-persistencia-marten.md)
6. [06 - 📊 Consultas y Proyecciones](./secciones/6-consultas-proyecciones.md)

---

Sigue el orden de las secciones para avanzar paso a paso en el workshop.

