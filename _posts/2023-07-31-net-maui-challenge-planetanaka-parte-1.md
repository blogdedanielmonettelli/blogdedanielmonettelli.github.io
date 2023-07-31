---
title: .NET MAUI UI/UX Challenge - Planetanaka - Parte 1
author: danielmonettelli
date: 2023-07-31 16:30:00 -0500
categories: [.NET MAUI, Retos]
tags: [netmaui, animations]
image:
  path: 9_1_cover_publication.png
  width: 1200
  height: 630
  alt: .NET MAUI UI/UX Challenge - Planetanaka - Parte 1
---

<p style='font-size: 20px;
  color: light-grey; margin: 0px 0px 20px; font-weight: bold; font-style: italic;'>En esta publicación, me centraré en las animaciones básicas e intermedias que se pueden incorporar en proyectos .NET MAUI. </p>

> **INFO:** La autenticidad de mis publicaciones es mi compromiso más sincero. Siempre inicio plasmando mis ideas con mis propias palabras y, ocasionalmente, empleo GPT-3.5 para mejorar el contenido. De esta forma, busco asegurar la máxima claridad en mis escritos. Si en el futuro vuelvo a utilizar GPT-3.5 u otra herramienta similar, lo compartiré abiertamente para mantener la transparencia en todo momento.
{: .prompt-info }

Esta publicación es mi humilde aporte de conocimiento para enriquecer a nuestra amada comunidad. Expreso mi más profundo agradecimiento a Matt, el organizador del increíble evento .NET MAUI UI July, por brindarme esta maravillosa oportunidad de participar y compartir. Juntos, podemos construir un futuro emocionante en el mundo de la tecnología y dejar una huella duradera en el progreso colectivo.

---

## Animaciones en Planetakuna

La aplicación de Planetanaka cuenta con una amplia variedad de animaciones, que abarcan desde las básicas hasta las intermedias. Es relevante destacar que crear animaciones en .NET MAUI es un proceso sencillo y versátil, ya que podemos implementarlas tanto en el code-behind de la capa de la vista como dentro de los viewmodels, y precisamente esta última es la forma en la que trabajé dicha aplicación.

{% include embed/youtube.html id='L6wGSI-DanM' %}

---

## Animaciones en WelcomePage

![9-1-parts-of-welcomepage](9_1_parts_of_welcomepage.png)
_Parts of WelcomePage_

La imagen anterior muestra las diferentes partes que componen la WelcomePage. En esta página, la animación se centra en el control de imagen y cuyo x:Name es planetEarth. A continuación, me enfocaré en explicar el comportamiento de esta animación en detalle

Para que el control de imagen pueda interactuar con las animaciones que se incorporarán en el viewmodel, es fundamental crear un puente y asegurarse de igualar las propiedades tanto en la vista como en el viewmodel.

```csharp
public partial class WelcomePage : ContentPage
{
   private readonly WelcomeViewModel vm = new();

    public WelcomePage()
    {
        InitializeComponent();

        vm.PlanetEarth = planetEarth;
    }
}
```
{: file='Planetanaka/Views/WelcomePage.xaml.cs'}

El método parcial OnPlanetEarthChanged() anima la propiedad PlanetEarth cuando cambia su valor. La animación hace que la imagen rote suavemente desde 0 grados hasta 180 grados en los primeros 0.5 utilizando una función de aceleración cúbica de entrada (CubicIn). Luego, en los siguientes 0.5, rota desde 180 grados hasta 360 grados con una función de aceleración cúbica de salida (CubicOut). Esta animación se repite infinitamente en ciclos de 8 segundos, generando un efecto visual atractivo en la aplicación. 

```csharp
[ObservableProperty]
private Image planetEarth;

partial void OnPlanetEarthChanged(Image value)
{
    try
    {
        if (value is not null)
        {
            new Animation
            {
                { 0, 0.5, new Animation(v => PlanetEarth.Rotation = v, 0, 180, Easing.CubicIn) },
                { 0.5, 1, new Animation(v => PlanetEarth.Rotation = v, 180, 360, Easing.CubicOut) }
            }.Commit(PlanetEarth, "PlanetEarthAnimation", length: 8000, repeat: () => true);
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine
            ($"An error has occurred in OnPlanetEarthChanged(): " +
            $"{ex.Message}");
    }
}
```
{: file='Planetanaka/ViewModel/WelcomeViewModel.cs'}

Al invocar el comando StartExplorationAsync(), se detiene cualquier animación en el control de imagen 'PlanetEarth' y luego navega a la página 'PlanetsLookoutPage' de forma asincrónica.

```csharp
[RelayCommand]
private async Task StartExplorationAsync()
{
    PlanetEarth.AbortAnimation("PlanetEarthAnimation");
    await Shell.Current.GoToAsync("PlanetsLookoutPage", true);
}
```
{: file='Planetanaka/ViewModel/WelcomeViewModel.cs'}

---

## Animaciones en PlanetsLookoutPage

![9-1-parts-of-planetslookoutpage](9_1_parts_of_planetslookoutpage.png)
_Parts of PlanetsLookoutPage_

La imagen anterior muestra todas las diferentes partes que componen PlanetsLookoutPage. Al seleccionar un planeta, la animación se refleja en el control de imagen, junto con el título y la descripción del planeta en la parte posterior, creando un efecto visual cautivador.

Al igual que en el viewmodel anterior, se ha creado un puente para las propiedades respectivas que serán animadas.

El método OnSelectedPlanetChanging es un evento que se dispara cuando cambia la selección de un planeta en la aplicación. El objetivo es animar la transición entre el planeta seleccionado anteriormente y el nuevo planeta.

El método está dividido en tres partes, dependiendo de si hay un valor antiguo (planeta seleccionado previamente) o no. Si hay un valor antiguo, se inician dos animaciones: "MixAnimationsBefore" y "MixAnimationsAfter". Si no hay un valor antiguo, se inicia una animación "MixAnimationsInitial".

En la animación "MixAnimationsBefore", se utilizan diferentes valores para realizar transformaciones en tres elementos visuales: una imagen (ImgBigPlanet) del planeta, y dos etiquetas (LblNamePlanet y LblDescriptionPlanet). La imagen rota 720 grados menos un valor v multiplicado por 720, mientras que la escala disminuye su tamaño desde 1 hasta 1 menos el valor v. Las etiquetas también disminuyen su opacidad desde 1 hasta 1 menos el valor v. Esta animación utiliza un efecto de "CubicIn" para una aceleración gradual al inicio.

Cuando finaliza "MixAnimationsBefore", se actualiza la imagen del planeta con una nueva imagen usando una URL proporcionada, y también se actualizan los textos de las etiquetas con información del nuevo planeta seleccionado.

Luego, se inicia la segunda animación "MixAnimationsAfter". Esta animación es similar a "MixAnimationsBefore", pero utiliza un efecto de "CubicOut" para una desaceleración gradual al final de la animación.

En el caso de que no haya un valor antiguo, se ejecuta "MixAnimationsInitial". Esta animación es similar a "MixAnimationsBefore", pero se utiliza para mostrar el primer planeta seleccionado, sin necesidad de cambios en la rotación y escala del planeta.

```csharp
partial void OnSelectedPlanetChanging(Planet oldValue, Planet newValue)
{
    try
    {
        if (oldValue is not null)
        {
            new Animation
            {
                { 0, 0.5, new Animation(v =>
                  {
                      ImgBigPlanet.Rotation = 720 - v * 720;
                      ImgBigPlanet.Scale = 1 - v;
                      LblNamePlanet.Opacity = 1 - v;
                      LblDescriptionPlanet.Opacity = 1 - v;
                  }, 0, 1, Easing.CubicIn, finished: ()=>
                  {
                      ImgBigPlanet.Source = ImageSource.FromUri(new Uri($"https://raw.githubusercontent.com/danielmonettelli/MyResources/main/Planetakuna_Resources/{newValue.Image_Planet}@10x.png"));
                      LblNamePlanet.Text= newValue.Name_Planet;
                      LblDescriptionPlanet.Text = newValue.Description_Planet;
                  })
                }
            }.Commit(ImgBigPlanet, "MixAnimationsBefore", length: 2500);
            new Animation
            {
                { 0.5, 1, new Animation(v =>
                  {
                      ImgBigPlanet.Rotation = v * 720;
                      ImgBigPlanet.Scale = v;
                      LblNamePlanet.Opacity = v;
                      LblDescriptionPlanet.Opacity = v;
                  }, 0, 1, Easing.CubicOut)
                },
            }.Commit(ImgBigPlanet, "MixAnimationsAfter", length: 2500);
        }
        else
        {
            new Animation
            {
                { 0, 0.5, new Animation(v =>
                  {
                      ImgBigPlanet.Rotation = v * 720;
                      ImgBigPlanet.Scale = v;
                      LblNamePlanet.Opacity = v;
                      LblDescriptionPlanet.Opacity = v;
                  }, 0, 1, Easing.CubicOut, finished: ()=>
                  {
                      ImgBigPlanet.Source = ImageSource.FromUri(new Uri($"https://raw.githubusercontent.com/danielmonettelli/MyResources/main/Planetakuna_Resources/{newValue.Image_Planet}@10x.png"));
                      LblNamePlanet.Text= newValue.Name_Planet;
                      LblDescriptionPlanet.Text = newValue.Description_Planet;
                  })
                }
            }.Commit(ImgBigPlanet, "MixAnimationsInitial", length: 3000);
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine
            ($"An error has occurred in OnSelectedPlanetChanging(): " +
            $"{ex.Message}");
    }
}
```
{: file='Planetanaka/ViewModel/PlanetViewModel.cs'}

---

## Repositorio

El proyecto es de código abierto y puedes verlo haciendo clic en la siguiente imagen. ¡No dudes en echar un vistazo!

[![8-x-github-repository](https://raw.githubusercontent.com/blogdedanielmonettelli/blogdedanielmonettelli.github.io/main/assets/img/images/9_x_github_repository.png)](https://github.com/danielmonettelli/netmaui-planetanaka-app-challenge){:target="_blank"}

---

## Recursos

- <a href="https://learn.microsoft.com/es-es/dotnet/maui/user-interface/animation/basic" target="_blank">Animación básica - Documentación oficial de .NET MAUI</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/maui/user-interface/animation/easing" target="_blank">Funciones de aceleración - Documentación oficial de .NET MAUI</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/maui/user-interface/animation/custom" target="_blank">Animación personalizada - Documentación oficial de .NET MAUI</a>.
- <a href="https://github.com/mdc-maui/mdc-maui" target="_blank">MDC-MAUI - Material design components library</a>.
- <a href="https://learn.microsoft.com/es-es/dotnet/maui/user-interface/controls/border?view=net-maui-7.0" target="_blank">Borde - Documentación oficial de .NET MAUI</a>.
- <a href="https://www.youtube.com/watch?v=oQluWTag-e4" target="_blank">MVVM is easier than ever before with Source Generators, .NET 7, & the MVVM Toolkit - .NET Conf 2022</a>.

---

## Publicación en inglés

- <a href="https://danielmonettelli.github.io/posts/net-maui-challenge-planetanaka-part-1/" target="_blank">.NET MAUI UI/UX Challenge - Planetanaka - Part 1</a>.

---

## Conclusiones

Las animaciones, como sutiles pinceladas de arte digital, ocupan un lugar insigne en el universo del diseño de la interfaz de usuario (UI) y la experiencia del usuario (UX) en las aplicaciones. Sin embargo, al ser amantes de la estética, sabemos que 'menos es más' y que su uso no debe ser desplegado indiscriminadamente a lo largo de la interfaz. El secreto para cautivar sin agobiar radica en el equilibrio exquisito en la duración de cada animación, permitiendo al usuario deleitarse en cada interacción con armonía y paciencia refinadas.

Siéntete libre de darme tu opinión y con la ayuda de mi repositorio, saca tus propias conclusiones. Si tiene alguna pregunta o sugerencia constructiva, me gustaría mucho leerla. Gracias por tu tiempo.
