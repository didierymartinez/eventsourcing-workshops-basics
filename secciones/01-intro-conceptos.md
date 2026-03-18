# 01 - 🧠 El rastro de lo que sucede

Bienvenido. En este workshop no vamos a construir una base de datos tradicional. Vamos a construir una **Biografía**.

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

---

[➡️ Siguiente sección: Preparando nuestro lienzo](./02-primer-proyecto.md)
