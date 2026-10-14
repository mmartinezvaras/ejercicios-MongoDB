# 🔴 Nivel 4 · Actualizaciones

`updateOne(filtro, cambios)` modifica el **primer** documento que cumple el filtro. Estos son los únicos ejercicios que **cambian los datos**.

[⬅️ Nivel 3](nivel-3-arrays-y-ordenacion.md) · [Índice](../README.md) · [Nivel 5 ➡️](nivel-5-agregaciones.md)

> 🛡️ **Antes de empezar: trabaja sobre una copia**
> Para no alterar la base de datos original, copia las colecciones que vas a modificar:
>
> ```js
> db.theaters.aggregate([ { $out: "theaters_copia" } ])
> db.movies.aggregate([ { $out: "movies_copia" } ])
> show collections   // comprueba que aparecen las copias
> ```
>
> `$out` guarda el resultado de la agregación en una colección nueva; sin filtro, copia todo. Después usa `db.theaters_copia` y `db.movies_copia` en lugar de los originales. Al terminar puedes borrarlas con `db.theaters_copia.drop()` y `db.movies_copia.drop()`.

---

### Ejercicio 34 · Añadir campos nuevos

**Enunciado:** al cine con `theaterId` 1000 añádele dos campos: `reformado` con valor `true` y `aforo` con valor `250`. Después comprueba el resultado con una consulta.

```js
db.theaters.updateOne(
  { theaterId: 1000 },
  { $set: { reformado: true, aforo: 250 } }
)
db.theaters.findOne({ theaterId: 1000 })
```

**Explicación:** la respuesta debe decir `matchedCount: 1, modifiedCount: 1`. `$set` crea el campo si no existe y lo sobrescribe si ya existe.

**En SQL:** `UPDATE theaters SET reformado = true, aforo = 250 WHERE theaterId = 1000;`

---

### Ejercicio 35 · Modificar un array y un contador

**Enunciado:** en la película "Casablanca":

1. Añade el género "Classic" a `genres` de forma que, si ejecutas la orden dos veces, no quede duplicado.
2. Incrementa en 1 el campo `num_mflix_comments`.
3. Deshaz ambos cambios (quita "Classic" y resta 1 al contador).

```js
// 1. Añadir sin duplicar
db.movies.updateOne({ title: "Casablanca" }, { $addToSet: { genres: "Classic" } })

// 2. Sumar 1
db.movies.updateOne({ title: "Casablanca" }, { $inc: { num_mflix_comments: 1 } })

// 3. Deshacer las dos cosas a la vez
db.movies.updateOne(
  { title: "Casablanca" },
  { $pull: { genres: "Classic" }, $inc: { num_mflix_comments: -1 } }
)

// Comprobar cada paso
db.movies.findOne({ title: "Casablanca" }, { genres: 1, num_mflix_comments: 1 })
```

**Explicación:**

| Operador | Qué hace |
|---|---|
| `$addToSet` | añade a la lista **solo si no está** (con `$push` se duplicaría) |
| `$inc` | suma; con un número negativo, resta |
| `$pull` | quita de la lista todos los elementos iguales al valor |

---

[⬅️ Nivel 3](nivel-3-arrays-y-ordenacion.md) · [Índice](../README.md) · [Nivel 5 ➡️](nivel-5-agregaciones.md)
