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

## Paso 2: Entendiendo nuestro lienzo

Antes de borrar nada, observa qué ha creado .NET por ti:

- **`TallerMarten.OrdenCompra.csproj`**: Es el corazón del proyecto. Aquí se definen la versión de .NET y las librerías que usaremos más adelante.
- **`Program.cs`**: Este es nuestro **lienzo en blanco**. No necesitamos múltiples archivos ni carpetas complejas todavía. 

### ¿Por qué trabajaremos aquí?
A medida que el workshop avance, verás cómo este archivo evoluciona. Es la forma más sencilla de **identificar las piezas** del rompecabezas de Event Sourcing:
1. Definiremos los **Eventos**.
2. Crearemos el **Agregado**.
3. Implementaremos la **Persistencia**.

Al tenerlo todo en un solo lugar al principio, podrás ver claramente cómo se conectan los conceptos antes de moverlos a una arquitectura más profesional.

---

## Paso 3: Limpieza

Abre el archivo `Program.cs` y borra todo su contenido. Deja el archivo vacío y listo para recibir tus primeros eventos.

---

[⬅️ Volver a la sección anterior](./01-intro-conceptos.md)

[➡️ Siguiente sección: Event Sourcing en Memoria](./03-es-en-memoria.md)
