# 01 - 🧠 El rastro de lo que sucede

Bienvenido. En este workshop no vamos a construir una base de datos tradicional. Vamos a construir una **Biografía**.

> [!NOTE]
> 🌱 **Antes de empezar — cómo leer este workshop (las "semillas").** A lo largo del camino verás cajas marcadas con 🌱 **Semilla**. Son adelantos cortos de conceptos avanzados (concurrencia, versionado de eventos, idempotencia, CQRS, generación de código…) plantados **mucho antes** de su sección dedicada. No tienes que dominarlos al verlos: solo *anclar la idea y el nombre correcto*. Cuando reaparezcan a fondo, ya tendrás dónde colgarlos. Es lo contrario a dejar todo lo difícil para el final. (La primera semilla aparece al terminar esta misma sección.)

## 🎯 El Objetivo
Imagina que quieres conocer la vida de una persona. Tienes dos opciones:
1. Ver su **foto actual** (donde ves si está feliz, qué ropa lleva y su edad).
2. Leer su **diario personal** (desde que nació hasta hoy).

Si el dueño de la empresa te pregunta: *"¿Cómo llegó esta persona a ser quien es hoy?"*, la foto no te sirve. Necesitas el diario.

En el software tradicional, solemos guardar "fotos" (el estado actual en una tabla). En este workshop, aprenderemos a guardar el **"diario"**.

---

## 1. El problema de la "Foto" (Estado Actual)
Cuando usamos una base de datos normal, si una persona se muda de ciudad, sobrescribimos su dirección. 
*   **Antes**: Calle A.
*   **Después**: Calle B.

**¿Qué perdimos?** Perdimos el hecho de que alguna vez vivió en la Calle A. Perdimos su historia.

## 2. La solución: El "Diario" (Hechos)
En lugar de guardar solo dónde vive alguien, vamos a guardar los hechos que lo llevaron ahí:
- *Persona nacida.*
- *Persona se mudó a Calle A.*
- *Persona se mudó a Calle B.*

Si tenemos la lista de hechos, siempre podemos saber dónde vive hoy, pero además ganamos el **pasado**.

### El Descubrimiento
A esta forma de diseñar sistemas donde la "Fuente de la Verdad" no es una foto del presente, sino la secuencia de todos los hechos del pasado, se le conoce como **Event Sourcing**.

> [!NOTE]
> 🌱 **Semilla — No todos los hechos necesitan salir del sistema.** Mira la diferencia con un ejemplo natural: que Jhon **consiga novia** es un hecho **interno** — personal, sin efecto legal; ningún otro sistema necesita hacer nada. Pero que Jhon **se case** *cambia su estado civil* — un hecho **legal** — y por eso el **Registro Civil** (otro sistema) sí debe reaccionar e inscribirlo oficialmente.
> El criterio **no** es si *socialmente* alguien se entera (a un cumpleaños va la familia, claro) — es si **otro sistema/contexto debe actuar** ante el hecho (típicamente cuando hay un efecto legal/oficial). En software esa distinción tiene nombre: **eventos de dominio** (internos) vs **eventos de integración** (otros sistemas reaccionan). En la **siguiente sección** dibujamos el mapa completo de "dentro vs fuera".

---

[➡️ Siguiente sección: El mapa de contextos (dentro y fuera)](./01b-mapa-de-contextos.md)
