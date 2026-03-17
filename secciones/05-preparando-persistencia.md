# 05 - Un baúl que no se borra

Para que nuestros hechos "sobrevivan" al cierre de la aplicación, necesitamos un lugar donde guardarlos en el disco duro. 

## 1. La solución al problema
En lugar de una lista en RAM, usaremos una base de datos. Pero no una cualquiera; usaremos una que sepa manejar grandes volúmenes de información y que sea segura.

Para no tener que instalar programas pesados en tu computadora, usaremos un **Contenedor**. Es como una caja cerrada que contiene todo lo necesario para que nuestra base de datos funcione.

---

## 2. Preparando el baúl persistente

Ejecuta el siguiente comando en tu terminal para crear este almacén:

```bash
docker run --name almacen-hechos -e POSTGRES_PASSWORD=Marten123 -e POSTGRES_DB=taller-hechos -p 5432:5432 -d postgres
```

### ¿Qué acabas de hacer?
- Has creado una "caja" llamada `almacen-hechos`.
- Le has puesto una contraseña segura.
- Has creado un espacio llamado `taller-hechos` para guardar nuestra información.

---

## 3. Verificar que el baúl está abierto
Para confirmar que todo está listo para recibir nuestros hechos, ejecuta:

```bash
docker exec -it almacen-hechos psql -U postgres -l
```

> Si ves **taller-hechos** en la lista, el baúl está listo.

---

### El Descubrimiento
Este contenedor que acabas de lanzar está corriendo un motor de base de datos llamado **PostgreSQL**, y la herramienta que usamos para gestionarlo fácilmente se llama **Docker**. 

Ahora tenemos el lugar físico para guardar los hechos, pero... ¿cómo hacemos para que nuestra aplicación .NET hable con este baúl automáticamente?

---

[⬅️ Volver a la sección anterior](./04-el-problema.md)

[➡️ Siguiente sección: Automatizando el historial](./06-marten-event-store.md)
