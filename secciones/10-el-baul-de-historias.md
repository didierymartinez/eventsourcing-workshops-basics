# 10 - Un baúl para nuestras historias

En la sección anterior vimos que "Jhon" desaparecía cada vez que cerrábamos el programa. Necesitamos un baúl resistente donde guardar su biografía para que sea eterna.

## 🎯 El Objetivo
Lograr que si guardamos un cumpleaños hoy, mañana (o después de reiniciar la computadora) ese cumpleaños siga ahí. 

Necesitamos una base de datos, pero no una que guarde "tablas", sino una que guarde **Streams** (nuestros diarios).

---

## 1. La herramienta: El Contenedor
Para no ensuciar tu computadora con instalaciones pesadas, usaremos una "caja" virtual llamada **Docker**. Dentro de esa caja, pondremos un motor de base de datos llamado **PostgreSQL**.

### Paso 1: Crear el baúl
Asegúrate de tener Docker instalado y ejecutando. Crea un archivo llamado `docker-compose.yml` en la raíz de tu proyecto `Taller.HistoriaVida` con este contenido:

```yaml
services:
  baul-historias:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: historias_vida
    ports:
      - "5432:5432"
```

### Paso 2: Encender el baúl
Abre tu terminal y ejecuta:

```bash
docker compose up -d
```

---

## 2. El Descubrimiento: El Event Store
Acabas de encender un servidor. Pero lo importante no es el software (`PostgreSQL`), sino el rol que va a cumplir en nuestra arquitectura.

Este almacén especializado en guardar los **Streams** (biografías) de tus entidades sin que el tiempo o los reinicios los borren se conoce conceptualmente como un **Event Store**.

> [!TIP]
> Un **Event Store** es una base de datos optimizada para añadir hechos al final de una lista (Append-only) y leerlos en orden cronológico. Tal como un diario real.

---

Ahora que tenemos el lugar físico, necesitamos el "pegamento" profesional para que nuestro código C# deje de usar listas en memoria y empiece a escribir en este baúl de forma automática.

[⬅️ Volver a la sección anterior](./09-el-riesgo-de-olvidar.md)

[➡️ Siguiente sección: Automatizando el diario](./11-automatizando-el-diario.md)
