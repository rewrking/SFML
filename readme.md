[![SFML logo](https://www.sfml-dev.org/images/logo.png)](https://www.sfml-dev.org)

# SFML — Simple and Fast Multimedia Library

SFML is a simple, fast, cross-platform and object-oriented multimedia API. It provides access to windowing, graphics, audio and network. It is written in C++, and has bindings for various languages such as C, .Net, Ruby, Python.

## Authors

  - Laurent Gomila — main developer (laurent@sfml-dev.org)
  - Marco Antognini — OS X developer (hiura@sfml-dev.org)
  - Jonathan De Wachter — Android developer (dewachter.jonathan@gmail.com)
  - Jan Haller (bromeon@sfml-dev.org)
  - Stefan Schindler (tank@sfml-dev.org)
  - Lukas Dürrenberger (eXpl0it3r@sfml-dev.org)
  - binary1248 (binary1248@hotmail.com)
  - Artur Moreira (artturmoreira@gmail.com)
  - Mario Liebisch (mario@sfml-dev.org)
  - And many other members of the SFML community

## Download

You can get the latest official release on [SFML's website](https://www.sfml-dev.org/download.php). You can also get the current development version from the [Git repository](https://github.com/SFML/SFML).

## Install

Follow the instructions of the [tutorials](https://www.sfml-dev.org/tutorials/), there is one for each platform/compiler that SFML supports.

## Learn

There are several places to learn SFML:

  * The [official tutorials](https://www.sfml-dev.org/tutorials/)
  * The [online API documentation](https://www.sfml-dev.org/documentation/)
  * The [community wiki](https://github.com/SFML/SFML/wiki/)
  * The [community forum](https://en.sfml-dev.org/forums/) ([French](https://fr.sfml-dev.org/forums/))

## Contribute

SFML is an open-source project, and it needs your help to go on growing and improving. If you want to get involved and suggest some additional features, file a bug report or submit a patch, please have a look at the [contribution guidelines](https://www.sfml-dev.org/contribute.php).

## Branch Notes:

Exposes `sf::priv::WindowImpl` so that one could use a separate library for window/events (SDL2 or GLFW). Keep in mind, this also disables SFML events entirely, so don't attempt to use them. Build using cmake with `SFML_CUSTOM_WINDOW=ON`, write a class that implements `sf::priv::WindowImpl`, and create the window with something like:

```cpp
// GameWindow extends sf::RenderWindow
void GameWindow::createWindowImpl(const bool inFullscreen, const sf::Uint32 inStyle)
{
    if (m_windowImpl != nullptr)
        return;

    sf::VideoMode mode;
    mode.width = m_windowResolution.x;
    mode.height = m_windowResolution.y;

    m_windowImpl = new SDLWindowImpl(mode, m_title, inStyle, m_contextSettings);
	
    // Use SDL_WINDOW_OPENGL flag w/ SDL_GL_SetAttribute, 
    // but do not create a SDL_Renderer (done by SFML)

    // m_windowImpl is injected here, so it will get deleted by SFML. Keeping a 
    // weak pointer is useful for calling implementation-specific things that 
    // aren't required by sf::priv::WindowImpl, like "setFullscreen". For example, 
    // GameWindow shouldn't make any calls directly to SDL2
    this->create(m_windowImpl, inFullscreen, mode, m_contextSettings);

    // This just calls SDL_GL_GetAttribute() to make sure the context is what 
    // is expected, but granted, you could do this however you like
    m_windowImpl->checkContext(); 
}
```
For SDL2, also make sure the window uses OpenGL exclusively, and everything initializes & destructs correctly. Write out your event handling somewhere (especially for SDL_WINDOWEVENT_CLOSE), otherwise you won't be able to do much. Take advantage of SDL_GameController and SDL_Haptic of course! That and dual-monitor support / more robust window handling are the primary reasons for doing something like this.

The setup is similar for GLFW, but you'll need to create the glfw callbacks that statically call your own event dispatcher.
