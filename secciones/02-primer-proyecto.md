# 02 - Tu primer proyecto .NET

Ahora que entendemos la teoría, vamos a preparar las manos. En esta etapa **solo necesitamos .NET**. No instalaremos bases de datos ni herramientas complejas todavía.

## 📝 Requisitos previos

- **.NET 9.0 SDK**: [Descargar aquí](https://dotnet.microsoft.com/download)
- **IDE**: Visual Studio, VS Code o Rider

---

## 🚀 Paso 1: Crear el proyecto

Abriremos una terminal en la carpeta donde quieras trabajar y ejecutaremos:

1. **Crear el proyecto de consola**:
   ```bash
   dotnet new console -n TallerMarten.OrdenCompra
   ```

2. **Acceder a la carpeta**:
   ```bash
   cd TallerMarten.OrdenCompra
   ```

3. **Abrir tu IDE**:
   Abre esta carpeta en tu editor favorito.

---

## Paso 2: Limpieza

Abre el archivo `Program.cs` y borra todo su contenido. Queremos un lienzo en blanco para empezar a construir nuestra lógica de eventos.

---

[⬅️ Volver a la sección anterior](./01-intro-conceptos.md)

[➡️ Siguiente sección: Event Sourcing en Memoria](./03-es-en-memoria.md)
