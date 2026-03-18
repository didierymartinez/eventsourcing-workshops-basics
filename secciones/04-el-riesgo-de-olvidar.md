# 04 - El gran problema: La pérdida de identidad

Nuestra biografía en C# funciona perfectamente, pero tiene un fallo fatal.

## 🎯 El Objetivo
Ejecuta tu programa varias veces. ¿Qué pasa con los cumpleaños de Jhon cada vez que reinicias la consola?

Exacto: **Vuelven a empezar de cero**.

---

## 1. El riesgo de la volatilidad
En el mundo real, una persona no olvida quién es cada vez que se va a dormir. Pero nuestro programa sí. 

Al guardar la `List<object> biografia` en la memoria RAM, estamos condenados a perder la historia en cuanto el proceso se detenga. 
- Si la luz se va: **Jhon deja de existir**.
- Si cerramos Visual Studio: **La biografía se borra**.

## 2. El desafío
En una aplicación real, no podemos permitir que nuestro "Agregado" pierda su historia. Necesitamos un lugar externo, un "Baúl" que no dependa de si nuestro programa está encendido o apagado.

### El Descubrimiento
Este problema nos obliga a buscar un **Almacén Persistente**. Pero no cualquier base de datos sirve; necesitamos una que entienda que lo que guardamos no son "fotos", sino una **secuencia de hechos**.

---

[⬅️ Volver a la sección anterior](./03b-decidir-el-futuro.md)

[➡️ Siguiente sección: El baúl de historias](./05-el-baul-de-historias.md)
