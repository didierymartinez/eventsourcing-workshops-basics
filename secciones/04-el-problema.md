# 04 - El gran problema: La Volatilidad

Si has ejecutado el código anterior, habrás visto que todo funciona perfectamente. Pero hay un problema crítico que ocurre en el mundo real.

## El Experimento
Haz lo siguiente:
1. Ejecuta tu programa (`dotnet run`). Verás el total de la orden.
2. Cierra o detén el programa.
3. Vuélvelo a ejecutar.

**¿Qué pasó con los hechos que "guardaste"?** 
Efectivamente, desaparecieron. Al ser una lista en memoria (RAM), se limpia cada vez que la aplicación se cierra. 

---

## La necesidad de la Persistencia

En un negocio real, un historial de ventas no puede borrarse al reiniciar el servidor. Necesitamos un **Event Store** que no sea volátil. Necesitamos que:
1. Guarde los hechos en un disco físico (Permanente).
2. Esté disponible para que otras partes del sistema lo consulten.
3. No dependa de si nuestra aplicación está encendida o apagada.

---

### El Descubrimiento
Acabamos de descubrir que una "Lista" en memoria no es suficiente para un sistema real. Necesitamos un **baúl persistente**. 

¿Cómo podemos guardar este historial para que nunca se borre, pero que sea fácil de consultar? 

---

[⬅️ Volver a la sección anterior](./03-es-en-memoria.md)

[➡️ Siguiente sección: Un baúl que no se borra](./05-preparando-persistencia.md)
