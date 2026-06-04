# 02 - Preparando nuestro lienzo

Para empezar a trabajar con hechos e historia, primero necesitamos un lugar muy sencillo donde experimentar. **Solo necesitamos .NET**. No instalaremos herramientas externas todavía.

## 🚀 Paso 1: Crear el proyecto

Abriremos una terminal en la carpeta de trabajo y ejecutaremos:

1. **Crear el proyecto**:
   ```bash
   dotnet new console -n Taller.HistoriaVida
   ```

2. **Acceder a la carpeta**:
   ```bash
   cd Taller.HistoriaVida
   ```

3. **Abrir tu IDE**:
   Abre esta carpeta en tu editor favorito (VS Code, Visual Studio, o Rider).

---

## Paso 2: Entendiendo nuestro lienzo

Observa qué ha creado .NET por ti:

- **`Taller.HistoriaVida.sln`**: El archivo de solución que agrupa y organiza tus proyectos.
- **`Taller.HistoriaVida.csproj`**: El archivo que define la configuración básica de tu proyecto.
- **`Program.cs`**: Este es nuestro **lienzo en blanco**. No nos perderemos en carpetas complejas ni arquitecturas sofisticadas. 

A medida que avancemos, veremos cómo este archivo evoluciona. Es la forma más sencilla de identificar cómo cada pieza que construyamos encaja con la anterior.

---

## Paso 3: Limpieza

Abre el archivo `Program.cs` y borra todo su contenido. Queremos un lienzo totalmente vacío para empezar a anotar nuestros primeros hechos.

---

> [!NOTE]
> 🌱 **Semilla — el norte arquitectónico: lógica pura al centro, infraestructura en los bordes.** Todo el workshop empuja hacia un estilo que el equipo de Wolverine llama **"A-Frame Architecture"**: la *decisión de negocio* vive en funciones puras (el agregado), y lo "sucio" (base de datos, red, bus) se queda en los bordes, lejos de esa lógica. La consecuencia práctica: **call stacks cortos** y código fácil de razonar y testear (sin saltar por 8 capas). Wolverine incluso desaconseja el exceso de capas tipo Onion/Clean. Guárdalo como brújula: cuando dudes dónde poner algo, pregúntate *"¿esto es decisión (centro) o infraestructura (borde)?"*.

---

[⬅️ Volver a la sección anterior](./01-el-diario-de-jhon.md)

[➡️ Siguiente sección: Vivir el pasado](./03-vivir-el-pasado.md)
