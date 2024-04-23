---
title: OpenGL绘制三角形
date: 2024-04-24 01:20:40
tags:
---

## 渲染流程
先把数据变成二进制，再让GPU解释二进制。把数据变成二进制的过程称为**管线（pipeline）**,用一种编程语言来操控这个管线，叫GLSL。
管线由各种着色器组成，至少需要俩着色器，顶点着色器和片段着色器。

```c
/**
 * 编译着色器
 * @return
 */
unsigned int compile_shader() {
    // glsl源码
    const char *vertexShaderSource = "#version 330 core\n"
                                     "layout (location = 0) in vec3 aPos;\n"
                                     "void main()\n"
                                     "{\n"
                                     "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
                                     "}\0";
    const char *fragmentShaderSource = "#version 330 core\n"
                                       "out vec4 FragColor;\n"
                                       "void main()\n"
                                       "{\n"
                                       "   FragColor = vec4(0.2f, 0.0f, 0.5f, 1.0f);\n"
                                       "}\n\0";
    // 编译shader
    // ------------------------------------
    // 顶点着色器
    unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);
    // check for shader compile errors
    int success;
    char infoLog[512];
    glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
    }
    // 片段着色器
    unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);
    // check for shader compile errors
    glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << infoLog << std::endl;
    }
    // 链接
    unsigned int shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    // check for linking errors
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if (!success) {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
    }
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
    return shaderProgram;
}
```

## 绘制
编译出渲染器之后，得给它数据让它处理。
```c
void draw_my_triangle(GLFWwindow *window, unsigned int shaderProgram) {
    float vertices[] = {
            -0.5f, -0.5f, 0.0f, // left
            0.5f, -0.5f, 0.0f, // right
            0.0f, 0.5f, 0.0f  // top
    };

    unsigned int VBO, VAO;

    //生成VBO，VAO
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);

    //绑定VBO，VAO
    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);

    //写入数据
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    //设置数据属性（如何解释数据）
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *) 0);
    glEnableVertexAttribArray(0);

    //绑定缓冲
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);

    while (!glfwWindowShouldClose(window)) {
        // input
        // -----
        processInput(window);

        // render
        // ------
        glClearColor(0.6f, 0.7f, 0.0f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // 使用着色器
        glUseProgram(shaderProgram);
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

        //
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // optional: 释放资源
    // ------------------------------------------------------------------------
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);
}
```
VBO是顶点缓冲对象，VAO是顶点数组对象。

VBO是一块内存，存着很多顶点数据。VAO里存了一堆顶点属性指针，VBO要想切换属性，就需要绑定不同的VAO。

## 窗口
需要调用glfw库。

```c
int main() {
    // 初始化glfw库
    glfw_init();
//MacOS
#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

    // 创建窗口
    GLFWwindow *window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "绘制三角形", NULL, NULL);
    // 检测窗口
    check_window(window);
    // 设置当前窗口context为主线程
    glfwMakeContextCurrent(window);
    // 设置窗口大小
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
    // 加载opengl函数指针
    load_opengl_fun_ptr();
    // 编译着色器
    auto shaderProgram = compile_shader();
    // 绘制
    draw_my_triangle(window, shaderProgram);
    // 清除glfw资源
    glfwTerminate();
    return 0;
}

```
opengl只能绘制一些简单的东西（图元），点，线，三角形。窗口这种高级的图形是绘制不了的。所以需要第三方库。

## 完整代码：
```c
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <iostream>

// 窗口大小回调函数
void framebuffer_size_callback(GLFWwindow *, int, int);

//按键检测
void processInput(GLFWwindow *);

//glfw初始化
void glfw_init();

//检测窗口
int check_window(GLFWwindow *);

// 加载glad
int load_opengl_fun_ptr();

// 编译着色器
unsigned int compile_shader();

// 绘制
void draw_my_triangle(GLFWwindow *window, unsigned int shaderProgram);

// 定义屏幕宽和高
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

int main() {
    // 初始化glfw库
    glfw_init();
//MacOS
#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

    // 创建窗口
    GLFWwindow *window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "绘制三角形", NULL, NULL);
    // 检测窗口
    check_window(window);
    // 设置当前窗口context为主线程
    glfwMakeContextCurrent(window);
    // 设置窗口大小
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
    // 加载opengl函数指针
    load_opengl_fun_ptr();
    // 编译着色器
    auto shaderProgram = compile_shader();
    // 绘制
    draw_my_triangle(window, shaderProgram);
    // 清除glfw资源
    glfwTerminate();
    return 0;
}

void processInput(GLFWwindow *window) {
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}

void framebuffer_size_callback(GLFWwindow *window, int width, int height) {
    glViewport(0, 0, width, height);
}

void glfw_init() {
    // glfw初始化
    glfwInit();
    // glfw主版本号
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    // glfw次版本号
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    // 使用glfw核心模式
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
}

int check_window(GLFWwindow *window) {
    if (window == NULL) {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    return 0;
}

int load_opengl_fun_ptr() {
    if (!gladLoadGLLoader((GLADloadproc) glfwGetProcAddress)) {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }
    return 0;
}

/**
 * 编译着色器
 * @return
 */
unsigned int compile_shader() {
    // glsl源码
    const char *vertexShaderSource = "#version 330 core\n"
                                     "layout (location = 0) in vec3 aPos;\n"
                                     "void main()\n"
                                     "{\n"
                                     "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
                                     "}\0";
    const char *fragmentShaderSource = "#version 330 core\n"
                                       "out vec4 FragColor;\n"
                                       "void main()\n"
                                       "{\n"
                                       "   FragColor = vec4(0.2f, 0.0f, 0.5f, 1.0f);\n"
                                       "}\n\0";
    // 编译shader
    // ------------------------------------
    // 顶点着色器
    unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER);
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);
    // check for shader compile errors
    int success;
    char infoLog[512];
    glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
    }
    // 片段着色器
    unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);
    // check for shader compile errors
    glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
    if (!success) {
        glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << infoLog << std::endl;
    }
    // 链接
    unsigned int shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    // check for linking errors
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if (!success) {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
    }
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
    return shaderProgram;
}


void draw_my_triangle(GLFWwindow *window, unsigned int shaderProgram) {
    float vertices[] = {
            -0.5f, -0.5f, 0.0f, // left
            0.5f, -0.5f, 0.0f, // right
            0.0f, 0.5f, 0.0f  // top
    };

    unsigned int VBO, VAO;

    //生成VBO，VAO
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);

    //绑定VBO，VAO
    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);

    //写入数据
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    //设置数据属性（如何解释数据）
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *) 0);
    glEnableVertexAttribArray(0);

    //绑定缓冲
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);

    while (!glfwWindowShouldClose(window)) {
        // input
        // -----
        processInput(window);

        // render
        // ------
        glClearColor(0.6f, 0.7f, 0.0f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // 使用着色器
        glUseProgram(shaderProgram);
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, 3);

        //
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // optional: 释放资源
    // ------------------------------------------------------------------------
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);
}
```

## glfw和glad
下载下来，工程目录里新建`include`和`lib`目录，把两个库的头文件放到`include`目录里，`lib`里放`glfw3.dll`,
`libglfw3.a`, `libglfw3dll.a`,把`glad.c`放在项目根目录。

CmakeList.txt的内容如下（Windows）：
```cmake
cmake_minimum_required(VERSION 3.27)
project(example)

set(CMAKE_CXX_STANDARD 17)

# 头文件
include_directories("./include/")

# 库
link_directories("./lib/")

# 编译
add_executable(demo triangle.cpp glad.c)

# 连接
target_link_libraries(demo glfw3 gdi32 opengl32)
```
最后一行链接`gdi32.dll`和`opengl32.dll`，这两个东西在C盘的`system32`文件夹里。

效果图：

<img src="https://images2.imgbox.com/42/20/TybxX3RD_o.png" alt="image host"/>
