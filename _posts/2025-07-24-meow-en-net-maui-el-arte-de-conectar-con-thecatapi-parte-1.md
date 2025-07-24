---
title: "Meow en .NET MAUI: El arte de conectar con TheCatAPI (Parte 1)"
author: danielmonettelli
date: 2025-07-24 00:12:40 -0500
categories: [.NET MAUI, Experiencias]
tags: [netmaui, api, servicios, patrones, meow, thecatapi]
image:
  path: 10_1_cover_publication.png
  alt: "Meow en .NET MAUI: El arte de conectar con TheCatAPI (Parte 1)"
---

<p style='font-size: 20px;
  color: light-grey; margin: 0px 0px 20px; font-weight: bold; font-style: italic;'> Te voy a contar cómo nació Meow, una app que hice para saciar mi curiosidad con .NET MAUI. Al principio solo quería ver qué tal se sentía consumir una API real... pero la cosa se fue de las manos (en el buen sentido).

Lo que empezó como "vamos a mostrar unos gatitos random" terminó siendo toda una aventura. Guardar favoritos, explorar razas, que todo fluya suave... Al final quedé más enganchado de lo que pensé. Y bueno, acá estamos.</p>

En esta primera parte te comparto cómo armé la comunicación entre las pantallas y TheCatAPI. Básicamente, cómo hice que la app hable con el mundo exterior usando una clase de servicios. Te voy a mostrar las peticiones GET, POST y DELETE con ejemplos reales, sin tanto rollo técnico.

Si estás empezando con .NET MAUI o te da curiosidad cómo se organizan las llamadas a una API, creo que algo de esto te va a servir. Por lo menos es lo que a mí me funcionó después de varios intentos.

#  La base de todo: cómo logré que Meow se conecte con el mundo exterior

Acá viene lo interesante. Te voy a contar cómo armé el "motor" de Meow, esa parte que hace que todo funcione pero que el usuario nunca ve.

## ¿De qué va la cosa?

- **Cómo organicé la charla con TheCatAPI** usando un par de archivos clave: `ICatService` y `CatService`.
- Las peticiones **GET, POST y DELETE** tal como las uso en la app real (sin adornos).
- **Ejemplos que funcionan**, explicados sin tanto tecnicismo, por si los quieres adaptar.

Esta es la base sobre la que construí todo lo demás. No es rocket science, pero sí es la estructura que me permite tener el código ordenado y no volverme loco cuando tengo que cambiar algo.

En Meow casi todo pasa por dos archivos:
- `ICatService` → básicamente una lista de "esto es lo que necesito que funcione"
- `CatService` → el que realmente hace el trabajo sucio de hablar con la API

Gracias a esta división, las pantallas solo se preocupan por mostrar cositas lindas. Todo lo pesado pasa atrás, invisible, para que la experiencia sea fluida.

En los próximos posts te voy a contar sobre el diseño, las animaciones, cómo hice que funcione sin internet y demás. Pero por ahora, si querías ver cómo conectar una app MAUI con una API real, esto es lo básico que necesitas saber.

---

## 📚 Sobre esta serie

Esta es la primera parte de una serie donde te voy a contar, sin tanto verso, cómo fui armando **Meow** con .NET MAUI:

- **Parte 1:** Cómo conecté servicios y vistas (`ICatService` y `CatService`)
- **Parte 2:** El diseño en Figma y cómo usé Zeplin para no irme por las ramas
- **Parte 3:** Crear el ícono y pantalla de inicio sin que queden horribles
- **Parte 4:** Pasar de los diseños de Zeplin a código MAUI que realmente funcione
- **Parte 5:** Armar modelos y viewmodels que tengan sentido
- **Parte 6:** Guardar datos con SQLite + que funcione sin internet
- **Parte 7:** Los retoques finales y mejoras

> Esta serie te lleva desde la idea inicial hasta tener algo funcionando de verdad. Si te gustó, guarda este post para no perderte lo que viene.
{: .prompt-tip }


## 🔄 Cómo funciona esto por dentro: imágenes y explicación

Te voy a mostrar qué pasa cuando tocas cada botón en Meow. Con imágenes y explicaciones cortas para que se entienda bien qué hace cada cosa.

Todo lo que ves acá está en el [repo de Meow en GitHub](https://github.com/danielmonettelli/dotnetmaui-meow-app-oss). Si quieres ver el código completo, ahí lo tienes:

- En `Constants` añado todos los endpoints ordenaditos.
- `ICatService` define qué métodos necesito.
- `CatService` hace todo el trabajo de hablar con TheCatAPI.

> **Ojo acá:** Para seguir esta serie necesitas saber lo básico de .NET MAUI, C# y cómo funcionan las APIs REST.  
> Si recién arrancas, dale una mirada a la [documentación oficial de .NET MAUI](https://learn.microsoft.com/dotnet/maui/) primero.
{: .prompt-warning }


### 1. Obtener gatos aleatorios (GET)

![10-1-get-random-cat-images](10_1_get_random_cat_images.png)
_GET /images/search – Gatos aleatorios para votar_

**¿Qué está pasando acá?**  
Esta parte me encanta porque es muy simple, pero funciona muy bien.

Apenas entras a la app, ya tienes un gatito en pantalla completa. Si te gusta, puedes tocar el corazón ❤️ para guardarlo. Si no, simplemente seleccionas los botones de **"Me gusta"** o **"No me gusta"** y aparece otro gatito.

Es un poco como pasar fotos, pero de gatos. Algunos tienen cara seria, otros están medio despeinados o dormidos. **Nunca sabes qué te va a tocar**, y eso lo hace divertido.

---

**¿Y cómo se logra eso?**  
Todo funciona gracias a una llamada a **`/images/search`**, que se ejecuta cada vez que votas. Eso pide una nueva imagen a la API, y la app la muestra al instante. **Rápido y sin vueltas.**

El método que lo maneja es sencillo, pero cumple muy bien: cada vez que se llama, le dice al servidor algo así como *"¿Tienes otro gatito por ahí?"* y listo, aparece uno nuevo con una expresión distinta.

**No hay magia complicada**, pero sí un flujo claro que hace que quieras seguir viendo uno más... y otro... y otro 😅.



```csharp
public async Task<List<Cat>> GetRandomKitty()
{
    // "¡TheCatAPI, mandame un gato sorpresa!"
    HttpResponseMessage response = await _httpClient.GetAsync(APIConstants.CustomRandomKittyEndPoint);

    // Si todo salió bien, procesamos al nuevo amigo felino
    if (response.IsSuccessStatusCode)
    {
        // Convertimos la respuesta en un objeto Cat que podemos usar
        string content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<List<Cat>>(content);
    }

    // Si no hay internet o algo falla, devolvemos vacío
    return default;
}
```
{: file='Services/CatService.cs'}

**¿Qué tiene de especial este código?**

Esta parte me gusta porque hace algo muy simple, pero que se siente bien hecho:

- Hay una URL (`CustomRandomKittyEndPoint`) que cada vez que se llama, trae un gatito distinto. Siempre cambia.
- Aunque normalmente viene uno solo, a veces llegan varios porque está pensado para recibir una lista.
- Todo se hace con `async`, así que la app no se detiene mientras carga. Se puede seguir usando sin problema.
- Nada se congela ni se traba. Todo va suave.

---

**¿Cómo funciona al usarlo?**

1. Abres la app y ya hay un gatito esperándote.
2. Si te gusta, lo marcas. Si no, pasas al siguiente.
3. En el fondo se ejecuta `GetRandomKitty()` y pide otro gato.
4. La nueva imagen aparece sin cortes ni saltos raros.
5. ¿Quieres seguir? Solo sigues tocando. Y llegan más.

---

La verdad es que es fácil quedarse ahí un buen rato.  
Yo lo probé y pensé que iba a ver un par… pero sin darme cuenta, ya había pasado como media hora solo mirando gatos 😅.



### 2. Obtener razas de gatos (GET)

![10-1-get-cats-breeds](10_1_get_cats_breeds.png)
_GET /breeds – Lista de razas de gatos desde TheCatAPI_

**¿Qué hay en esta pantalla?**  
Cuando entras acá, es como encontrarte con un catálogo de razas felinas. Hay más de 60 opciones: Maine Coon, Siamés, Persa... y un montón que no tenía ni idea que existían.

Cada raza muestra varias imágenes, el nombre y algunos datos interesantes.
Y lo bonito es que, si alguna te llama la atención, solo necesitas tocarla. La app enseguida te muestra más gatitos parecidos, todos de esa misma raza. Sin complicaciones, todo muy natural.

Es genial si quieres conocer más sobre estos michis, o si simplemente quieres ver versiones adorables del mismo tipo de gato 😺.

**¿Cómo funciona esto en código?**  
Acá intervienen dos métodos que trabajan juntos: uno trae todas las razas desde la API, y el otro te deja filtrar gatos según la raza que elijas. Simple y efectivo.


```csharp
// Método 1: Obtener todas las razas disponibles
public async Task<List<Breed>> GetBreeds()
{
    // Pedimos a TheCatAPI todas las razas existentes
    HttpResponseMessage response = await _httpClient.GetAsync(APIConstants.BreedsEndPoint);

    if (response.IsSuccessStatusCode)
    {
        // Convertimos la respuesta JSON en objetos Breed
        string content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<List<Breed>>(content);
    }

    return default;
}

// Método 2: Obtener gatos filtrados por raza específica
public async Task<List<Cat>> GetRandomKittensByBreed(string breed)
{
    // Construimos la URL con el ID de la raza específica
    HttpResponseMessage response = await _httpClient.GetAsync($"{APIConstants.CustomRandomKittiesByBreedEndPoints}&breed_ids={breed}");

    if (response.IsSuccessStatusCode)
    {
        // Recibimos hasta 10 gatos de la raza seleccionada
        string content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<List<Cat>>(content);
    }

    return default;
}
```
{: file='Services/CatService.cs'}

**¿Qué tiene de especial esta parte?**

- **`GetBreeds()`**: Pide todas las razas disponibles, carga cuando la pantalla de razas se abre.
- **`GetRandomKittensByBreed(breed)`**: Muestra gatitos según la raza que elijas.
Y apenas entras a la sección, ya aparece la primera opción sin que tengas que mover un dedo. Así puedes empezar a ver desde el primer momento, sin pasos extra.
- **Parámetro dinámico**: `&breed_ids={breed}` cambia según la opción del usuario.
- **Todo fluye sin fricciones**: Apenas entras, ya ves algo. Y si quieres cambiar de raza, solo das un toque. Así de fácil.
- **Varios resultados a la vez**: No llega un solo gatito, sino hasta 10 de la raza elegida.
- **Con información clara**: Cada imagen viene con detalles útiles sobre esa raza.

> **Nota técnica:** Esta lógica —mostrar primero una raza y sus gatos sin que el usuario haga nada— está dentro del `BreedsViewModel`, usando MVVM.  
> En las próximas publicaciones voy a explicar cómo funciona ese ViewModel y cómo se conecta todo para que funcione de forma natural.  
{: .prompt-info }

---

**¿Cómo se ve en acción?**

1. Entras a la sección de “Razas” → Se llama a `GetBreeds()` y aparecen todas.
2. Tocas la que te guste → Por ejemplo, “Maine Coon”.
3. La app toma su ID único → Por ejemplo: `"mcoo"`.
4. Se ejecuta `GetRandomKittensByBreed("mcoo")` → Y llegan solo de esa raza.
5. Te muestra una galería hecha a medida → Solo Maine Coons en este caso.

---

Este flujo deja que cada persona explore a su ritmo: puede ver muchas razas o centrarse en una.  
Es como una especie de catálogo visual de gatitos, pero mucho más cercano y fácil de usar 😺.



### 3. Agregar un gato a favoritos (POST)

![10-1-post-add-cat-to-favorites](10_1_post_add_cat_to_favorites.png)
_POST /favourites - Agregar gato a favoritos_

**¿Qué ves en la imagen?**  
¡Un momento especial! Encontraste *al* gato. Ese que te ganó con su mirada tierna o una pose divertida. Tocas el ícono del corazón y… ¡boom! Se llena de color, aparece una pequeña animación, y ese gatito queda guardado en tu colección personal.

Es ese instante de: *“¡Este me lo quedo!”* — algo simple, pero que logra conectar con quien está usando la app.

**¿Cómo se logra esto en código?**  
Aquí es donde la lógica se pone interesante. No solo se envía un dato a la API, sino que se establece una relación entre el usuario y el gato. A partir de ese momento, ese gatito queda marcado como uno de tus favoritos dentro de la app.


```csharp
public async Task<string> AddFavoriteKitten(string cat_id)
{
    // Primero, creamos un "paquete" con la información del gato favorito
    var favoriteCatRequest = new FavoriteCatRequest()
    {
        Image_id = cat_id  // El ID único de esa foto específica
    };

    // Convertimos nuestro objeto C# a formato JSON (el idioma de las APIs)
    var body = JsonSerializer.Serialize(favoriteCatRequest);
    var content = new StringContent(body, Encoding.UTF8, "application/json");

    // Enviamos el "paquete" a TheCatAPI con método POST
    var response = await _httpClient.PostAsync(APIConstants.FavoriteKittensEndPoint, content);

    // Si todo salió bien, el gato ya está guardado en la nube
    if (response.IsSuccessStatusCode)
    {
        return await response.Content.ReadAsStringAsync();
    }

    return default;
}
```
{: file='Services/CatService.cs'}

**¿Qué me gusta de este código?**

No es algo complicado, pero mientras lo iba armando, fui entendiendo cosas que me parecieron útiles y quería compartirlas.

Una de ellas es la diferencia entre `GET` y `POST`. Con `GET` solo pides información, como quien dice: “muéstrame algo”. Pero con `POST`, ya estás enviando datos al servidor. En este caso, lo que hago es decirle a la API: *“este usuario quiere guardar este gato como favorito”*.

Para eso uso un objeto llamado `FavoriteCatRequest`, que me sirve para ordenar bien la información antes de enviarla. Y con `StringContent`, lo convierto todo a JSON, que es el formato que la API entiende.

Cuando tocas el corazón en un gatito, la app le manda su ID a TheCatAPI y el ícono cambia de color. Es una forma rápida de marcar que te gustó.

> Pero hay un detalle importante: como no se está usando un `sub_id`, esos favoritos no quedan realmente guardados a tu nombre. Todo se guarda de forma temporal en tu celular.  
{: .prompt-warning }

Eso quiere decir que, si desinstalas la app o cambias de teléfono, los vas a perder sin previo aviso.

En las siguientes publicaciones voy a explicar cómo mejorar esto para que tus favoritos no desaparezcan.  
Mientras tanto, puedes echarle un vistazo a la documentación en [thecatapi.com](https://thecatapi.com).

**¿Cómo funciona cuando lo usas?**

Tocas el corazón, la app toma el ID de ese gato específico, prepara la información, se la manda a TheCatAPI, y listo. El corazón cambia de color y ese gatito ya forma parte de tu colección personal.

Si no tienes internet en ese momento, no te preocupes. Meow se acuerda de lo que hiciste y sincroniza todo cuando vuelves a tener conexión.
3. **Se arma el paquete de datos** → Es como preparar un sobre con toda la info necesaria.
4. **Se envía a TheCatAPI** → El mensaje es claro: *“Este usuario ha marcado este gato como favorito”*.
5. **Respuesta al instante** → El corazón se anima y ese gatito ya es parte de tu colección.


Incluso si estás sin internet, Meow recuerda tu elección y la sincroniza cuando vuelvas a tener conexión. ¡Nunca pierdes un favorito!

### 4. Listar favoritos (GET) y eliminar (DELETE)

![10-1-get-list-favorite-cats](10_1_get_list_favorite_cats.png)
_GET /favourites- Lista de favoritos_

**¿Qué ves en la imagen?**  
Este espacio es solo tuyo.
Aquí se muestran los gatitos que alguna vez marcaste como favoritos, acomodados en una cuadrícula limpia, sin complicaciones. No hace falta mucho para entenderlo: es como una pequeña colección de momentos que te hicieron sonreír. Quizá te acuerdes de ese Maine Coon con cara de sabio, del Siamés que te llamó la atención, o del gatito naranja que simplemente te dio ternura.

Cada uno está ahí por algo. Y eso lo hace especial.

Y si un día decides quitar alguno, también puedes hacerlo sin complicaciones. Solo tienes que tocar la imagen y se eliminará automáticamente. Así de fácil.

**¿Cómo se logra esto en código?**  
Todo esto se logra con dos métodos que trabajan juntos:
uno muestra tus favoritos, el otro los elimina si tú lo decides.

No hay nada complejo. Es solo una forma de darte control sobre lo que quieres guardar… y sobre lo que ya no necesitas.


```csharp
// El "bibliotecario": trae todos los favoritos organizaditos
public async Task<List<FavoriteCatResponse>> GetFavoriteKittens()
{
    // Pedimos a TheCatAPI: "Muéstrame todos los favoritos de este usuario"
    HttpResponseMessage response = await _httpClient.GetAsync(APIConstants.FavoriteKittensEndPoint);

    if (response.IsSuccessStatusCode)
    {
        // Convertimos la lista JSON en objetos C# que podemos mostrar
        string content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<List<FavoriteCatResponse>>(content);
    }

    return default;
}

// El "encargado de limpieza": elimina favoritos específicos
public async Task<string> DeleteFavoriteKitten(int favourite_id)
{
    // Le decimos a la API: "Elimina este favorito específico"
    var response = await _httpClient.DeleteAsync($"{APIConstants.FavoriteKittensEndPoint}/{favourite_id}");

    if (response.IsSuccessStatusCode)
    {
        return await response.Content.ReadAsStringAsync();
    }

    return default;
}
```
{: file='Services/CatService.cs'}

**¿Qué hace especial a estos métodos?**

- **`GET`** me sirve para mostrar todos los favoritos sin modificar nada.
- **`DELETE`** borra un gatito de la lista usando su ID.
- Creé un objeto llamado **`FavoriteCatResponse`** que solo organiza mejor los datos que necesito mostrar.
- La URL se adapta según el gatito que quieras quitar: `/favourites/{id}`.
- Y cada cambio se **sincroniza solo**, sin que el usuario tenga que hacer nada. Eso me pareció importante..

**Así se vive desde la app:**

1. Cuando abres la sección de favoritos, todo carga rápido y sin vueltas.
2. Los gatitos aparecen en cuadrícula, bien ordenados, y es fácil volver a verlos.
3. Cada uno tiene algo que, en su momento, te hizo elegirlo.
4. Y si quieres sacar alguno, con una pulsación larga aparece la opción de eliminar.
5. Todo se actualiza enseguida, sin importar si usas otro dispositivo después.

Pensé esta parte como un pequeño rincón dentro de Meow.
Un espacio tranquilo, donde puedas guardar esos gatitos que te sacaron una sonrisa.
Y también para que, si en algún momento ya no quieres tenerlos ahí, puedas quitarlos sin complicarte.

Está hecho con mucho cuidado, pensando en los detalles y en que la experiencia sea suave.
No es una parte compleja, pero sí es una de las que más cariño tiene en toda la app 🐾

---

## ¿Por qué este patrón me ayuda tanto en .NET MAUI?

Cuando empecé con Meow, tenía claro algo: si la app no responde bien, la gente la cierra. Y es totalmente comprensible.
Por eso traté de construir una base simple, que funcionara bien desde el primer toque.

### 🚀 **Una estructura que me dio tranquilidad**

Todo gira en torno a una clase que llamé CatService.
Ahí puse la lógica para hablar con la API y obtener los datos. Nada complejo, pero sí bien separado.
Eso me permitió tener las pantallas limpias, sin tanto lío.

Los métodos tienen nombres fáciles de entender. Cosas como GetRandomKitty() o AddFavoriteKitten().
No quise reinventar nada, solo hacer que el código se lea sin tener que pensarlo dos veces.

Y como uso esa clase en toda la app, si en algún momento la API cambia, solo toco ahí… y ya.

### 🛠️ **Una experiencia que buscaba ser simple**

Lo que más me importaba era que la app se sintiera fluida.
Que cuando alguien toque algo, pase algo. Sin demoras raras.

Además, traté de que no dependa tanto del internet. Si estás sin conexión, igual guarda lo que haces, y después se sincroniza.
Eso lo hice porque a mí mismo me frustra cuando una app no funciona por un rato sin señal.

---

**Y al final…**  
No hice nada mágico. Solo traté de pensar en quien iba a usar Meow.
Si esta forma de organizar el código te sirve, me alegra un montón.
Y si no, igual gracias por leer hasta acá. 🙌

---

## 💭 Una reflexión honesta

Cuando arranqué con Meow, no tenía un plan maestro. Solo quería experimentar con .NET MAUI y hacer algo divertido. Lo que no me imaginaba era cuánto iba a aprender en el camino.

Cada línea de código, cada error que tuve que solucionar, cada pequeño logro... están ahí, en esta app. No me considero un experto, soy simplemente un desarrollador que disfruta compartir lo que aprende, con la esperanza de que le sirva a alguien más.

### ¿Por qué compartir esto?

Porque me parece justo. Aprendí mucho de otras personas que también se tomaron el tiempo de escribir lo que sabían.
Así que si esto le ahorra tiempo a alguien, o le da una idea, o simplemente le saca una sonrisa... con eso me siento más que bien.

### Los detalles, eso es lo que más cuido

Meow no es una app compleja. La verdad, está hecha con cosas simples.
Lo que más traté fue de cuidar los detalles. Esas cosas pequeñas que a veces pasan desapercibidas, pero que hacen que usar algo se sienta mejor.

Desde una animación chiquita hasta que funcione aunque no tengas internet.

**¿Tu próximo proyecto?**  
No tiene que ser sobre gatos (aunque si lo es, ya quiero verlo 😸).
Puede ser lo que quieras. Lo único que diría es: hacerlo claro, hacerlo tuyo, y prestale atención a esas pequeñas cosas que a veces dicen más que el resto.

---

## Repositorio

El proyecto es open source y lo puedes ver haciendo clic en la imagen de abajo. ¡Dale una mirada!

[![10-x-github-repository](https://raw.githubusercontent.com/danielmonettelli/danielmonettelli.github.io/main/assets/images/10_x_github_repository.png)](https://github.com/danielmonettelli/dotnetmaui-meow-app-oss){:target="_blank"}

---

## Recursos

- <a href="https://thecatapi.com/" target="_blank">TheCatAPI - API oficial para obtener imágenes y datos de gatos</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/maui/?view=net-maui-9.0" target="_blank">Documentación oficial de .NET MAUI</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/api/system.net.http.httpclient?view=net-9.0" target="_blank">HttpClient - Documentación oficial de Microsoft</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/api/system.text.json?view=net-9.0" target="_blank">System.Text.Json - Serialización JSON en .NET</a>.

---

## Publicación en inglés

- <a href="https://danielmonettelli.github.io/posts/meow-dotnet-maui-the-art-of-connecting-with-thecatapi-part-1/" target="_blank">Meow in .NET MAUI: The art of connecting with TheCatAPI (Part 1)</a>.

---

## ¿Y qué más?

Esto fue solo el comienzo. Me quedan varias cosas por contar, y me emociona seguir compartiéndolas.

El diseño lo trabajé en Figma. Vengo usándolo desde hace un tiempo, así que quise aplicar lo que he aprendido y cuidar los detalles. Siempre se puede mejorar, pero fue lindo ver cómo todo iba tomando forma.

Las animaciones también tienen su historia. Son detalles pequeños, pero ayudan a que todo se sienta más natural. Y sí, lograr que funcione sin conexión me dio más trabajo del que imaginaba, pero valió la pena.

Si algo de esto te llama la atención o tienes alguna duda, escríbeme. Siempre es bueno saber qué piensan los demás

Hasta la próxima.
