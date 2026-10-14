# 🔵 Nivel 1 · Filtros simples y proyecciones

`find(filtro, proyección)`: el primer JSON dice **qué documentos** quieres; el segundo, **qué campos** ver (`1` = mostrar, `0` = ocultar).

[⬅️ Nivel 0](nivel-0-primeros-pasos.md) · [Índice](../README.md) · [Nivel 2 ➡️](nivel-2-operadores.md)

---

### Ejercicio 7 · Buscar por título

**Enunciado:** busca la película cuyo título es exactamente "The Matrix".

```js
db.movies.find({ title: "The Matrix" })
```

**Explicación:** `find()` recibe un filtro con el campo y el valor buscado. Devuelve todos los documentos cuyo `title` sea exactamente ese.

**En SQL:** `SELECT * FROM movies WHERE title = 'The Matrix';`

---

### Ejercicio 8 · Mostrar solo algunos campos

**Enunciado:** de "The Matrix", muestra solo el título, el año y el argumento (`plot`), sin el `_id`.

```js
db.movies.find(
  { title: "The Matrix" },
  { title: 1, year: 1, plot: 1, _id: 0 }
)
```

**Explicación:** el segundo JSON es la **proyección**. El `_id` sale siempre salvo que lo ocultes expresamente con `0`.

**En SQL:** `SELECT title, year, plot FROM movies WHERE title = 'The Matrix';`

---

### Ejercicio 9 · Casablanca con proyección

**Enunciado:** busca la película cuyo título es exactamente "Casablanca". Muestra solo título, año y directores, sin el `_id`.

```js
db.movies.find(
  { title: "Casablanca" },
  { title: 1, year: 1, directors: 1, _id: 0 }
)
```

---

### Ejercicio 10 · Películas de un año, solo 5

**Enunciado:** muestra el título de 5 películas de 1994.

```js
db.movies.find({ year: 1994 }, { title: 1, _id: 0 }).limit(5)
```

**Explicación:** `.limit(5)` se queda con los cinco primeros resultados. Muy útil para no llenar la pantalla.

**En SQL:** `SELECT title FROM movies WHERE year = 1994 LIMIT 5;`

---

### Ejercicio 11 · Contar las películas de un año

**Enunciado:** ¿cuántas películas son del año 2000?

```js
db.movies.countDocuments({ year: 2000 })
```

**Explicación:** el mismo filtro que en `find()`, pero devuelve un número en lugar de la lista.

**En SQL:** `SELECT COUNT(*) FROM movies WHERE year = 2000;`

---

### Ejercicio 12 · Dos condiciones de igualdad

**Enunciado:** ¿cuántas películas de 1999 tienen la clasificación por edades (`rated`) "PG-13"?

```js
db.movies.countDocuments({ year: 1999, rated: "PG-13" })
```

**Explicación:** si pones varios campos en el mismo filtro, se tienen que cumplir **todos** (es un AND).

---

### Ejercicio 13 · Comentarios de un usuario

**Enunciado:** cuenta cuántos comentarios ha dejado el usuario con email `sean_bean@gameofthron.es`.

```js
db.comments.countDocuments({ email: "sean_bean@gameofthron.es" })
```

---

### Ejercicio 14 · Cines de un estado

**Enunciado:** cuenta los cines de California ("CA"). El estado está en `location` → `address` → `state`.

```js
db.theaters.countDocuments({ "location.address.state": "CA" })
```

**Explicación:** la notación con punto baja tantos niveles como haga falta.

**En SQL:** `SELECT COUNT(*) FROM theaters WHERE state = 'CA';`

> 💡 **Duda frecuente: ¿por qué aquí las comillas son obligatorias?**
> En la shell las claves siguen las reglas de JavaScript. Un nombre simple como `title` o `year` puede ir sin comillas, pero `location.address.state` lleva puntos y, sin comillas, JavaScript intentaría leerlo como variables y daría error. Por eso **toda ruta con punto** (`"imdb.rating"`, `"awards.wins"`…) va entre comillas. Las comillas de `"CA"` son otra cosa: es un valor de texto, y esas van siempre.

---

### Ejercicio 15 · Cines de Texas con proyección

**Enunciado:** lista 10 cines situados en Texas ("TX"). Muestra únicamente la ciudad y la calle.

```js
db.theaters.find(
  { "location.address.state": "TX" },
  { "location.address.city": 1, "location.address.street1": 1, _id: 0 }
).limit(10)
```

**Explicación:** la notación con punto también vale en la proyección.

> 💡 **Duda frecuente: ¿por qué `street1` y por qué hay que poner la ruta entera?**
> Un cine se guarda así:
>
> ```js
> {
>   theaterId: 1000,
>   location: {
>     address: {
>       street1: "340 W Market",
>       city: "Bloomington",
>       state: "MN"
>     }
>   }
> }
> ```
>
> Piénsalo como carpetas: el documento solo tiene dos campos arriba, `theaterId` y `location`. `location` es una carpeta con `address` dentro, y `address` es otra carpeta con `street1`, `city` y `state`. No es una lista (no hay corchetes): son **subdocumentos**, llaves dentro de llaves.
>
> - `street1` es el nombre real del campo; `street` no existe, y si pides un campo que no existe no sale nada.
> - Si pones solo `city`, MongoDB lo busca en el primer nivel, no lo encuentra y no muestra nada. Con `"location.address.city"` le dices por dónde bajar.
>
> Truco: comprueba siempre los nombres con `db.theaters.findOne()`.

---

[⬅️ Nivel 0](nivel-0-primeros-pasos.md) · [Índice](../README.md) · [Nivel 2 ➡️](nivel-2-operadores.md)
