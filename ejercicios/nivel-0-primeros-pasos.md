# 🟢 Nivel 0 · Primeros pasos

Comandos para orientarte: elegir la base de datos, ver qué hay dentro y contar.

[⬅️ Volver al índice](../README.md) · [Nivel 1 ➡️](nivel-1-filtros-y-proyecciones.md)

---

### Ejercicio 1 · Elegir la base de datos

**Enunciado:** indica a la shell que vas a trabajar con la base de datos `sample_mflix`.

```js
use sample_mflix
```

**Explicación:** responde `switched to db sample_mflix`. Desde aquí, `db` significa sample_mflix. Hay que repetirlo cada vez que abres una shell nueva.

**En SQL:** `USE sample_mflix;`

---

### Ejercicio 2 · Ver las colecciones

**Enunciado:** lista las colecciones que tiene la base de datos.

```js
show collections
```

**Explicación:** es como preguntar qué tablas hay. Verás `comments`, `movies`, `theaters`, `users`…

**En SQL:** `SHOW TABLES;`

---

### Ejercicio 3 · Ver un documento de ejemplo

**Enunciado:** muestra una película cualquiera para ver cómo es un documento.

```js
db.movies.findOne()
```

**Explicación:** `findOne()` sin filtro devuelve el primer documento que encuentra. Fíjate en las listas entre corchetes `[ ]` (`genres`, `cast`) y en los subdocumentos entre llaves `{ }` (`imdb`, `awards`).

**En SQL:** `SELECT * FROM movies LIMIT 1;`

---

### Ejercicio 4 · Contar todas las películas

**Enunciado:** ¿cuántas películas hay en la colección `movies`?

```js
db.movies.countDocuments({})
```

**Explicación:** las llaves vacías `{}` son un filtro vacío, es decir, «todos». Salen más de 20 000.

**En SQL:** `SELECT COUNT(*) FROM movies;`

---

### Ejercicio 5 · Contar los cines

**Enunciado:** ¿cuántos cines hay registrados en la colección `theaters`?

```js
db.theaters.countDocuments({})
```

**Explicación:** igual que el anterior, cambiando de colección.

---

### Ejercicio 6 · Valores distintos de un campo

**Enunciado:** obtén la lista de clasificaciones por edades (`rated`) distintas que existen, sin repetir.

```js
db.movies.distinct("rated")
```

**Explicación:** `distinct` devuelve los valores diferentes de un campo. Sirve para saber qué valores puedes usar luego en un filtro.

**En SQL:** `SELECT DISTINCT rated FROM movies;`

---

[⬅️ Volver al índice](../README.md) · [Nivel 1 ➡️](nivel-1-filtros-y-proyecciones.md)
