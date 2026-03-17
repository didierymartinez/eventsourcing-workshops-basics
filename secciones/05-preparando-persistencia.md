# 05 - Preparando la Persistencia (Docker)

Para que nuestros eventos "sobrevivan" al reinicio de la aplicación, necesitamos una base de datos. Usaremos **PostgreSQL** porque es robusto, de código abierto y tiene extensiones geniales para trabajar con JSON.

## 1. ¿Por qué Docker?
En lugar de instalar PostgreSQL directamente en tu sistema (lo cual puede ser complicado y ensuciar tu configuración), usaremos **Docker**. 

Docker nos permite crear un "contenedor" con PostgreSQL listo para usar en segundos.

### Requisitos
- **Docker Desktop**: [Descargar aquí](https://www.docker.com/products/docker-desktop/)

---

## 2. Paso a paso: Creando tu base de datos

Ejecuta el siguiente comando en tu terminal para crear el contenedor y la base de datos automáticamente:

```bash
docker run --name postgres-marten -e POSTGRES_PASSWORD=Marten123 -e POSTGRES_DB=taller-marten -p 5432:5432 -d postgres
```

### ¿Qué hace este comando?
- `--name postgres-marten`: Le da un nombre a nuestro contenedor.
- `-e POSTGRES_PASSWORD=Marten123`: Configura la contraseña del administrador.
- `-e POSTGRES_DB=taller-marten`: Crea una base de datos llamada `taller-marten` al iniciar.
- `-p 5432:5432`: Abre el "puerto" para que nuestra app .NET pueda hablar con la base de datos.

---

## 3. Verificar disponibilidad
Es importante confirmar que la base de datos está lista. Ejecuta:

```bash
docker exec -it postgres-marten psql -U postgres -l
```

> Busca **taller-marten** en la lista. Si aparece, ya tienes tu baúl de eventos listo.

---

[⬅️ Volver a la sección anterior](./04-el-problema.md)

[➡️ Siguiente sección: Marten: Tu Event Store profesional](./06-marten-event-store.md)
