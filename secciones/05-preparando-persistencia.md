# 05 - Un baúl que no se borra

Ya experimentamos el dolor de perder nuestros datos al reiniciar la aplicación. Es hora de buscar una solución permanente.

## 🎯 El Objetivo
Necesitamos un lugar que sea capaz de guardar nuestra lista de hechos (nuestro Event Store) de tal manera que, si el servidor explota o la aplicación se reinicia, podamos volver a leer el historial exactamente donde lo dejamos.

¿Cómo podemos externalizar ese almacenamiento fuera de la memoria de nuestro programa?

---

## 1. La solución al problema
Para que nuestros hechos "sobrevivan", usaremos un **Contenedor**. Es como una caja fuerte digital que vive fuera de tu aplicación y guarda la información en el disco duro.

---

## 2. Preparando el baúl persistente

Ejecuta el siguiente comando en tu terminal para crear este almacén:

```bash
docker run --name almacen-hechos -e POSTGRES_PASSWORD=Marten123 -e POSTGRES_DB=taller-hechos -p 5432:5432 -d postgres
```

### ¿Qué acabas de hacer?
- Has creado una "caja" persistente llamada `almacen-hechos`.
- Le has creado un espacio llamado `taller-hechos` para guardar nuestra información.

---

## 3. Verificar que el baúl está abierto
Para confirmar que todo está listo para recibir nuestros hechos, ejecuta:

```bash
docker exec -it almacen-hechos psql -U postgres -l
```

> Si ves **taller-hechos** en la lista, el baúl está listo.

---

### El Descubrimiento
Este almacén resistente que acabas de lanzar está corriendo un motor de base de datos llamado **PostgreSQL**, y la caja contenedora se gestiona con una herramienta llamada **Docker**. 

Ahora tenemos el lugar físico, pero necesitamos el "pegamento" para conectar nuestro código C# con este baúl de forma automática.

---

[⬅️ Volver a la sección anterior](./04-el-problema.md)

[➡️ Siguiente sección: Automatizando el historial](./06-marten-event-store.md)
