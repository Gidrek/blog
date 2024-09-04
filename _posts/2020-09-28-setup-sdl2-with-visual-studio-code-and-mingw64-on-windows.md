---
layout: post
title:  Setup SDL2 with Visual Studio Code and mingw64 on Windows
date:   2020-09-28 15:00:00 -0600
description: Setup SDL2 with Visual Studio Code and mingw64 on Windows
categories: GameDev
---

Recently I changed my setup from macOS to Windows, and I’d like to do some experiments with GameDev in Windows. 
So one of the first things that I did was try to set up my dev environment and test some SDL2 code.

This is a simple guide to how to setup SDL2 with Visual Studio Code and mingw64, 
for me worked and I hope that you can start with SDL2 in Windows.

## Installing the tools.

The first thing that we need to install is [Visual Studio Code editor](https://code.visualstudio.com/download). Download, install, and launch VS Code.

You need to **install the C/C++ plugin from Microsoft**.

![cpp-plugin-editor](/assets/img/gamedev/cpp-plugin-editor.png)

Now, you need to install mingw64 to use the compilers that are included like g++, if you want to use the Visual Studio compiler, 
you can change some of the settings here but for now we continue with mingw64.

For an easier installation, use the [MinGW-W64 Online Installer](https://sourceforge.net/projects/mingw-w64/files/), 
take note of the path where you installed your compiler, because we’re going to need it in the configuration.

**Note:** Be sure that you are added the bin directory of MinGW to your Windows PATH. If you don’t do this step, VS Code won’t be able to 
find the proper tools!

Now, we need to download [SDL2](https://www.libsdl.org/download-2.0.php). For our case, we have to download the Development libraries for MinGW

![sdl2-library-mingw](/assets/img/gamedev/sdl2-library-mingw.png)

And download the Runtime libraries to run our games.

![sdl2-runtime-libraries](/assets/img/gamedev/sdl2-runtime-libraries.png)

We are going to use the x64 version. Unzip those folders in a location that you can find because we are going to need it.

## Creating the SDL2 project

Now we are ready to start our SDL2 project. Create a folder for your project and open it with **VSCode**. 
Inside the folder, create another called **src**, here is where are going to save our code files. For our SDL2 and C++ configuration, 
create a **.vscode** folder. For the output of our compilation, create a **build** folder. Your editor should be like this

![sdl2-src-vscode-build-folder](/assets/img/gamedev/sdl2-src-vscode-build-folder.png)

The first thing that we need to create is our Task to compile our code. In the **.vscode** folder, create a new file and name it **tasks.json**. Inside the file, 
write the following configuration.

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "shell",
			"label": "SDL2",
			"command": "D:\\sdk\\mingw-w64\\x86_64-8.1.0-posix-seh-rt_v6-rev0\\mingw64\\bin\\g++.exe",
			"args": [
				"-g",
				"src\\*.cpp",
				"-o",
				"build\\game.exe",
				"-ID:/sdk/sdl2/mingw/x86_64/include",
				"-LD:/sdk/sdl2/mingw/x86_64/lib",
				"-lmingw32",
				"-lSDL2main",
				"-lSDL2",
				"-mwindows"
			],
			"options": {
				"cwd": "${workspaceFolder}"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": {
				"kind": "build",
				"isDefault": true
			}
		}
	]
}
```

**label** option is the name in which we are going to identify our task.

In the **command** option, we need to put the path to our g++ installation, that is where we install mingw64.

In the args options, we have some arguments that are important to notice. For example, `"build\\game.exe"`, 
is the name of our game and the folder where we’re going to save it.

`"-ID:/sdk/sdl2/mingw/x86_64/include"` is where we save our SDL2 files, this is for SDL2 include libraries, 
the same as `"-LD:/sdk/sdl2/mingw/x86_64/lib"`, is the path to SDL2 libraries.

`"-lmingw32"` is an option to compile with the mingw libraries, besides that said 32, we’re are using the 64 bits compiler. 
`"-lSDL2main", "-lSDL2", "-mwindows"` in this case, we are indicating to our compiler that we need to include the SDL2main and SDL2 
libraries and we don’t want the command prompt when launching our game.

Now, we need to create a **launch.json** file to launch our game and debugging.

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb)",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}\\build\\game.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "D:\\sdk\\mingw-w64\\x86_64-8.1.0-posix-seh-rt_v6-rev0\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "Enably pretty printing",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "SDL2"
        }
    ]
}
```

`"program": "${workspaceFolder}\\build\\game.exe"`, is the executable that we are going to launch, in our case is the same that our game.

`"miDebuggerPath": "D:\\sdk\\mingw-w64\\x86_64-8.1.0-posix-seh-rt_v6-rev0\\mingw64\\bin\\gdb.exe"` is the path where gdb.exe is.

`"preLaunchTask": "SDL2"` before launch our program, we need to run the task “SDL2”, this is the name that we put in the label option in tasks.json.

Now, as the last configuration step, we need to configure our C++ options to have a better Intellisense autocomplete and configuration of our compiler.

In Visual Studio Code, press **CTRL + SHIFT + P**, write C/C++ and select **C/C++ Edit configurations (GUI)**. Here we’re are going to change some configurations.

In **Compiler Path**, you need to put the path to your compiler. Like we put in the tasks.json and change the IntelliSense mode to gcc_x64. In Include Path, add the path to your SDL2 includes.

![cpp-options](/assets/img/gamedev/cpp-options.png)

Other options that you may change are **C Standard** and **C++ Standard**, If you save these configurations, you are going to have a 
file named **c_cpp_properties.json** in your .vscode folder.

Now, you need to put the **SDL2.dll** file in the **build** folder, to be able to run our SDL2 game.

Remember to use the version of your game

![build-sdldll](/assets/img/gamedev/build-sdldll.png)


## Testing our SDL2 game.

Create a **main.cpp** file in the src folder to test our environment, and write the next code

```cpp
#include <SDL2/SDL.h>
#include <iostream>

int main(int argv, char** args)
{
	SDL_Init(SDL_INIT_EVERYTHING);

	SDL_Window *window = SDL_CreateWindow(
		"Hello SDL", 
		SDL_WINDOWPOS_CENTERED, 
		SDL_WINDOWPOS_CENTERED,
		800, 600, 0);
	
	SDL_Renderer *renderer = SDL_CreateRenderer(window, -1, 0);

	bool isRunning = true;
	SDL_Event event;

	while (isRunning)
	{
		while (SDL_PollEvent(&event))
		{
			switch (event.type)
			{
			case SDL_QUIT:
				isRunning = false;
				break;

			case SDL_KEYDOWN:
				if (event.key.keysym.sym == SDLK_ESCAPE)
				{
					isRunning = false;
				}
			}
		}

		SDL_RenderClear(renderer);
		SDL_SetRenderDrawColor(renderer, 255, 0, 0, 255);

		SDL_RenderPresent(renderer);
	}

	SDL_DestroyRenderer(renderer);
	SDL_DestroyWindow(window);
	SDL_Quit();

	return 0;
}
```

Press **F5**  and cross your fingers. You should watch a red window, is you see it, then you are ready to work with SDL2 and VS Code!

![sdl2-window](/assets/img/gamedev/sdl2-window.png)

This is a simple configuration for start with SDL2, if you want to use SDL_image, you should do the same steps to add the headers and libraries. 
I will continue with SDL2 and check if with this configuration can be have some problems, but for now, you are able to start working with SDL2.