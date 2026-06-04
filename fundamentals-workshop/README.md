# El Camino del Arquitecto: Pre-requisitos de Diseño

Bienvenido al "Campo de Entrenamiento" de Arquitectura de Software. 

El modelo de **Event Sourcing** y **CQRS** introduce paradigmas que rompen la mente de un programador tradicional (acostumbrado a los CRUD simples con bases de datos relacionales). Sin embargo, el esfuerzo cognitivo al aprender Event Sourcing no suele venir de los eventos en sí, sino de carecer de una base sólida en patrones de diseño subyacentes.

Este workshop introductorio está diseñado puramente para aislar, afilar y dominar las "herramientas del arquitecto" **antes** de construir el motor de eventos. 

> [!TIP]
> 📍 **Esto es el Nivel 0 — Nivelación (opcional).** Aquí los conceptos se ven **en aislamiento**, sin Event Sourcing. Es para quien aún no domina records, interfaces, polimorfismo, DI o DDD básico. Si ya los manejas, **salta al workshop principal** ([`../README.md`](../README.md)), donde estos mismos conceptos se **descubren en contexto** (el viaje con Jhon). No es obligatorio hacerlo primero; es una red de seguridad.

A lo largo de este recorrido, no guardaremos nada en archivos de texto exóticos ni simularemos transacciones temporales. Nos enfocaremos en código puro, limpio y empresarial.

## 🗺️ El Mapa

### Fase 1: Las Herramientas del Lenguaje (C# Moderno)
Entender las piezas finas que C# nos da para esquivar los problemas de Orientación a Objetos tradicional.
1. [Inmutabilidad y Records](./secciones/01-records-e-inmutabilidad.md)
2. [Interfaces vs Clases Abstractas](./secciones/02-interfaces-vs-abstract-classes.md)
3. [La Magia del Enrutamiento (Polimorfismo)](./secciones/03-polimorfismo-y-dynamic-dispatching.md)

### Fase 2: Patrones Estructurales 
El arte de desacoplar el negocio de la tecnología.
4. [El Patrón Comando (Command Pattern)](./secciones/04-el-patron-comando.md)
5. [El Patrón Repositorio](./secciones/05-el-patron-repositorio.md)
6. [Inversión de Control (Inyección de Dependencias)](./secciones/06-inyeccion-de-dependencias.md)

### Fase 3: Introducción a DDD
Pensar junto a negocio.
7. [Lenguaje Ubicuo](./secciones/07-lenguaje-ubicuo.md)
8. [El Agregado y su Raíz](./secciones/08-aggregate-y-aggregate-root.md)
9. [Eventos de Dominio](./secciones/09-eventos-de-dominio.md)

### Fase 4: Escalando a Sistemas Distribuidos
10. [Async / Await en el Mundo Real](./secciones/10-async-await-y-concurrencia.md)
11. [Fundamentos de CQRS](./secciones/11-fundamentos-de-cqrs.md)
12. [El Patrón Transaccional Outbox](./secciones/12-el-patron-outbox.md)

---
[Comenzar con la Sección 01 🚀](./secciones/01-records-e-inmutabilidad.md)
