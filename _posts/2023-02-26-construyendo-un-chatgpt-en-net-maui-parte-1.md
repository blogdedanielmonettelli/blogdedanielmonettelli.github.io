---
title: Construyendo un ChatGPT en .NET MAUI Parte 1
author: danielmonettelli
date: 2023-02-26 22:30:00 -0500
categories: [.NET MAUI, Retos]
tags: [netmaui, openai, httpclient]
image:
  path: 8_1_cover_publication.png
  width: 1200
  height: 630
  alt: Construyendo un ChatGPT en .NET MAUI Parte 1
---

<p style='font-size: 20px;
  color: light-grey; margin: 0px 0px 20px; font-weight: bold; font-style: italic;'>En esta primera parte de mi publicación, me enfocaré en cómo utilizar las APIs de OpenAI a través de solicitudes HTTP POST, tanto para obtener respuestas de texto(completions) como para generar imágenes con la tecnología de OpenAI.</p>

> **INFO:** Es importante para mí que tengas plena confianza en la autenticidad de mis publicaciones. Por ello, quiero informarte de que esta publicación contó con la ayuda de GPT-3.5. Sin embargo, siempre utilizo mis criterios para asegurarme de que la publicación sea clara y fácil de entender. En caso de que en el futuro vuelva a utilizar esta herramienta o cualquier otra, te lo haré saber de antemano para asegurar la transparencia en todo momento.
{: .prompt-info }
> **ADVERTENCIA:** Esta publicación enfatiza la importancia de utilizar la aplicación de manera ética y responsable, la cual ha sido desarrollada en la plataforma .NET MAUI y se basa en la tecnología de OpenAI. Si bien no me considero una autoridad moral, reconozco que he tenido errores en el pasado y que seguiré aprendiendo y evolucionando como ser humano en el futuro. Esta misma filosofía se aplica al ámbito tecnológico, donde tanto los usuarios como los desarrolladores deben ser conscientes de las posibles implicaciones éticas y sociales de sus acciones. Es crucial considerar el impacto de la tecnología en la sociedad y trabajar juntos para garantizar su uso adecuado y responsable. Para ello, es necesario adoptar estándares éticos y responsables, como la transparencia, la equidad y la responsabilidad. La transparencia implica una divulgación adecuada y clara de los procesos y los resultados del uso de la tecnología. La equidad, por su parte, requiere que las aplicaciones sean accesibles y justas para todas las personas, independientemente de su origen, género, raza o creencias. Y la responsabilidad implica asumir las consecuencias de nuestras acciones y decisiones, así como la responsabilidad social de garantizar que la tecnología no cause daño a la sociedad. En resumen, el uso ético y responsable de la tecnología es esencial para avanzar hacia una sociedad más justa y equitativa. Es necesario que tanto los usuarios como los desarrolladores adopten estándares éticos y responsables, y trabajemos juntos para garantizar que la tecnología se utilice para el bien común.
{: .prompt-warning }

Quiero destacar que toda la información presentada en esta publicación se debe a la valiosa charla impartida por <a href="https://twitter.com/darkicebeam" target="_blank">Luis Beltran, PhD</a>, quien nos enseñó cómo construir un ChatGPT en .NET MAUI con las APIs de OpenAI. Agradezco su contribución y espero que esta publicación pueda ayudar a otros desarrolladores a implementar esta tecnología en sus propias aplicaciones de .NET.

---

## Parámetros esenciales para la lógica empresarial con las APIs de OpenAI

Antes de realizar solicitudes HTTP POST, es fundamental establecer un modelo de negocio apropiado. No es necesario emplear todos los parámetros que brindan las APIs de OpenAI, sino aquellos que resulten relevantes para la lógica de la aplicación. A continuación te mostraré algunas imágenes que ilustran cómo aplicar esta idea en la generación de texto _(completions)_ e imágenes _(image generation)_.

![8-1-selection-of-parameters-in-completions](8_1_selection_of_parameters_in_completions.png)
_Selection of parameters in completions_

![8-1-selection-of-parameters-in-image-generation](8_1_selection_of_parameters_in_image_generation.png)
_Selection of parameters in image generation_

---

## HTTP POST Request and Response in OpenAI API - Esquema

Las siguientes imágenes detallan el proceso de generación de texto _(completions)_ e imágenes _(image generations)_ a través de la OpenAI APIs.

> **TIP:** Para una mejor comprensión, lee cada parte e interactúa con las imágenes.
{: .prompt-tip }

1. Para iniciar el proceso, el usuario debe realizar una consulta que desencadena una solicitud HTTP POST. Es importante destacar que dicha solicitud debe incluir una cabecera de autorización y especificar el tipo de contenido, así como también serializar un objeto de la clase `CompletionRequest.cs`_(Modo texto)_ u `GenerationRequest.cs`_(Modo imagen)_ con los parámetros necesarios en un formato JSON. Una vez que se ha preparado la solicitud, esta se enviará a la API de OpenAI para su procesamiento.

2. En caso de que la respuesta de la API sea exitosa, se devolverá un cuerpo de respuesta en el campo `choices`_(Modo texto)_ u `data`_(Modo imagen)_. Este cuerpo de respuesta se deserializa mediante un objeto de la clase `CompletionResponse.cs`_(Modo texto)_ u `GenerationResponse.cs`_(Modo imagen)_, permitiendo así que el usuario reciba la respuesta en la aplicación.

![8-1-openai-api-request-and-response-flow-with-parameters-in-mode-write](8_1_openai_api_request_and_response_flow_with_parameters_in_mode_write.png)
_OpenAI API request and response flow with parameters in mode write_

![8-1-openai-api-request-and-response-flow-with-parameters-in-mode-image](8_1_openai_api_request_and_response_flow_with_parameters_in_mode_image.png)
_OpenAI API request and response flow with parameters in mode image_

---

## HTTP POST Request and Response in OpenAI API - Capa de servicio

Ahora te mostraré de manera práctica cómo funcionan los flujos de solicitud y respuesta HTTP POST con las APIs de OpenAI para generar texto e imágenes en la capa de servicio de la aplicación. Acompáñame mientras exploramos el código necesario para llevar a cabo estas solicitudes:

> **TIP:** Para una comprensión más clara, te sugiero que leas una parte y luego verifiques el código. Asegúrate de hacer una pausa en cada proceso.
{: .prompt-tip }

### Constructor

En el constructor de la clase OpenAIService, se establece una conexión a la API de OpenAI mediante la creación de un objeto `HttpClient`, el cual será utilizado para enviar peticiones HTTP a la API.

1. Se establece la dirección base de la API en la propiedad `BaseAddress` del objeto HttpClient.
2. Se agrega un encabezado de autorización a una solicitud HTTP a la API de OpenAI utilizando el token de acceso y el esquema de autorización `"Bearer"`. Este esquema es comúnmente utilizado en la autorización de API para enviar el token de acceso al servidor protegido, permitiendo autenticar y autorizar la solicitud de acceso al recurso solicitado.
3. Se agrega un encabezado `Accept` que indica que el cliente espera recibir una respuesta en formato JSON.

```csharp
public class OpenAIService : IOpenAIService
{
    HttpClient client;

    public OpenAIService()
    {
        client = new HttpClient();
        // Part 1
        client.BaseAddress = new Uri(APIConstants.OpenAIUrl);

        // Part 2
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", APIConstants.OpenAIToken);
        // Part 3
        client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    }
}
```
{: file='ChatGPT/Services/OpenAIService.cs(Constructor)'}

#### Métodos AskQuestion y CreateImage

Estos métodos utilizan las APIs de OpenAI para generar texto e imágenes a partir de una cadena de consulta (query) proporcionada como parámetro.

1. En primer lugar, se crea un objeto de la clase `CompletionRequest`_(para el método AskQuestion)_ u `GenerationRequest`_(para el método CreateImage)_ y se establece su propiedad `Prompt` con la cadena de consulta proporcionada como parámetro.

2. A continuación, se utiliza la clase `JsonSerializer` para serializar la instancia de `CompletionRequest`_(para el método AskQuestion)_ u `GenerationRequest`_(para el método CreateImage)_ a formato JSON y se almacena en la variable `body`.

3. Luego, se crea una nueva instancia de la clase `StringContent`, que se utiliza para enviar datos en formato JSON a través de una solicitud HTTP, donde `body` es la cadena de texto que contiene los datos en formato JSON, `Encoding.UTF8` especifica la codificación de caracteres utilizada y `"application/json"` especifica el tipo de contenido que se está enviando.

4. El método `PostAsync` de la clase HttpClient se llama con la URL de punto final de la API de OpenAI para generar texto o imágenes y la variable `content` que contiene el cuerpo de la solicitud POST. Esto envía la solicitud HTTP POST a la API de OpenAI con el cuerpo en formato JSON y devuelve una respuesta.

5. Si la respuesta es exitosa, se lee y deserializa el contenido de la respuesta utilizando el método `ReadFromJsonAsync` y se almacena en la variable `data`.

6. Luego, se accede a la propiedad `Choices`(para el método AskQuestion) u `Data`(para el método CreateImage) de la variable `data` y se utiliza el método FirstOrDefault() para obtener el primer objeto `Text`_(para el método AskQuestion)_ u `Url`_(para el método CreateImage)_ disponible. Este objeto se devuelve como resultado en cada método.

7. Si la respuesta no es exitosa, el método devuelve un valor predeterminado.

```csharp
public class OpenAIService : IOpenAIService
{
    HttpClient client;

    JsonSerializerOptions options = new() { PropertyNameCaseInsensitive = true };

    public async Task<string> AskQuestion(string query)
    {
        // Part 1
        var completion = new CompletionRequest()
        {
            Prompt = query
        };

        // Part 2 - Serialize parameters for the response
        var body = JsonSerializer.Serialize(completion);
        // Part 3
        var content = new StringContent(body, Encoding.UTF8, "application/json");

        // Part 4 - HTTP POST request with headers and parameters
        var response = await client.PostAsync(APIConstants.OpenAIEndpoint_Completions, content);

        if (response.IsSuccessStatusCode)
        {
            // Part 5 - Deserialize the response to a object of class
            var data = await response.Content.ReadFromJsonAsync<CompletionResponse>(options);
            // Part 6
            return data?.Choices?.FirstOrDefault().Text;
        }

        // Part 7
        return default;
    }
}
```
{: file='ChatGPT/Services/OpenAIService.cs(AskQuestion)'}

```csharp
public class OpenAIService : IOpenAIService
{
    HttpClient client;

    JsonSerializerOptions options = new() { PropertyNameCaseInsensitive = true };

    public async Task<string> CreateImage(string query)
    {
        // Part 1
        var generation = new GenerationRequest()
        {
            Prompt = query
        };

        // Part 2 - Serialize parameters for the response
        var body = JsonSerializer.Serialize(generation);
        // Part 3
        var content = new StringContent(body, Encoding.UTF8, "application/json");

        // Part 4 - HTTP POST request with headers and parameters
        var response = await client.PostAsync(APIConstants.OpenAIEndpoint_Generations, content);

        if (response.IsSuccessStatusCode)
        {
            // Part 5 - Deserialize the response to a object of class
            var data = await response.Content.ReadFromJsonAsync<GenerationResponse>(options);
            // Part 6
            return data.Data?.FirstOrDefault()?.Url;
        }

        // Part 7
        return default;
    }
}
```
{: file='ChatGPT/Services/OpenAIService.cs(CreateImage)'}

---

## Repositorio

El proyecto es de código abierto y puedes verlo haciendo clic en la siguiente imagen. ¡No dudes en echar un vistazo!

[![8-x-github-repository](https://raw.githubusercontent.com/blogdedanielmonettelli/blogdedanielmonettelli.github.io/main/assets/img/images/8_x_github_repository.png)](https://github.com/danielmonettelli/dotnetmaui-chatgpt-oss){:target="_blank"}

---

## Recursos

- <a href="https://www.youtube.com/watch?v=JE_SdgP-jJo" target="_blank">Conversando con ChatGPT en .NET MAUI - Video tutorial realizado por Luis Beltran, PhD</a>.
- <a href="https://platform.openai.com/docs/api-reference/completions" target="_blank">Completions - Documentación oficial de OpenAI</a>.
- <a href="https://platform.openai.com/docs/api-reference/images" target="_blank">Image generation - Documentación oficial de OpenAI</a>.

---

## Publicación en inglés

- <a href="https://danielmonettelli.github.io/posts/building-a-chatgpt-in-net-maui-part-1/" target="_blank">Building a ChatGPT in .NET MAUI Part 1</a>.

---

## Conclusiones

Una de las formas principales de interactuar con las APIs de OpenAI es mediante solicitudes HTTP, lo que permite generar contenido enriquecido a partir de datos de entrada. Para implementar estas solicitudes, es fundamental comprender los principios de serialización y deserialización para aprovechar al máximo las capacidades de cualquier tipo de APIs. Al adoptar prácticas adecuadas y utilizar herramientas efectivas, se pueden generar soluciones exitosas y eficientes.

Siéntete libre de darme tu opinión y con la ayuda de mi repositorio, saca tus propias conclusiones. Si tiene alguna pregunta o sugerencia constructiva, me gustaría mucho leerla. Gracias por tu tiempo.
