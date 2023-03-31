---
title: Construyendo un ChatGPT en .NET MAUI Parte 2
author: danielmonettelli
date: 2023-03-31 11:10:00 -0500
categories: [.NET MAUI, Retos]
tags: [netmaui, openai, ui/ux, border, data templates]
image:
  path: 8_2_cover_publication.png
  width: 1200
  height: 630
  alt: Construyendo un ChatGPT en .NET MAUI Parte 2
---

<p style='font-size: 20px;
  color: light-grey; margin: 0px 0px 20px; font-weight: bold; font-style: italic;'>En esta segunda parte de mi publicación, el enfoque se centra en la interfaz de usuario de la aplicación y su interacción con el ViewModel.</p>

> **INFO:** Es importante para mí que tengas plena confianza en la autenticidad de mis publicaciones. Por ello, quiero informarte de que esta publicación contó con la ayuda de GPT-3.5. Sin embargo, siempre utilizo mis criterios para asegurarme de que la publicación sea clara y fácil de entender. En caso de que en el futuro vuelva a utilizar esta herramienta o cualquier otra, te lo haré saber de antemano para asegurar la transparencia en todo momento.
{: .prompt-info }
> **ADVERTENCIA:** Esta publicación enfatiza la importancia de utilizar la aplicación de manera ética y responsable, la cual ha sido desarrollada en la plataforma .NET MAUI y se basa en la tecnología de OpenAI. Si bien no me considero una autoridad moral, reconozco que he tenido errores en el pasado y que seguiré aprendiendo y evolucionando como ser humano en el futuro. Esta misma filosofía se aplica al ámbito tecnológico, donde tanto los usuarios como los desarrolladores deben ser conscientes de las posibles implicaciones éticas y sociales de sus acciones. Es crucial considerar el impacto de la tecnología en la sociedad y trabajar juntos para garantizar su uso adecuado y responsable. Para ello, es necesario adoptar estándares éticos y responsables, como la transparencia, la equidad y la responsabilidad. La transparencia implica una divulgación adecuada y clara de los procesos y los resultados del uso de la tecnología. La equidad, por su parte, requiere que las aplicaciones sean accesibles y justas para todas las personas, independientemente de su origen, género, raza o creencias. Y la responsabilidad implica asumir las consecuencias de nuestras acciones y decisiones, así como la responsabilidad social de garantizar que la tecnología no cause daño a la sociedad. En resumen, el uso ético y responsable de la tecnología es esencial para avanzar hacia una sociedad más justa y equitativa. Es necesario que tanto los usuarios como los desarrolladores adopten estándares éticos y responsables, y trabajemos juntos para garantizar que la tecnología se utilice para el bien común.
{: .prompt-warning }

Quiero destacar que toda la información presentada en esta publicación se debe a la valiosa charla impartida por <a href="https://twitter.com/darkicebeam" target="_blank">Luis Beltran, PhD</a>, quien nos enseñó cómo construir un ChatGPT en .NET MAUI con las APIs de OpenAI. Agradezco su contribución y espero que esta publicación pueda ayudar a otros desarrolladores a implementar esta tecnología en sus propias aplicaciones de .NET.

---

## Partes de ConversationView

La vista de ConversationView se compone de tres secciones distintas: la sección superior, la sección central y la sección inferior. La sección superior incluye la imagen de marca y el título de la aplicación, junto con botones para generar texto e imágenes. La sección central presenta la conversación entre el usuario y el chatbot, presentada en un CollectionView. Y, por último, la sección inferior incluye un campo para introducir texto y botones para enviar o capturar audio por voz.

![8-2-parts-of-conversationview](8_2_parts_of_conversationview.png)
_Parts of Conversationview_

---

## Animación de Lottie cuando la lista de mensajes está vacía

Cuando la lista de mensajes de la aplicación está vacía, se activa automáticamente una animación Lottie que tiene como objetivo mejorar la experiencia del usuario y añadir un toque de personalidad a la interfaz de la aplicación. Esta animación se reproduce de manera continua y llama la atención del usuario, indicándole que puede enviar una consulta al chatbot en cualquier momento. Una vez que el usuario envía un mensaje o recibe una respuesta del chatbot, la animación se detiene de forma automática, permitiendo que el usuario se centre en la conversación con el chatbot. De esta manera, la animación es una herramienta eficaz para atraer la atención del usuario y guiarlo hacia el uso del chatbot.

![8-2-lottie-animation-when-message-list-is-empty](8_2_lottie_animation_when_message_list_is_empty.png)
_Lottie animation when message list is empty_

{% include embed/youtube.html id='yMCbGVo38YE' %}

---

## Interacción de la UI con el ViewModel

La forma en que el chatbot proporciona información no depende únicamente de las consultas que los usuarios ingresen en el campo de entrada de texto, sino también de los botones de la parte superior que cambian la respuesta del chatbot, ya sea a través de texto o imágenes. Detrás de escena, el ViewModel se encarga de administrar la lógica de la aplicación y comunicarse con el servicio de OpenAI. A continuación, se presenta una imagen que resume cómo fluye la información entre la interfaz de usuario y el ViewModel.

![8-2-command-flow-and-methods-in-write-and-image-mode](8_2_command_flow_and_methods_in_write_and_image_mode.png)
_Command flow and methods in write and image mode_

{% include embed/youtube.html id='ZtuSfWNcQ_g' %}

### Para el método AskQuestion

1. Se establece la instancia `CurrentCommand` en su estado inicial, que es `AskQuestion`. Además, se ajusta la `OpacityModeMessage` a 1 y la de `OpacityModeImage` a 0.5, y se muestra un toast _(mensaje breve y temporal)_ que indica que está en modo de escritura.

2. Se crea una nueva instancia de `AsyncRelayCommand` que llama al método `AskQuestionAsync`.

3. Cuando se llama al método `AskQuestionAsync`, se establece el indicador `IsTextActive` en `true` y el indicador `IsImageActive` en `false`.

4. El método `QueryManagerAsync` es llamado con la función `_openAIService.AskQuestion` como argumento.

5. En `QueryManagerAsync`, se agrega el mensaje de usuario a la lista de mensajes, se establece el indicador `isUserMessage` en `true` y se llama a la función `_openAIService.AskQuestion` con la consulta del usuario como argumento.

6. Se agrega la respuesta del bot a la lista de mensajes, se establece el indicador `isUserMessage` en `false` y se desplaza la vista hacia el final de la lista de mensajes.

{% include embed/youtube.html id='fBYBOybi7dc' %}

### Para el método CreateImage

1. Cuando el usuario cambia al modo imagen a través del botón correspondiente, se llama al método `CreateImage`. Dentro de este método, se establece la `OpacityModeMessage` en 0.5 y la `OpacityModeImage` en 1, y se muestra un toast _(mensaje breve y temporal)_ indicando que se ha activado el modo de imagen.

2. Después, se asigna a la propiedad `CurrentCommand` un nuevo comando asincrónico (`AsyncRelayCommand`) que, al activarse, ejecutará el método `CreateImageAsync`.

3. Cuando se llama al método `CreateImageAsync`, se establece el indicador `IsTextActive` en `false` y el indicador `IsImageActive` en `true`.

4. El método `QueryManagerAsync` es llamado con la función `_openAIService.CreateImage` como argumento.

5. En `QueryManagerAsync`, se agrega el mensaje de usuario a la lista de mensajes, se establece el indicador `isUserMessage` en `true` y se llama a la función `_openAIService.CreateImage` con la consulta del usuario como argumento.

6. Se agrega la respuesta del bot a la lista de mensajes, se establece el indicador `isUserMessage` en `false` y se desplaza la vista hacia el final de la lista de mensajes.

{% include embed/youtube.html id='N9dLwE2z0dc' %}

---

## Data templates en ConversationView

La distinción entre los mensajes enviados por el usuario y los mensajes enviados por el chatbot se realiza mediante el uso de dos plantillas diferentes en la UI. La plantilla de mensaje de usuario _(`UserMessageItemTemplate`)_ es utilizada para mostrar los mensajes enviados por el usuario, mientras que la plantilla de mensaje de chatbot _(`BotMessageItemTemplate`)_ es utilizada para mostrar los mensajes enviados por el chatbot. Esto permite una mejor organización visual de la conversación y facilita la comprensión por parte del usuario de los mensajes que recibe.

![8-2-templates-in-conversationview](8_2_templates_in_conversationview.png)
_Templates in Conversationview_

---

## Selección de temas

Cuando se hace clic en el branding de la aplicación, se activa el método `SelectTheme`. Este método, que es un método asíncrono, permite cambiar el tema de la aplicación y realiza una animación en la vista de conversación para que la transición sea más fluida.

Primero, el método obtiene el tema actual de la aplicación y establece el nuevo tema como el opuesto al actual _(oscuro o claro)_. Luego, establece el nuevo tema como el tema de la aplicación.

A continuación, se ejecuta una animación en la vista de conversación utilizando el método `ScaleTo`. Durante 250 milisegundos, la vista se reduce a un 95% de su tamaño original, luego se amplía a un 105% de su tamaño original durante otros 250 milisegundos y, finalmente, se restablece al tamaño original durante otros 250 milisegundos. La animación utiliza la función de aceleración `CubicOut` para reducir la velocidad de la animación al final.

```csharp
[RelayCommand]
private async Task SelectTheme()
{
    AppTheme currentTheme = Application.Current.RequestedTheme;
    AppTheme newTheme = currentTheme == AppTheme.Dark ? AppTheme.Light : AppTheme.Dark;
    Application.Current.UserAppTheme = newTheme;
    await ConversationView.ScaleTo(0.95, 250, Easing.CubicOut);
    await ConversationView.ScaleTo(1.05, 250, Easing.CubicIn);
    await ConversationView.ScaleTo(1, 250, Easing.CubicOut);
}
```
{: file='ChatGPT.ViewModels.ConversationViewModel.cs'}

{% include embed/youtube.html id='FA2_ql7q4Uk' %}

---

## Repositorio

El proyecto es de código abierto y puedes verlo haciendo clic en la siguiente imagen. ¡No dudes en echar un vistazo!

[![8-x-github-repository](https://raw.githubusercontent.com/blogdedanielmonettelli/blogdedanielmonettelli.github.io/main/assets/img/images/8_x_github_repository.png)](https://github.com/danielmonettelli/dotnetmaui-chatgpt-oss){:target="_blank"}

---

## Recursos

- <a href="https://medium.com/@reyes.leomaris/trabajando-con-gridlayout-in-xamarin-forms-db3506001eaa" target="_blank">Trabajando con GridLayout en Xamarin Forms - Escrito por Leomaris Reyes</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/maui/fundamentals/datatemplate?view=net-maui-7.0" target="_blank">Plantillas de datos - Documentación oficial de .NET MAUI</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/maui/user-interface/controls/border?view=net-maui-7.0" target="_blank">Borde - Documentación oficial de .NET MAUI</a>.
- <a href="https://www.youtube.com/watch?v=oQluWTag-e4" target="_blank">MVVM is easier than ever before with Source Generators, .NET 7, & the MVVM Toolkit - .NET Conf 2022</a>.

---

## Publicación en inglés

- <a href="https://danielmonettelli.github.io/posts/building-a-chatgpt-in-net-maui-part-2/" target="_blank">Building a ChatGPT in .NET MAUI Part 2</a>.

---

## Conclusiones

En resumen, para construir la UI de chatbot en .NET MAUI se han mostrado las tres secciones distintas que componen la vista de ConversationView, así como la plantilla de mensaje de usuario _(UserMessageItemTemplate)_ y la plantilla de mensaje de chatbot _(BotMessageItemTemplate)_. Además, se ha presentado una animación Lottie que se muestra cuando la lista de mensajes está vacía, lo que contribuye a mejorar la experiencia del usuario.

Siéntete libre de darme tu opinión y con la ayuda de mi repositorio, saca tus propias conclusiones. Si tiene alguna pregunta o sugerencia constructiva, me gustaría mucho leerla. Gracias por tu tiempo.
