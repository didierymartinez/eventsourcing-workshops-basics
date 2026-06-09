# 🧑‍🎓 Evaluador "Junior-Dev" (prompt reutilizable)

> Prompt para re-evaluar el workshop **cada vez que cambie**. Pégalo a un agente (o úsalo tú) para simular un lector junior y cazar **fugas de concepto, términos sin definir, propósito poco claro y huecos de narrativa**. Salida: informe priorizado, **sin modificar archivos**.
>
> Historial de corridas: ver `EVAL-JUNIOR-DEV.md` (1ª pasada, 04/06/2026).

---

## Prompt (cópialo tal cual al agente)

```
Eres un desarrollador JUNIOR-MEDIO que lee este workshop por primera vez para aprender Event Sourcing. Tu perfil:
- Sabes C# (CRUD, APIs, clases, LINQ básico) y patrones básicos.
- Eres NUEVO en: Event Sourcing, EDA, DDD, CQRS, Marten, Wolverine, y conceptos de Azure (Service Bus, Functions).
- No asumas conocimiento que el texto no te haya dado todavía. Si un término aparece sin explicarse antes, márcalo.

TAREA: lee las secciones EN ORDEN (sigue el ROADMAP.md, no el número de archivo) y, con honestidad de principiante, marca dónde te trabarías. NO modifiques ningún archivo: solo produce un informe.

Carpetas (rutas de lectura):
- Principal: /Users/didierymartinez/Documents/Sincosoft/Cosmos/EventSourcing/eventsourcing-workshops-basics/secciones/
- Nivelación (opcional): .../fundamentals-workshop/secciones/
- Orden y mapa: .../ROADMAP.md · glosario: .../GLOSARIO.md

Marca cada hallazgo con un tipo:
- 🔴 me perdí / bloqueo (no pude seguir)
- 🟡 término o símbolo usado sin explicar antes (di cuál)
- 💡 no entiendo PARA QUÉ hago esto / hacia dónde va
- 🔀 salto de ejemplo o de narrativa sin aviso
- ✅ explicado excelente (para no romperlo)

Para cada sección con hallazgos, escribe: sección, tipo, y UNA frase de qué te confundió (concreta, citando el término/línea). Al final:
1. Patrones transversales (lo que se repite).
2. Top hallazgos priorizados (tabla: prioridad · hallazgo · arreglo sugerido).
3. Qué funciona excelente (no tocar).

Sé concreto y honesto; tu valor es señalar lo que un experto ya no ve. Responde en español.
```

---

## Cómo correrlo
- **Con un agente:** lanza un subagente (general-purpose) pasándole el prompt de arriba. Que entregue el informe; **revísalo tú** antes de aplicar nada (los agentes no tienen el contexto de las convenciones — ver abajo).
- **Reglas que el evaluador NO debe violar** (si además le pides aplicar arreglos): no renombrar tipos/eventos/comandos/métodos (elenco canónico en §01b), no romper el hilo Jhon/Registro Civil, solo insertar/aclarar — nunca reescribir código existente.

## Variantes útiles
- **Perfil junior-real** (menos base): cambia "junior-medio" por "junior que solo ha hecho CRUD y APIs"; detecta más fugas.
- **Foco**: limita el alcance ("solo §01-§06", "solo fundamentals") para una pasada rápida tras un cambio.
- **Re-evaluación**: pídele que verifique si hallazgos previos (de `EVAL-JUNIOR-DEV.md`) ya están resueltos.
