# Configuración inicial y requisitos previos

Antes de comenzar con el workshop, asegúrate de cumplir con los siguientes requisitos y de preparar tu entorno:

## 📝 Requisitos previos

- Tener instalado [.NET 9.0 SDK](https://dotnet.microsoft.com/download)
- Tener instalado [PostgreSQL](https://www.postgresql.org/download/) (versión 12 o superior)
  - Alternativa: Usar [Docker](https://hub.docker.com/_/postgres) con la imagen oficial de PostgreSQL
  - Ejemplo: `docker run --name postgres-marten -e POSTGRES_PASSWORD=Marten123 -p 5432:5432 -d postgres`
- IDE recomendado: Visual Studio, Visual Studio Code o Rider
- **Debes crear la base de datos `marten-demo` en tu instancia de PostgreSQL** (puedes hacerlo con una herramienta gráfica como pgAdmin, DBeaver, o desde la terminal con `createdb marten-demo`)

---

## ⚙️ Configuración inicial

1. Crea un nuevo proyecto de consola muy sencillo en .NET donde implementarás tu dominio y lógica de eventos. Esto permitirá avanzar paso a paso y entender cada parte del proceso.
   - Ejecuta en la terminal:
     ```bash
     dotnet new console -n NombreDelProyecto
     ```
   - Reemplaza `NombreDelProyecto` por el nombre que prefieras para tu proyecto.
2. Prepara tu entorno de desarrollo y asegúrate de tener acceso a la base de datos PostgreSQL creada.

---

[⬅️ Volver al índice](../README.md)

[➡️ Siguiente sección: Definición de eventos](./definicion-eventos.md)
