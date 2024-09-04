---
layout: post
title:  OpenGL Setup in macOS
date:   2019-12-10 15:00:00 -0600
description: OpenGL Setup in macOS
categories: GameDev
---

We have [configured SFML in macOS](/sfml-2-5-1-setup-on-macos-with-clion/), now we’re going to configure OpenGL with macOS and Xcode.

First, you need to install GLFW

```
brew install glfw
```

Then, you need to download GLAD to get the OpenGL headers: Use **gl version 4.1** if you want to be compatible with macOS 
and in Profile select **Core**. Click in **Generate**. Download the **glad.zip** and unzip in your Download folder.

[https://glad.dav1d.de](https://glad.dav1d.de)

Once you are downloaded and unzipped the folder, put the **glad** and **KHR** folders in the **/usr/local/include** directory of your mac.

## Creating the Xcode Project

After installed GLFW and downloading the glad files, is time to create the XCode project.

Create a new Project and select **macOS**, then **Command Line Tool**, finally click on Next

![create-command-line-took](/assets/img/gamedev/create-command-line-took.png)

Then, put a name to your project and select C++ as language, next and save the project

![create-project-name-xcode-opengl](/assets/img/gamedev/create-project-name-xcode-opengl.png)

In the zip that you downloaded, you will have a glad.c file, add that file to your project

![glad-to-xcde](/assets/img/gamedev/glad-to-xcde.png)

We need to configure Xcode to find the libraries that we added, so in your Xcode, 
select the Project Target and then **Build Settings** and in **Header Search Paths** add **/usr/local/include**

![header-search-paths](/assets/img/gamedev/header-search-paths.png)

Now, in **Framework and Libraries** you need to add the **libglfw** library, so clic on + symbol and add the 
library that you will find in **/usr/local/Cellar/glfw/3.3/lib**

![lib-cellar-glfw](/assets/img/gamedev/lib-cellar-glfw.png)

Now, in the **main.cpp** file, copy the follow code to test OpenGL

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

void framebuffer_size_callback(GLFWwindow* window, int width, int height);

int main()
{
    GLFWwindow* window;

    // Initialize the library
    if(!glfwInit())
        return -1;

    // Define version and compatibility settings
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 1);
    glfwWindowHint(GLFW_OPENGL_PROFILE,GLFW_OPENGL_CORE_PROFILE);
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);

    // Create a windowed mode window and its OpenGL context
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }

    // Mathe the window's context current
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    // Initialize the OpenGL API with GLAD
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }

    // Loop until the user closes the window
    while(!glfwWindowShouldClose(window))
    {
        // Render here!
        glClear(GL_COLOR_BUFFER_BIT);

        // Swap front and back buffers
        glfwSwapBuffers(window);

        // Poll for and process events
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}

// glfw: whenever the window size changed (by OS or user resize) this callback function executes
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    // make sure the viewport matches the new window dimensions; note that width
    // and height will be significantly larger than specified on retina displays
    glViewport(0, 0, width, height);
}
```

If you are in Catalina and run the project, your are going to see an error, that is because the library has 
not the permissions to run in your mac, so you will need to sign it to run.

In the terminal copy and paste the command

```
codesign -f -s "your@email.com" /usr/local/Cellar/glfw/3.3/lib/libglfw.3.3.dylib
```

Edit: Thanks to a comment of **Maxwell Omdal**, if you can’t sign with you email, you can do it with your name:

```
codesign -s -f "Developer ID Application: Name /usr/local/Cellar/glfw/3.3/lib/libglfw.3.3.dylib"
```

Now, run the project again and you should see the window 🙂

![Screenshot-2019-12-10-11.55.10](/assets/img/gamedev/Screenshot-2019-12-10-11.55.10.png)

Now you will be able to play with OpenGL 🙂




