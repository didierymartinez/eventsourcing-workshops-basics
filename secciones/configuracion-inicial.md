# Configuración inicial y requisitos previos

Antes de comenzar con el workshop, asegúrate de cumplir con los siguientes requisitos y de preparar tu entorno:

## 📝 Requisitos previos

- Tener instalado [.NET 9.0 SDK](https://dotnet.microsoft.com/download)
- IDE recomendado: Visual Studio, Visual Studio Code o Rider

---

## 🗄️ Creación de la base de datos

Marten necesita una base de datos PostgreSQL llamada `marten-demo` para funcionar. Tienes tres formas de prepararla:

### Opción A: Usando Docker (Recomendado)
Puedes crear el contenedor y la base de datos automáticamente con un solo comando:
```bash
docker run --name postgres-marten -e POSTGRES_PASSWORD=Marten123 -e POSTGRES_DB=marten-demo -p 5432:5432 -d postgres
```
> [!TIP]
> Al usar `-e POSTGRES_DB=marten-demo`, Docker creará la base de datos al iniciar el contenedor por primera vez.

### Opción B: Herramientas gráficas (pgAdmin / DBeaver)
Si ya tienes PostgreSQL instalado localmente o en un contenedor:
1. Conéctate a tu servidor PostgreSQL.
2. Haz clic derecho en **Databases** (Bases de datos).
3. Selecciona **Create** > **Database...**.
4. Escribe `marten-demo` como nombre y guarda los cambios.

### Opción C: Línea de comandos (CLI)

#### Si tienes PostgreSQL instalado localmente:
Puedes usar el comando `createdb` directamente. Estas herramientas suelen estar disponibles si:
- Instalaste **PostgreSQL** usando el instalador oficial (asegúrate de que la carpeta `bin` esté en tu PATH).
- En **Mac**, si las instalaste con Homebrew: `brew install postgresql`.
- Si usas **pgAdmin**, estas herramientas están incluidas en su carpeta de instalación (ej. `C:\Program Files\pgAdmin 4\v7\runtime`).

Ejecuta:
```bash
createdb -h localhost -p 5432 -U postgres marten-demo
```

#### Si usas Docker (y no tienes PostgreSQL local):
No necesitas instalar nada en tu máquina. Puedes ejecutar el comando directamente **dentro** del contenedor que ya está corriendo:
```bash
docker exec -it postgres-marten createdb -U postgres marten-demo
```
> [!NOTE]
> `postgres-marten` es el nombre que le dimos al contenedor en la Opción A.


---

## ✅ Verificación de la base de datos

Es importante confirmar que la base de datos `marten-demo` existe y es accesible antes de continuar. Tienes estas opciones según tu entorno:

### Opción 1: Desde la Terminal (CLI)

#### Si usas Docker:
Ejecuta el siguiente comando para listar las bases de datos dentro del contenedor:
```bash
docker exec -it postgres-marten psql -U postgres -l
```
> Busca `marten-demo` en la columna **Name**. Si aparece, la base de datos fue creada correctamente.

#### Si tienes PostgreSQL local:
```bash
psql -U postgres -l
```

### Opción 2: Desde herramientas gráficas (pgAdmin / DBeaver)
1. Abre tu herramienta y refresca la lista de bases de datos (**Databases**).
2. Verifica que `marten-demo` aparezca en el árbol de navegación.
3. Intenta expandir la base de datos; si puedes ver las carpetas internas (como *Schemas*), la conexión es exitosa.

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
