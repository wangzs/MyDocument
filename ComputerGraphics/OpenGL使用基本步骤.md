# OpenGL运行基本步骤
## 1.初始化一个窗口
```c++
// 使用GLFW库来作为窗口显示
#include <GLFW/glfw3.h>

// 窗口初始化流程
glfwInit();
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
glfwWindowHint(GLFW_RESIZABLE, GL_FALSE);

// 设置显示窗口的大小和名字
auto window = glfwCreateWindow(800, 600, "LearnOpenGL", nullptr, nullptr);
if (window == nullptr) {
	std::cout << "Failed to create GLFW window" << std::endl;
	glfwTerminate();
	return -1;
}
glfwMakeContextCurrent(window);


// 设置相应的回调函数
glfwSetKeyCallback(window, [](GLFWwindow* window, int key, int scancode, int action, int mode){
switch (key) {
case GLFW_KEY_ESCAPE:
	glfwSetWindowShouldClose(window, GL_TRUE);
	break;
default:
	break;
}
});


// 每一次的绘制需要调用（双缓冲机制）
glfwSwapBuffers(window);
```

## 2.初始化glew，支持OpenGL的新特性
```c++
// 使用的是glew的静态库
#define GLEW_STATIC
#include <GL/glew.h>

glewExperimental = GL_TRUE;
if (glewInit() != GLEW_OK) {
	std::cout << "Failed to initialize GLEW" << std::endl;
	return -1;
}
```

## 3.编译Shader
```c++
// vertex shader与fragment shader的编译流程类似如下：
GLuint vertex_shader;
vertex_shader = glCreateShader(GL_VERTEX_SHADER);
GLchar* vertex_shader_src = "#version 330 core\n"
	"layout (location = 0) in vec3 position;\n"
	"void main()\n"
	"{\n"
	"gl_Position = vec4(position.x, position.y, position.z, 1.0);\n"
	"}\0";
glShaderSource(vertex_shader, 1, &vertex_shader_src, nullptr);
glCompileShader(vertex_shader);

// 获取编译状态，判断Shader编译是否成功
GLint success;
GLchar infoLog[512];
glGetShaderiv(vertex_shader, GL_COMPILE_STATUS, &success);
if (!success) {
	glGetShaderInfoLog(vertex_shader, 512, NULL, infoLog);
	std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```

## 4.连接编译结果到Shader Program
```c++
GLuint shaderProgram;
shaderProgram = glCreateProgram();
glAttachShader(shaderProgram, vertex_shader);
glAttachShader(shaderProgram, fragment_shader);
glLinkProgram(shaderProgram);

glGetShaderiv(shaderProgram, GL_COMPILE_STATUS, &success);
if (!success) {
	glGetShaderInfoLog(shaderProgram, 512, NULL, infoLog);
	std::cout << "ERROR::SHADER::PROGRAM::LINK_FAILED\n" << infoLog << std::endl;
}
// 链接成功后，删除vertex与fragment shader资源
glDeleteShader(vertex_shader);
glDeleteShader(fragment_shader);



// 每次绘制前，使用Shader Program
glUseProgram(shaderProgram);
```


## 5.基本绘制
** VAO(Vertex Array Objects):一般包含一个或多个VBO，主要是用于存储一个完整的渲染对象（顶点、颜色、纹理信息等）**
** VBO(Vertex Buffer Objects): 显卡中的一块内存buffer，存放vertices信息，如vertices的坐标位置、每个顶点颜色、法线、纹理、索引等 **
```c++
// 包含顶点坐标位置的数组
GLfloat vertices[] = {
		-0.5f, -0.5f, 0.0f, // Left
		0.5f, -0.5f, 0.0f, // Right
		0.0f, 0.5f, 0.0f  // Top
	};

GLuint VAO;
// Allocate and assign a Vertex Array Object to our handle
glGenVertexArrays(1, &VAO);
// Bind the Vertex Array Object first, then bind and set vertex buffer(s) and attribute pointer(s).
glBindVertexArray(VAO);


GLuint VBO;
// Allocate and assign Vertex Buffer Objects to our handle
glGenBuffers(1, &VBO);
// Bind our first VBO as being the active buffer and storing vertex attributes (coordinates)
glBindBuffer(GL_ARRAY_BUFFER, VBO);
// Copy the vertex data from vertices to our buffer(从内存拷贝到显存)
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// Specify that our coordinate data is going into attribute index 0, and contains 3 floats per vertex
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
// Enable attribute index 0 as being used
glEnableVertexAttribArray(0);

// Reset bindings for VBO
glBindBuffer(GL_ARRAY_BUFFER, 0);
// Unbind VAO (it's always a good thing to unbind any buffer/array to prevent strange bugs)
glBindVertexArray(0);


// 绘制上面的元素，因为数据已经在显存中，只要在绘制前调用绑定VAO的函数即可
glBindVertexArray(VAO);
// 开始绘制△
glDrawArrays(GL_TRIANGLES, 0, 3);
// 绘制结束时再unbind
glBindVertexArray(0);

```


## 6.绘制结束清理
```c++
// 清理申请的VAO资源
glDeleteVertexArrays(1, &VAO);
// 清理申请的VBO资源
glDeleteBuffers(1, &VBO);

// 关闭GLFW库，回收GLFW的资源
glfwTerminate();
```






















