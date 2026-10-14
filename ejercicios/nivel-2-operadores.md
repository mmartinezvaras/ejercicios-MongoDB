# 🟡 Nivel 2 · Operadores de comparación y lógicos

En vez de un valor exacto, el campo recibe un objeto con un operador: `{ campo: { $gt: 5 } }`.

| Operador | Significa |
|---|---|
| `$gt` / `$gte` | mayor que / mayor o igual |
| `$lt` / `$lte` | menor que / menor o igual |
| `$in` | uno de varios valores del **mismo campo** |
| `$exists` | el campo existe (o no) |
| `$regex` | el texto cumple un patrón |
| `$or` | se cumple **alguna** de varias condiciones |

[⬅️ Nivel 1](nivel-1-filtros-y-proyecciones.md) · [Índice](../README.md) · [Nivel 3 ➡️](nivel-3-arrays-y-ordenacion.md)

---

### Ejercicio 16 · Películas con nota mayor que 9

**Enunciado:** busca las películas con nota de IMDb mayor que 9. Muestra título y nota.

```js
db.movies.find(
  { "imdb.rating": { $gt: 9 } },
  { title: 1, "imdb.rating": 1, _id: 0 }
)
```

**Explicación:** la nota está dentro del subdocumento `imdb`, por eso se escribe con punto y entre comillas. `$gt` = *greater than*.

**En SQL:** `SELECT title, rating FROM movies WHERE rating > 9;`

---

### Ejercicio 17 · Películas muy cortas

**Enunciado:** muestra 5 películas que duren menos de 10 minutos, con título y duración.

```js
db.movies.find(
  { runtime: { $lt: 10 } },
  { title: 1, runtime: 1, _id: 0 }
).limit(5)
```

**Explicación:** `$lt` = *less than*.

**En SQL:** `SELECT title, runtime FROM movies WHERE runtime < 10 LIMIT 5;`

---

### Ejercicio 18 · Un rango de valores

**Enunciado:** encuentra las películas breves pero bien valoradas: duración entre 60 y 75 minutos, ambos incluidos, y nota en IMDb superior a 8. Muestra título, duración y nota, sin `_id`.

```js
db.movies.find(
  { runtime: { $gte: 60, $lte: 75 }, "imdb.rating": { $gt: 8 } },
  { title: 1, runtime: 1, "imdb.rating": 1, _id: 0 }
)
```

**Explicación:** «ambos incluidos» = `$gte` y `$lte`. Dos operadores sobre el mismo campo van juntos en el mismo objeto.

---

### Ejercicio 19 · Varios valores posibles

**Enunciado:** cuenta las películas con clasificación "G" o "PG".

```js
db.movies.countDocuments({ rated: { $in: ["G", "PG"] } })
```

**Explicación:** `$in` busca documentos cuyo campo tenga **cualquiera** de los valores de la lista.

**En SQL:** `SELECT COUNT(*) FROM movies WHERE rated IN ('G', 'PG');`

> 💡 **Duda frecuente: ¿aquí no se podría usar `$or`?**
> Sí, da el mismo resultado:
>
> ```js
> db.movies.countDocuments({ $or: [ { rated: "G" }, { rated: "PG" } ] })
> ```
>
> Pero con `$or` repites el campo en cada condición. Cuando todas las opciones son del mismo campo, `$in` es más corto.
>
> **Entonces, ¿para qué sirve `$or`?** `$in` solo mira **un campo**. `$or` junta condiciones sobre **campos distintos**, o condiciones que no son valores fijos (`$gt`…), y eso `$in` no lo puede hacer.
>
> ```js
> // $in: un campo, varios valores → películas de España, Francia o Italia
> db.movies.find(
>   { countries: { $in: ["Spain", "France", "Italy"] } },
>   { title: 1, countries: 1, _id: 0 }
> ).limit(5)
>
> // $or: campos distintos → españolas O de más de 200 minutos
> db.movies.find(
>   { $or: [ { countries: "Spain" }, { runtime: { $gt: 200 } } ] },
>   { title: 1, countries: 1, runtime: 1, _id: 0 }
> ).limit(5)
> ```
>
> **Regla rápida:** mismo campo y valores exactos → `$in`. Campos distintos o condiciones como `$gt` → `$or`.

---

### Ejercicio 20 · Series con `$in`

**Enunciado:** lista 15 series (`type: "series"`) con clasificación "TV-14" o "TV-MA". Muestra título, año y clasificación.

```js
db.movies.find(
  { type: "series", rated: { $in: ["TV-14", "TV-MA"] } },
  { title: 1, year: 1, rated: 1, _id: 0 }
).limit(15)
```

---

### Ejercicio 21 · Dos condiciones a la vez

**Enunciado:** busca las películas de animación ("Animation") de 2010 en adelante. Muestra título y año.

```js
db.movies.find(
  { genres: "Animation", year: { $gte: 2010 } },
  { title: 1, year: 1, _id: 0 }
)
```

**Explicación:** varios campos en el mismo filtro = AND: se tienen que cumplir todos.

**En SQL:** `... WHERE genre = 'Animation' AND year >= 2010;`

---

### Ejercicio 22 · Campos que no existen

**Enunciado:** ¿cuántas películas de 2010 no tienen el campo `metacritic`?

```js
db.movies.countDocuments({ year: 2010, metacritic: { $exists: false } })
```

**Explicación:** `$exists: false` busca documentos donde el campo no aparece. En MongoDB cada documento puede tener campos distintos.

**En SQL:** `... WHERE metacritic IS NULL`

---

### Ejercicio 23 · Empieza por…

**Enunciado:** ¿cuántas películas tienen un título que empieza por "The Last"?

```js
db.movies.countDocuments({ title: { $regex: "^The Last" } })
// forma corta equivalente:
db.movies.countDocuments({ title: /^The Last/ })
```

**Explicación:** `^` significa «al principio del texto». Sin él contaría los títulos que contienen "The Last" en cualquier posición.

**En SQL:** `... WHERE title LIKE 'The Last%'`

> 💡 **Duda frecuente: ¿y si quiero los que *acaban* en algo? ¿`"The Last^"`?**
> No: para «acaba en» se usa `$` **al final**.
>
> | Patrón | Significa | SQL |
> |---|---|---|
> | `/^The Last/` | empieza por | `LIKE 'The Last%'` |
> | `/Story$/` | acaba en | `LIKE '%Story'` |
> | `/Love/` | contiene | `LIKE '%Love%'` |
>
> ```js
> db.movies.countDocuments({ title: /Story$/ })
> ```
>
> Ojo: distingue mayúsculas, así que `/The last/` no encuentra "The Last". Para ignorarlas añade `i`: `/Story$/i`.

---

### Ejercicio 24 · Una condición u otra

**Enunciado:** busca las películas que hayan ganado más de 100 premios (`awards.wins`) **o** que superen el millón de votos en IMDb (`imdb.votes`). Muestra título, premios y votos.

```js
db.movies.find(
  { $or: [
      { "awards.wins": { $gt: 100 } },
      { "imdb.votes": { $gt: 1000000 } }
  ] },
  { title: 1, "awards.wins": 1, "imdb.votes": 1, _id: 0 }
)
```

**Explicación:** `$or` recibe una lista de condiciones y basta con que se cumpla una. Aquí son dos campos distintos, así que `$in` no sirve.

**En SQL:** `... WHERE wins > 100 OR votes > 1000000`

> ⚠️ **Errores típicos en este ejercicio**
> - **Llaves descuadradas:** cada `{` necesita su `}`. Un `_id: 0}})` con una llave de más da error de sintaxis; debe ser `_id: 0 })`.
> - **`$gte` en vez de `$gt`:** «más de 100» no incluye el 100, así que es `$gt`.
> - **Proyección distinta a la pedida:** el enunciado pide título, premios y votos.

---

[⬅️ Nivel 1](nivel-1-filtros-y-proyecciones.md) · [Índice](../README.md) · [Nivel 3 ➡️](nivel-3-arrays-y-ordenacion.md)
