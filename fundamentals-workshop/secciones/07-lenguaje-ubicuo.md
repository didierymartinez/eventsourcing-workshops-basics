# 07 - Hablar con el Negocio: Lenguaje Ubicuo (Ubiquitous Language)
> 🎯 **Hacia dónde va:** El Lenguaje Ubicuo es lo que hará que tus comandos, métodos y eventos se llamen como el negocio realmente habla; en Event Sourcing es vital porque los nombres de tus eventos son el registro histórico que leerán los expertos del dominio.
> 📦 **Ejemplo de esta sección:** Hospital / Paciente (ingreso crítico por urgencias).

Domain-Driven Design (DDD) es una filosofía creada por Eric Evans. Su premisa número uno es simple pero revolucionaria: **El código no debe reflejar la estructura de una base de datos, debe reflejar el lenguaje exacto que hablan los expertos del negocio.**

La piedra angular de esta filosofía es el **Lenguaje Ubicuo (Ubiquitous Language)**.

## 🗣️ El Problema de Traducción

Imagina que estás construyendo software para un Hospital.
Te sientas con el Médico Jefe (el experto del dominio). Él te dice: *"Cuando ingresa un paciente crítico por urgencias, le asignamos una cama y tramitamos un triage rojo"*.

Tú vas a tu escritorio y tu código termina luciendo así:

```csharp
// ❌ Código Típico CRUD-céntrico (Anémico)
public void UpdatePatientStatus(Guid patientId, int bedNum)
{
    var patient = db.Patients.Find(patientId);
    patient.IsCritical = true;
    patient.BedId = bedNum;
    patient.TriageLevel = 1;
    patient.UpdatedAt = DateTime.UtcNow;
    db.SaveChanges();
}
```

¿Qué pasó aquí?
Reemplazaste *"Ingresa paciente crítico"* por `UpdatePatientStatus`.
Reemplazaste *"Asignamos cama"* por `BedId = bedNum`.
Reemplazaste *"Tramitamos triage rojo"* por `TriageLevel = 1`.

Has creado una "Deuda de Traducción". Si el médico lee tu código, no entenderá nada. Si hay un error ("Oye, el triage rojo falló ayer"), tú tendrás que adivinar que él se refiere a la propiedad `TriageLevel = 1` en la fila modificada a las 4pm.

## 🌉 El Puente: Lenguaje Ubicuo

El Lenguaje Ubicuo obliga a que el código adopte **exactamente las palabras del experto del negocio** en: variables, nombres de archivos, nombres de métodos, comandos y eventos. Sin tecnicismos, sin jerga de programador.

Mira cómo luce tu código cuando aplicas Lenguaje Ubicuo:

```csharp
// ✅ Código Guiado por el Dominio (Rico)
public class GestionarIngresosHandler
{
    public void Handle(IngresarPacienteCritico comando)
    {
        var paciente = _repository.Get(comando.PacienteId);

        // El experto médico entendería perfectamente leerte en voz alta esta línea:
        paciente.TramitarIngresoCriticoPorUrgencias(comando.AsignacionCama);

        _repository.Save(paciente);
    }
}
```

## DDD en Event Sourcing

En arquitecturas de Event Sourcing, los "Eventos" (Eventos de Dominio) **no son "Cambios de Tablas", son "Hechos Notificables para el Negocio"**. 

Un evento no debe llamarse `PatientUpdated`. 
Debe llamarse `PacienteCriticoIngresado`. 

### Reglas para nombrar bien:
1. **Comandos (Intención):** Verbo en imperativo exacto. Ej: `TramitarIngresoCritico`. No uses `UpdatePatient`.
2. **Métodos Internos (Acción):** Misma palabra que usa el gerente en su día a día. Ej: `paciente.AsignarCama()`. No uses `paciente.BedId = 12;`.
3. **Eventos (El Pasado Perfecto):** Verbo en pasado participio. Describe que algo sucedió irremediablemente. Ej: `PacienteCriticoIngresado`. Jamás uses `PatientRecordChanged`.

Si al leer en voz alta tus clases y métodos frente al Gerente de Operaciones de la empresa éste te entiende a la perfección, ¡Felicidades! Has logrado usar Lenguaje Ubicuo.

> [!NOTE]
> **El Lenguaje Ubicuo es *por Bounded Context*.** La misma palabra puede significar cosas distintas en módulos distintos: para Recursos Humanos una persona es un *Empleado* (con cargo y vacaciones); para Contabilidad es un *Tercero* (con cuenta por pagar). Mismo humano, **dos lenguajes**, porque son **dos Bounded Contexts** (la frontera donde un modelo y su lenguaje son coherentes). No existe "el lenguaje ubicuo de la empresa" — existe *uno por contexto*. En el workshop principal verás esto en acción cuando `Persona` (Biografías) se traduce a `Ciudadano` (Registro Civil) en la frontera.

---
[⬅️ Volver a la Fase anterior](./06-inyeccion-de-dependencias.md) | [➡️ Siguiente sección: El Agregado y su Raíz](./08-aggregate-y-aggregate-root.md)
