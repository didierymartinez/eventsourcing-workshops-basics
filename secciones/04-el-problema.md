# 04 - El gran problema: La Volatilidad

Si has ejecutado el código de la sección anterior, habrás visto que todo funciona de maravilla. Pero hay un problema "invisible" que todo desarrollador debe enfrentar.

## El Experimento
Haz lo siguiente:
1. Ejecuta tu programa (`dotnet run`). Verás el total de la orden.
2. Detén el programa.
3. Vuélvelo a ejecutar.

**¿Qué pasó con los eventos que "guardaste"?** 
Efectivamente, desaparecieron. La memoria RAM se limpia cada vez que la aplicación se cierra. 

---

## La necesidad de la Persistencia

En una aplicación real, no podemos permitir que los datos se borren al reiniciar el servidor. Necesitamos un **Event Store** que sea:
1. **Permanente:** Que guarde los eventos en un disco duro.
2. **Seguro:** Que maneje transacciones y fallos.
3. **Escalable:** Que permita buscar eventos de miles de órdenes distintas.

---

## ¿Cómo lo resolvemos?

Para solucionar esto, necesitamos dos cosas:
1. **Un motor de base de datos:** Usaremos **PostgreSQL**.
2. **Una forma de correrlo fácil:** Usaremos **Docker** para no tener que instalar nada pesado en tu computadora.

> [!NOTE]
> En la siguiente sección, prepararemos este "baúl" permanente para que nuestros eventos dejen de ser volatiles.

---

[⬅️ Volver a la sección anterior](./03-es-en-memoria.md)

[➡️ Siguiente sección: Preparando la persistencia](./05-preparando-persistencia.md)
