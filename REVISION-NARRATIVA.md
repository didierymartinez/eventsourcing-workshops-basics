# 🔎 Revisión de Narrativa y Secuencia (auditoría del workshop completo)

> Revisión crítica de la estructura completa: ¿qué conviene introducir antes, qué reordenar, qué arreglar en la narrativa? Veredictos con prioridad. *Esta es mi recomendación como diseñador del workshop; tú decides qué ejecutar.*

---

> **Estado global (04/06):** ✅ resueltos #1, #3, #4, #5 y #6. Único pendiente (opcional, cosmético): **#2** — renumerar la Plantilla (§17) para que sea el capstone real al final (hoy mitigado con el banner de "cambio de hilo").

## 🔴 1. Dos workshops que se solapan (lo más importante) — ✅ RESUELTO (relación aclarada)
**Hallazgo.** Existen dos cuerpos: `fundamentals-workshop/` (01-12: records, interfaces, polimorfismo, comando, repositorio, DI, lenguaje ubicuo, aggregate, eventos de dominio, async, CQRS, outbox) y el **principal** (`secciones/`). Pero el principal **re-enseña** muchos de esos temas (records §03, DI §10, comando §08, aggregate §03-04, async §09, CQRS §20, outbox §14). Un alumno no sabe por dónde entrar ni por qué algo se explica dos veces.

**Veredicto.** No fusionar (cada uno tiene un propósito), pero **dejar explícita la relación**:
- `fundamentals-workshop` = **Nivel 0 — Nivelación (opcional)**: conceptos de C#/diseño *en aislamiento*, para quien no los tiene. Sin Event Sourcing.
- `secciones` (principal) = **el viaje** con Jhon, donde esos conceptos *se descubren en contexto*.
- Acción: un puente en ambos README ("¿ya dominas records, DI, async? salta al principal; ¿no? pasa por nivelación primero"). Y en el principal, cuando un tema ya está en fundamentos, **enlazar** en vez de competir.

## 🔴 2. La Plantilla (§17) está fuera de lugar
**Hallazgo.** §17 (Plantilla Cosmos.BuildingBlocks) es el **capstone**, pero el archivo está en medio (14 → 17 → 18). Un lector llega a §17 y ve `[Aggregate]`, `FetchForWriting`, `IPublicEvent` **antes** de que §18/§24 los expliquen. Narrativa invertida.

**Veredicto.** La Plantilla debe ir **al final** (es donde se junta todo). Recomiendo **renumerarla como §27** (última) y dejar §17 con redirección, o moverla. Mínimo viable: un banner al inicio de §17 — *"esto es el capstone/referencia; los conceptos que ves aquí se explican en §18-26; sigue el ROADMAP"*.

## 🔴 3. Referencias colgantes §15/§16 — ✅ RESUELTO
**Hallazgo.** El README enlazaba `15-limites-busqueda.md` y `16-proyecciones.md` como "Próximamente", pero no existían y su contenido ya lo cubre **§20**.
**Acción (04/06):** eliminados del README; Fase 5 ahora apunta a §20 y se aclaró que el orden real es el ROADMAP. ✅

---

## 🟡 4. Introducir **Testing** antes — ✅ RESUELTO
**Hallazgo.** Testing estaba en §21 (casi al final), pero es el premio de las funciones puras que existen desde §03/§07.
**Acción (04/06):** §03 ya nombra `evolve` como pura y testeable; **§07 ahora trae el primer test Given-When-Then** (casar a Jhon + proteger la invariante de idempotencia) con el mensaje "empieza a testear desde ya, no lo dejes para el final". §21 queda como profundización. ✅

## 🟡 5. El hilo conductor "salta" de ejemplo — ✅ RESUELTO
**Hallazgo.** §01-14 usan **Jhon/Persona** (Biografías). §17-25 cambiaban a **Orden/Obligación** (dominios Cosmos). §26 vuelve con **Registro Civil**. El hilo se rompía a mitad de camino sin avisar.

**Acción ejecutada (04/06):**
- Se fijó un **elenco canónico** en §01b (Biografías/`Persona` + Registro Civil/`ActaMatrimonio`).
- Se reescribieron los ejemplos de **§18, §20, §21, §24, §25** de `Orden`/`Obligación` → `Persona`/`RegistroCivil`. Cosmos real queda solo en cajas "🪐 Ancla Cosmos".
- **§17** (Plantilla) mantiene `OrdenDeCompra` —es el puente intencional a un producto real— pero ahora con un **banner de "cambio de hilo a propósito"** que lo enmarca (`OrdenDeCompra` ↔ `Persona`).
- Resultado: hilo único Biografías + Registro Civil de §01 a §26.

---

## 🟢 6. Nombrar la tríada Command / Event / Query antes — ✅ RESUELTO
**Hallazgo.** La distinción (comando = 1, rechazable; evento = N, hecho; query = lectura) aparecía dispersa.
**Acción (04/06):** semilla con tabla comparativa en **§07**, justo donde se introduce el primer "Comando" — incluye el error clásico ("evento con un solo dueño = comando disfrazado") y reenvía las queries a CQRS (§20). ✅

---

## 📋 Plan recomendado (orden de ejecución)
1. **Limpieza rápida (cero riesgo):** quitar enlaces colgantes §15/§16 del README (→ §20). Banner en §17.
2. **Relación de los dos workshops:** puente en ambos README (Nivel 0 opcional ↔ principal).
3. **Testing desde el inicio:** semillas Given-When-Then en §03 y §07.
4. **Hilo conductor único:** enmarcar los saltos a Cosmos o reescribir ejemplos a `Persona`/`RegistroCivil`.
5. **Tríada Command/Event/Query:** semilla en §07/§08.
6. **(Opcional mayor)** Renumerar la Plantilla a §27 (capstone real al final).

> Mi recomendación: ejecutar 1-3 ya (alto valor, bajo riesgo). 4 es la mejora de fondo de la narrativa. 5 es rápido. 6 solo si quieres pulcritud total de numeración.
