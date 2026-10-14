# 🟠 Nivel 3 · Arrays, ordenación y top N

Una lista se consulta como un campo normal: `{ genres: "Comedy" }` busca "Comedy" **dentro** de la lista. `.sort({ campo: 1 })` ordena ascendente y `-1` descendente.

[⬅️ Nivel 2](nivel-2-operadores.md) · [Índice](../README.md) · [Nivel 4 ➡️](nivel-4-actualizaciones.md)

---

### Ejercicio 25 · Buscar dentro de una lista

**Enunciado:** muestra 5 películas que tengan el género "Comedy" (aunque tengan otros). Muestra título y géneros.

```js
db.movies.find({ genres: "Comedy" }, { title: 1, genres: 1, _id: 0 }).limit(5)
```

**Explicación:** `genres` es una lista, pero se consulta igual que un campo normal: MongoDB busca "Comedy" dentro de ella.

**En SQL:** haría falta una tabla aparte de géneros y un JOIN.

---

### Ejercicio 26 · Películas de un actor

**Enunciado:** muestra título y año de todas las películas en las que aparece Tom Hanks.

```js
db.movies.find({ cast: "Tom Hanks" }, { title: 1, year: 1, _id: 0 })
```

---

### Ejercicio 27 · Ordenar por año

**Enunciado:** muestra las películas de Steven Spielberg ordenadas de la más antigua a la más reciente.

```js
db.movies.find(
  { directors: "Steven Spielberg" },
  { title: 1, year: 1, _id: 0 }
).sort({ year: 1 })
```

**Explicación:** `.sort({ year: 1 })` ordena de menor a mayor año; con `-1` sería al revés.

**En SQL:** `... WHERE director = 'Steven Spielberg' ORDER BY year;`

> 💡 **Duda frecuente: ¿valdría con `year: { $lte: 2030 }`?**
> No. Eso es un **filtro**: decide *qué* películas salen, no *en qué orden*. Devolvería todas las de Spielberg, pero en el orden en que están guardadas.
>
> **Filtro = qué sale. `sort` = en qué orden sale.**

---

### Ejercicio 28 · Las más recientes de un país

**Enunciado:** muestra las 5 películas más recientes en las que ha participado España ("Spain").

```js
db.movies.find(
  { countries: "Spain" },
  { title: 1, year: 1, _id: 0 }
).sort({ year: -1 }).limit(5)
```

**Explicación:** `countries` es otra lista. Ordenar con `-1` y limitar a 5 da las cinco más recientes.

**En SQL:** `... WHERE country = 'Spain' ORDER BY year DESC LIMIT 5;`

> ⚠️ **Error típico:** escribir `country` en singular. El campo se llama `countries` (es una lista de países). Si el campo no existe, MongoDB **no da error**: simplemente no devuelve nada. Si una consulta sale vacía, revisa los nombres con `db.movies.findOne()`.

---

### Ejercicio 29 · Los últimos comentarios

**Enunciado:** muestra los 3 comentarios más recientes con nombre, texto y fecha.

```js
db.comments.find(
  {},
  { name: 1, text: 1, date: 1, _id: 0 }
).sort({ date: -1 }).limit(3)
```

**Explicación:** el filtro vacío `{}` significa «todos». Se ordenan por fecha descendente y se muestran tres.

**En SQL:** `SELECT name, text, date FROM comments ORDER BY date DESC LIMIT 3;`

---

### Ejercicio 30 · Top 5 mejor valoradas

**Enunciado:** entre las películas con más de 100 000 votos en IMDb, muestra las 5 con mejor nota.

```js
db.movies.find(
  { "imdb.votes": { $gt: 100000 } },
  { title: 1, "imdb.rating": 1, _id: 0 }
).sort({ "imdb.rating": -1 }).limit(5)
```

**Explicación:** se piden muchos votos para que la nota sea fiable. **Filtrar → ordenar → limitar** es la receta de cualquier «top N».

**En SQL:** `... WHERE votes > 100000 ORDER BY rating DESC LIMIT 5;`

---

### Ejercicio 31 · Dos actores juntos

**Enunciado:** ¿en qué películas aparecen juntos Tom Hanks y Meg Ryan? Muestra título y año.

```js
db.movies.find(
  { cast: { $all: ["Tom Hanks", "Meg Ryan"] } },
  { title: 1, year: 1, _id: 0 }
)
```

**Explicación:** `$all` exige que la lista contenga **todos** los valores, en cualquier orden. Con `$in` bastaría con uno de los dos.

---

### Ejercicio 32 · Top 5 de un director

**Enunciado:** muestra las 5 películas de Hayao Miyazaki con mejor nota en IMDb, de mayor a menor. Incluye título, año y nota.

```js
db.movies.find(
  { directors: "Hayao Miyazaki", "imdb.rating": { $type: "number" } },
  { title: 1, year: 1, "imdb.rating": 1, _id: 0 }
).sort({ "imdb.rating": -1 }).limit(5)
```

**Explicación:** igual que el ejercicio 30, más un seguro: `$type: "number"`.

> 💡 **Duda frecuente: ¿para qué sirve `$type: "number"`? ¿No vale sin él?**
> `$type` filtra por el **tipo de dato** del campo, no por su valor: «quédate solo con las películas cuya nota es un número».
>
> En sample_mflix casi todas tienen algo como `rating: 7.9`, pero algunas tienen `rating: ""` (texto vacío). Al ordenar, MongoDB considera cualquier texto «mayor» que cualquier número, así que con `sort(-1)` esas películas sin nota saldrían **las primeras** del top.
>
> Sin el `$type` la consulta también funciona y, si ninguna película de Miyazaki tiene la nota vacía, da el mismo resultado. Es un seguro.
>
> Otros tipos útiles: `"string"`, `"array"`, `"date"`, `"bool"`.

---

### Ejercicio 33 · Un único género

**Enunciado:** ¿cuántas películas tienen como **único** género "Documentary"? No basta con que lo incluya.

```js
db.movies.countDocuments({ genres: ["Documentary"] })

// alternativa: lista de tamaño 1 que contiene "Documentary"
db.movies.countDocuments({ genres: { $size: 1, $all: ["Documentary"] } })
```

**Explicación:** con la lista **entre corchetes**, MongoDB compara la lista entera (mismos elementos y mismo orden). Con `"Documentary"` sin corchetes buscaría dentro de la lista y contaría también las que tienen más géneros.

---

[⬅️ Nivel 2](nivel-2-operadores.md) · [Índice](../README.md) · [Nivel 4 ➡️](nivel-4-actualizaciones.md)
