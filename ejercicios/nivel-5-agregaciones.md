# 🟣 Nivel 5 · Agregaciones

`aggregate([ etapa1, etapa2, ... ])` pasa los documentos por una cadena de etapas, como una tubería: lo que sale de una etapa entra en la siguiente.

| Etapa | Qué hace | En SQL |
|---|---|---|
| `$match` | filtrar | `WHERE` |
| `$group` | agrupar y calcular | `GROUP BY` |
| `$unwind` | desplegar una lista (un documento por elemento) | — |
| `$project` | elegir y renombrar campos | `SELECT` |
| `$sort` | ordenar | `ORDER BY` |
| `$limit` | quedarse con N | `LIMIT` |
| `$lookup` | unir con otra colección | `JOIN` |

Dentro de las etapas, los campos de los documentos se nombran con `$` delante: `"$rated"`, `"$imdb.rating"`.

[⬅️ Nivel 4](nivel-4-actualizaciones.md) · [Índice](../README.md)

---

### Ejercicio 36 · Contar por grupo

**Enunciado:** cuenta cuántas películas hay de cada clasificación (`rated`) y ordénalas de la más frecuente a la menos.

```js
db.movies.aggregate([
  { $group: { _id: "$rated", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
])
```

**Explicación:** el `_id` del `$group` es el campo por el que agrupas; `$sum: 1` suma 1 por cada documento. Las películas sin `rated` salen agrupadas como `null`.

**En SQL:** `SELECT rated, COUNT(*) AS total FROM movies GROUP BY rated ORDER BY total DESC;`

---

### Ejercicio 37 · Desplegar una lista

**Enunciado:** obtén los 5 géneros con más películas. Cada película tiene varios géneros y debe contar en todos.

```js
db.movies.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
])
```

**Explicación:** `$unwind` convierte una película con 3 géneros en 3 documentos, uno por género. Así el `$group` la cuenta en cada uno.

---

### Ejercicio 38 · Usuarios más activos

**Enunciado:** en la colección `comments`, obtén los 5 usuarios que más comentarios han escrito. Muestra nombre, email y total.

```js
db.comments.aggregate([
  { $group: { _id: "$email", nombre: { $first: "$name" }, total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 5 },
  { $project: { _id: 0, nombre: 1, email: "$_id", total: 1 } }
])
```

**Explicación:** se agrupa por email porque identifica al usuario (dos personas pueden llamarse igual). `$first` recoge el nombre del primer comentario de cada grupo. En el `$project`, `email: "$_id"` renombra el `_id`.

---

### Ejercicio 39 · Nota media por década

**Enunciado:** calcula la nota media de IMDb por década para las películas estrenadas desde 1950. Redondea la media a 2 decimales, muestra cuántas películas hay en cada década y ordena por década.

*Pista:* la década de un año es el año menos el resto de dividirlo entre 10 (1987 − 7 = 1980). *Cuidado:* en algunos documentos `imdb.rating` no es un número.

```js
db.movies.aggregate([
  { $match: { year: { $gte: 1950 }, "imdb.rating": { $type: "number" } } },
  { $group: {
      _id: { $subtract: ["$year", { $mod: ["$year", 10] }] },
      media: { $avg: "$imdb.rating" },
      peliculas: { $sum: 1 }
  } },
  { $project: { _id: 0, decada: "$_id", media: { $round: ["$media", 2] }, peliculas: 1 } },
  { $sort: { decada: 1 } }
])
```

**Explicación:**
- El `$match` va primero para trabajar con menos documentos y descartar las notas que no son números (ver la duda del ejercicio 32).
- `$mod` da el resto de la división y `$subtract` lo resta al año.
- `$avg` calcula la media y `$round` la redondea a 2 decimales.

---

### Ejercicio 40 · Unir dos colecciones

**Enunciado:** muestra los 10 comentarios más recientes junto con el título y el año de su película. Cada resultado debe tener autor, fecha, texto, título y año.

*Pista:* el comentario guarda el `_id` de la película en `movie_id`.

```js
db.comments.aggregate([
  { $sort: { date: -1 } },
  { $limit: 10 },
  { $lookup: {
      from: "movies",
      localField: "movie_id",
      foreignField: "_id",
      as: "pelicula"
  } },
  { $unwind: "$pelicula" },
  { $project: {
      _id: 0, autor: "$name", fecha: "$date", texto: "$text",
      titulo: "$pelicula.title", anio: "$pelicula.year"
  } }
])
```

**Explicación:**
- `$lookup` es el JOIN de MongoDB: añade a cada comentario una lista `pelicula` con las películas cuyo `_id` coincide con su `movie_id`.
- `$unwind` convierte esa lista de un elemento en un subdocumento normal.
- Ordenar y limitar **antes** del `$lookup` evita unir miles de comentarios que luego se descartan.

**En SQL:**

```sql
SELECT c.name, c.date, c.text, m.title, m.year
FROM comments c JOIN movies m ON c.movie_id = m._id
ORDER BY c.date DESC LIMIT 10;
```

---

[⬅️ Nivel 4](nivel-4-actualizaciones.md) · [Índice](../README.md)
