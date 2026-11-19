# Vacaciones de la banda

Una sola página (`index.html`, sin dependencias ni build) donde cada uno carga sus períodos de
vacaciones en un calendario, y otra pestaña donde se ven todos los calendarios juntos para
encontrar los días en que estamos **todos en Buenos Aires** (o solo los integrantes que tildes).

- **Cargar mis vacaciones**: escribís tu nombre, tocás el primer y el último día de cada período
  (podés cargar varios) y enviás. Si volvés a entrar con el mismo nombre, se cargan tus períodos
  para editarlos; al enviar se reemplazan.
- **Ver calendarios**: un checkbox por integrante. Los días en verde son los que no está de
  vacaciones ninguno de los tildados; en los demás, los puntos de colores dicen quién está afuera.
  Arriba hay una lista de los tramos en que estamos todos. Tocá un día para ver el detalle.

Los datos se guardan en **Firebase Realtime Database** (gratis, plan Spark, sin tarjeta).

## Puesta en marcha (unos 5 minutos)

1. Entrá a <https://console.firebase.google.com> con tu cuenta de Google → **Agregar proyecto**
   (cualquier nombre, podés desactivar Google Analytics).
2. En el menú: **Compilación → Realtime Database → Crear base de datos**. Elegí cualquier región
   y arrancá en **modo bloqueado**.
3. Pestaña **Reglas**: pegá esto y tocá **Publicar**.

   ```json
   {
     "rules": {
       "vacaciones": {
         ".read": true,
         "$id": {
           ".write": true,
           ".validate": "newData.hasChild('nombre') && newData.child('nombre').isString() && newData.child('nombre').val().length <= 40"
         }
       }
     }
   }
   ```

   Esto deja que cualquiera con el link lea y escriba **solo** dentro de `vacaciones/<nombre>`
   (no se puede borrar la base entera). No hay login: es para una banda de amigos.
4. Pestaña **Datos**: arriba aparece la URL de la base. Si elegiste la región de EE.UU. es
   `https://vertigo-lopez-default-rtdb.firebaseio.com`, que es la que ya está puesta en `DB_URL`
   (al principio del `<script>` de `index.html`). Si elegiste otra región, copiá la URL que
   aparece ahí y reemplazala.
5. Abrí `index.html`, cargá tus días y mirá en la pestaña Datos de Firebase que aparecen.

No hace falta el `apiKey` ni el resto de la config web de Firebase: la app usa la API REST de la
Realtime Database, que se maneja con las reglas del paso 3.

Si `DB_URL` dijera `TU-PROYECTO`, la app funcionaría en **modo prueba**: guarda solo en el
navegador (aparece un aviso amarillo), para poder probarla antes de configurar nada.

## Publicarla para que la use la banda

El archivo se puede abrir directo, pero para pasarles un link lo más fácil es subirlo a un hosting
estático gratis:

- **Netlify Drop**: entrá a <https://app.netlify.com/drop> y arrastrá la carpeta (o solo `index.html`).
- **GitHub Pages**: Settings → Pages → *Deploy from a branch* → `main` / root.

Con `#ver` al final del link (`…/index.html#ver`) abre directo en "Ver calendarios".

## Cómo se guardan los datos

```
vacaciones/
  juan/
    nombre: "Juan"
    periodos: [ { d: "2026-12-20", h: "2027-01-05" }, … ]   // d = desde, h = hasta (inclusive)
    t: 1766000000000                                         // cuándo se envió
```

La clave es el nombre sin tildes, en minúsculas ("Juan", "juan " y "JUAN" son la misma persona).
Los datos se ven y se pueden editar o borrar a mano en la pestaña Datos de Firebase.

## Configuración

En `index.html`, arriba del `<script>`:

| Constante | Qué hace |
|-----------|----------|
| `DB_URL`  | URL de tu Realtime Database (sin `/` al final). |
| `MESES`   | Cuántos meses mostrar a partir del actual (12 por defecto). |
