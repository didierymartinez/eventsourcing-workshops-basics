# 11 - El Archivero Incombustible: Docker y PostgreSQL

Hasta la Fase 2, nuestra **Agencia de Biografías** dependía de un delicado escritorio de cristal (la Memoria RAM). Era rápido e inmediato, pero bastaba con un corte de luz (detener la aplicación) para que todos los recuerdos de nuestros clientes desaparecieran para siempre.

Con la Inyección de Dependencias (El Recepcionista) ya configurada, estamos listos para reemplazar ese escritorio de cristal por una **bóveda incombustible**.

---

## 🚀 El Problema: El Mundo Relacional vs El Evento Libre

Históricamente, los desarrolladores en C# han guardado datos usando tablas estrictas (`Id`, `Nombre`, `Edad`, `Ciudad`). A esto se le llama **Modelo Relacional** o SQL tradicional.

Pero en Event Sourcing, nosotros no guardamos a "Jhon el hombre". **Nosotros guardamos sus eventos**, que tienen esquemas (propiedades) completamente diferentes entre sí:
*   `PersonaNacida` tiene: Nombre, Fecha, Ciudad.
*   `MatrimonioRegistrado` tiene: NombrePareja.
*   `MudanzaRealizada` tiene: NuevaCiudad.

Si intentáramos guardar esto en SQL tradicional, tendríamos que crear decenas de tablas diferentes o una tabla gigante llena de columnas vacías (`NombrePareja = null` cuando nace). ¡Una pesadilla de mantenimiento!

## 🌊 La Solución: Bases de Datos Documentales (NoSQL)

Las bases de datos documentales (como MongoDB) nos permiten guardar objetos completos como texto en formato JSON. No nos obligan a tener un esquema rígido; simplemente guardan el "documento".

`{ "PersonaId": "123", "NombrePareja": "María" }`

Esto es **perfecto** para Event Sourcing. Podemos tener un único cuaderno infinito (Stream) y pegar allí cualquier JSON sin importar si es un nacimiento o una boda.

### ¿Por qué Sincosoft Cosmos usa PostgreSQL si PostgreSQL es Relacional?

¡Aquí viene un secreto increíble de la industria moderna!

**PostgreSQL** es famoso por ser un motor relacional sólido, pero en versiones recientes incorporó un superpoder llamado **`JSONB`** (JSON Binario). 

Esta característica le permite a PostgreSQL guardar y buscar dentro de documentos JSON enteros con una velocidad y eficiencia que rivaliza, e incluso supera, a bases de datos NoSQL dedicadas como MongoDB, pero sin perder la seguridad transaccional (ACID) que aman los bancos y las empresas.

PostgreSQL con `JSONB` se ha convertido en la herramienta suprema para Event Sourcing en el ecosistema .NET.

---

## 🛠️ Levantando nuestra Bóveda de Concreto (Docker)

Para no instalar bases de datos gigantes en nuestra computadora personal, usaremos **Docker**, que es como traer una bóveda mágica prefabricada y encenderla en segundos.

En la raíz de nuestro proyecto, crearemos un archivo llamado `docker-compose.yml`. Este archivo es una receta de cocina que le dice a Docker exactamente qué bóveda queremos.

1.  Crea un archivo llamado `docker-compose.yml` en la misma carpeta donde está tu `.sln` (Solución) o tu proyecto de consola.
2.  Pega el siguiente contenido:

```yaml
version: '3.8'

services:
  # El nombre de nuestro archivero
  postgres-eventstore:
    image: postgres:15-alpine # Usamos la versión 15 súper ligera
    container_name: cosmos_event_store
    environment:
      # Las llaves de la bóveda
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: Password123!
      POSTGRES_DB: eventstore_db
    ports:
      # Conectamos la puerta 5432 de la bóveda a nuestra computadora
      - "5432:5432"
    volumes:
      # Los papeles se guardan aquí para no perderlos si docker se apaga
      - postgres_data:/var/lib/postgresql/data

volumes:
  # Declaramos el disco duro virtual seguro
  postgres_data:
```

3.  Abre una terminal en esa carpeta y ejecuta el hechizo de invocación:

```bash
docker-compose up -d
```

*(El `-d` significa 'detach', para que la bóveda se ejecute silenciosamente en el fondo).*

---

## 🔗 La Cadena Inquebrantable (Connection String)

Nuestra aplicación en C# ahora necesita la dirección exacta y la llave para abrir esta bóveda. A esto se le llama la **Cadena de Conexión (`ConnectionString`)**:

`"Host=localhost;Port=5432;Database=eventstore_db;Username=admin;Password=Password123!"`

### El Descubrimiento

En este momento tenemos una bóveda inexpugnable (Postgres) encendida en nuestra terminal.

Pero si miras nuestra app en C#, seguimos teniendo un `InMemoryEventStore`. ¿Tendríamos que empezar a escribir sentencias SQL a mano (con librerías como Dapper o Entity Framework) para convertir nuestros records de C# a JSON, abrir la conexión de red a Postgres, mandarlos por el cable y guardar la respuesta?

Programar toda esa fontanería desde cero (traducir el `EventoAlmacenado` a sentencias `INSERT INTO eventos (id, version, data) VALUES (...)`) nos tomaría semanas y estaría lleno de errores de principiantes.

¿Recuerdas que la Recepcionista (en la sección 10) estaba esperando instrucciones?
Es hora de presentarle a nuestro **Bibliotecario Experto**, la librería que hará todo este trabajo sucio por nosotros con una sola línea de código.

Bienvenido al corazón de Sincosoft Cosmos. ¡Pasa a la siguiente sección para conocer a **Marten**!

---

[⬅️ Volver a la sección anterior](./10-inyeccion-de-dependencias.md)

[➡️ Siguiente sección: El Bibliotecario Experto (Introducción a Marten)](./12-introduccion-a-marten.md)
