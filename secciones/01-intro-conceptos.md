# 01 - El rastro de lo que sucede

Antes de empezar a programar, pensemos en cómo guardamos la realidad en el software. La mayoría de los sistemas actuales se enfocan en el **"Ahoritismo"**.

## 1. El modelo del "Ahora" (Estado Actual)

Imagina que estás gestionando una **Orden de Compra**. Normalmente, tu base de datos guarda una foto del presente:

| ID | Numero | Estado | Total |
|----|--------|--------|-------|
| 1  | FAC-001| Pagada | $125  |

Si el cliente cancela la orden, simplemente sobrescribimos el valor: cambiamos "Pagada" por "Cancelada".
**El problema:** Al sobrescribir, hemos borrado la historia. No sabemos *cuándo* estuvo pagada, ni qué pasó entre que se creó y se canceló. Solo sabemos el presente.

---

## 2. El modelo de la "Historia" (Hechos pasados)

¿Y si en lugar de sobrescribir, simplemente anotamos cada cosa que pasa? Como si fuera el diario de vida de la orden.

Para nuestra Orden de Compra, las anotaciones serían:
1. Se creó la orden `FAC-001`.
2. Se agregó una `Laptop` por `$1200`.
3. Se agregó un `Mouse` por `$25`.
4. Se recibió el pago.

**¿Cuál es el total actual?** No lo tenemos guardado en ninguna celda, pero si leemos todas las notas de arriba hacia abajo, podemos calcularlo: `$1225`.

---

## 3. ¿Por qué es potente este enfoque?
- **No pierdes nada:** Tienes el rastro de todo lo que ocurrió.
- **Auditoría natural:** Si el total no cuadra, puedes revisar línea por línea qué se anotó mal.
- **Nuevas respuestas:** Si mañana el negocio pregunta "¿Cuántas veces la gente cambia de opinión antes de pagar?", con el modelo de historia puedes responder. Con el modelo del "ahora", esa información ya se borró.

---

### El Descubrimiento
Este enfoque de guardar la secuencia completa de hechos en lugar de solo el estado actual es lo que en el mundo del software conocemos formalmente como **Event Sourcing**.

---

[➡️ Siguiente sección: Preparando nuestro lienzo](./02-primer-proyecto.md)
