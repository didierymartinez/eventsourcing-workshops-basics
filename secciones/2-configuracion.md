# 02 - Configuración inicial y requisitos previos

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
docker run --name postgres-marten -e POSTGRES_PASSWORD=Marten123 -e POSTGRES_DB=taller-marten -p 5432:5432 -d postgres
```

> [!TIP]
> El parámetro `-e POSTGRES_DB=taller-marten` crea la base de datos automáticamente al iniciar el contenedor.

### 1.2 Verificar disponibilidad
Es importante confirmar que la base de datos está lista. Ejecuta:

```bash
docker exec -it postgres-marten psql -U postgres -l
```

> Busca **taller-marten** en la lista. Si aparece, ¡estás listo para continuar!

---

## 🚀 Paso 2: Crear el proyecto .NET

Ahora que la base de datos está lista, crearemos el proyecto donde trabajaremos.

1. **Crear el proyecto de consola**:
   Ejecuta en la terminal:
   ```bash
   dotnet new console -n TallerMarten.OrdenCompra
   ```

2. **Acceder a la carpeta**:
   ```bash
   cd TallerMarten.OrdenCompra
   ```

---

[⬅️ Volver a la sección anterior](./1-fundamentos.md)

[➡️ Siguiente sección: Modelado de eventos](./3-modelado-eventos.md)
