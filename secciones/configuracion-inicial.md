# ⚙️ Configuración inicial y requisitos previos

Antes de comenzar con el workshop, asegúrate de cumplir con los siguientes requisitos y de preparar tu entorno.

## 📝 Requisitos previos

- **.NET 9.0 SDK**: [Descargar aquí](https://dotnet.microsoft.com/download)
- **Docker Desktop**: [Descargar aquí](https://www.docker.com/products/docker-desktop/) (Necesario para la base de datos)
- **IDE**: Visual Studio, VS Code o Rider

---

## 🗄️ Paso 1: Preparar la base de Datos (Docker)

Para este workshop utilizaremos PostgreSQL corriendo en Docker. Esto asegura que todos tengamos el mismo entorno sin instalaciones complejas.

### 1.1 Crear el contenedor
Ejecuta el siguiente comando en tu terminal para crear el contenedor y la base de datos automáticamente:

```bash
docker run --name postgres-marten -e POSTGRES_PASSWORD=Marten123 -e POSTGRES_DB=marten-demo -p 5432:5432 -d postgres
```

> [!TIP]
> El parámetro `-e POSTGRES_DB=marten-demo` crea la base de datos automáticamente al iniciar el contenedor.

### 1.2 Verificar disponibilidad
Es importante confirmar que la base de datos está lista. Ejecuta:

```bash
docker exec -it postgres-marten psql -U postgres -l
```

> Busca **marten-demo** en la lista. Si aparece, ¡estás listo para continuar!

---

## 🚀 Paso 2: Crear el proyecto .NET

Ahora que la base de datos está lista, crearemos el proyecto donde trabajaremos.

1. **Crear el proyecto de consola**:
   Ejecuta en la terminal:
   ```bash
   dotnet new console -n NombreDelProyecto
   ```
   *(Reemplaza `NombreDelProyecto` por el nombre que prefieras)*.

2. **Acceder a la carpeta**:
   ```bash
   cd NombreDelProyecto
   ```

---

[⬅️ Volver al índice](../README.md)

[➡️ Siguiente sección: Definición de eventos](./definicion-eventos.md)
