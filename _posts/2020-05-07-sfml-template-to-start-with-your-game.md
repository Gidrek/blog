---
layout: post
title:  SFML Template to start with your game
date:   2020-05-07 15:00:00 -0600
description: SFML Template to start with your game
categories: GameDev
---

If you have setup SFML  with the [tutorial that I wrote](sfml-2-5-1-setup-on-macos-with-clion), 
the example is easy but is not very useful if you want to create more complex games.

In this post, I going to show you how I create a simple Game class to have more control about the game loop, 
the user input and the frame rate.

First, you need to create a **Game.h** and **Game.cpp** files. In your Game.h write this code

```cpp
#ifndef GAME_H
#define GAME_H

#include <SFML/Graphics.hpp>

class Game
{
public:
    Game();
    void Run();

private:
    void processEvents();
    void update(sf::Time deltaTime);
    void render();
    void handlePlayerInput(sf::Keyboard::Key key, bool isPressed);

    sf::RenderWindow mWindow;

    static const sf::Time TimePerFrame;
};

#endif //GAME_H
```

This is the header of our Game class, as you can see, we have only a constructor and one public method run, all other methods are private to Game.

We have one method for process the SFML events and one for the player input. The **update** method take a delta 
time which is going to help us with the frame rate and with other stuffs.

And we have the **render** method, that is going to draw our game.

We need a **RenderWindow** to render our window 😛 and finally a **const** to help us with the frame rate.

Now, in the **Game.cpp** file, write the next code


```cpp
//
// Created by Giovanni Cortes on 07/05/20.
//

#include "Game.h"

const sf::Time Game::TimePerFrame = sf::seconds(1.f/60.f);

Game::Game()
: mWindow(sf::VideoMode(640, 480), "Your Awesome Game!")
{
}

void Game::Run()
{
    sf::Clock clock;
    sf::Time timeSinceLastUpdate = sf::Time::Zero;

    while (mWindow.isOpen())
    {
        timeSinceLastUpdate += clock.restart();
        while (timeSinceLastUpdate > TimePerFrame)
        {
            timeSinceLastUpdate -= TimePerFrame;
            processEvents();
            update(TimePerFrame);
        }
        render();
    }
}

void Game::processEvents()
{
    sf::Event event{};

    while (mWindow.pollEvent(event))
    {
        switch (event.type)
        {
            case sf::Event::KeyPressed:
                handlePlayerInput(event.key.code, true);
                break;
            case sf::Event::KeyReleased:
                handlePlayerInput(event.key.code, false);
                break;
            case sf::Event::Closed:
                mWindow.close();
                break;
            default:
                break;
        }

    }
}

void Game::update(sf::Time deltaTime)
{
    // TODO: Update your objects here
    // Example: mWindow.draw(mPlayer);
}

void Game::render()
{
    mWindow.clear();

    // TODO: Draw your objects here

    mWindow.display();
}

void Game::handlePlayerInput(sf::Keyboard::Key key, bool isPressed)
{
    // TODO: Key events for your game
    // Example:
    // if (key == sf::Keyboard::W)
    // {
    //     mIsMovingUp = isPressed;
    // }
}
```

The most important method, is the **run** method. We have locked our game loop to 60 frames per second thanks to line

```cpp
const sf::Time TimePerFrame = sf::seconds(1.f/60.f);
```

SFML has a `setFrameLimit()` method, but I prefer use mine for control and for learning.

The remains methods are self explanatory but if you have any doubt, you can write me to explain more in detail.


## Running the game loop

```cpp
#include "Game.h"

int main()
{
    Game game;
    game.Run();
    return 0;
}
```

As you can see, the code is very simple, we create a Game instance and run our game loop. If you run this, you are going to see this window

![running_SFML](/assets/img/gamedev/running_SFML.png)

Now, you can extend your game and have a better control of your game loop.
