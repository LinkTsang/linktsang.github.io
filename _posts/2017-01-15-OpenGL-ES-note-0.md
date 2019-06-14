---
layout: post
title: OpenGL ES 学习笔记(0)
date: 2017-01-15 13:44:55
tags: ["Android", "OpenGL ES"]
---

本文记录了本人学习 Android 上 OpenGL ES 2.0 开发的过程，纯笔记向，非教程向，仅供备忘。

<!-- more -->

OpenGL ES（OpenGL for Embedded Systems）是 OpenGL API 的子集，由C语言写成，Android 靠 JNI 实现了 Java 调用底层的 OpenGL ES。

# 前期知识准备 

- 基本的线性代数 :-)

- Java 
  - Introduction to Java Programming
  - Effective Java
  - Thinking in Java

- Android
  - [Building Your First App](https://developer.android.com/training/basics/firstapp/index.html)
  - [Managing the Activity Lifecycle](https://developer.android.com/training/basics/activity-lifecycle/index.html)

- 参考资料
  - [Displaying Graphics with OpenGL ES](https://developer.android.google.cn/training/graphics/opengl/index.html)
  - [A real Open GL ES 2.0 2D tutorial](http://androidblog.reindustries.com/a-real-open-gl-es-2-0-2d-tutorial-part-1/)
  - OpenGL ES 2 for Android. A Quick-Start Guide.

源码主要参考自 Android 开发文档 以及 《OpenGL ES 2 for Android》。

# 初始化 OpenGL ES 环境

方便起见，默认导入 `import static android.opengl.GLES20.* ` 。
OpenGL ES 渲染视图主要依赖 GLSurfaceView 以及 实现 GLSurfaceView.Renderer 抽象类。
注意根据 Android Activity 的生命周期采取相应的操作。

```Java
public class OpenGLES20Activity extends Activity {

    private GLSurfaceView mGLView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Fullscreen
        getSupportActionBar().hide();
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);

        // Set the content view
        mGLView = new MyGLSurfaceView(this);        
        setContentView(mGLView);
    }

    @Override
    public void onPause() {
        super.onPause();
        mGLView.onPause();
    }

    @Override
    public void onResume() {
        super.onResume();
        mGLView.onResume();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        // 这个一般可不管
    }
```

```Java
class MyGLSurfaceView extends GLSurfaceView {
    private final MyGLRenderer mRenderer;

    @Override
    public boolean onTouchEvent(MotionEvent e) {
        // Deal with touch event
        return true;
    }

    public MyGLSurfaceView(Context context) {
        super(context);

        // Create an OpenGL ES 2.0 context
        setEGLContextClientVersion(2);

        // Set the Renderer for drawing on the GLSurfaceView
        mRenderer = new MyGLRenderer(context);
        setRenderer(mRenderer);

        // Render the view only when there is a change in the drawing data
        setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
    }
}
```

`setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);` 表示只有调用 `requestRender()` 才会进行重绘操作

```Java
public class MyGLRenderer implements GLSurfaceView.Renderer {
    private final Context context;

    public MyGLRenderer(Context context) {
        this.context = context;
    }

    @Override
    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        // Set the background frame color
        glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    }

    @Override
    public void onSurfaceChanged(GL10 unused, int width, int height) {
        glViewport(0, 0, width, height);
    }

    @Override
    public void onDrawFrame(GL10 unused) {
        // Clear the rendering surface.
        glClear(GL_COLOR_BUFFER_BIT);
    }

}
```

这样就基本完成了 OpenGL ES 的初始化。

# 载入着色器

为了绘制图形，我们需要分别定义`Vertex Shader`，`Fragment Shader`，以及`Program`。
> - Vertex Shader - OpenGL ES graphics code for rendering the vertices of a shape.
> - Fragment Shader - OpenGL ES code for rendering the face of a shape with colors or textures.
> - Program - An OpenGL ES object that contains the shaders you want to use for drawing one or more shapes.

![载入流程](/images/OpenGL_GLSL.bmp)
<p class="text-center">载入流程</p>

## 编译
```Java
public static int compileShader(int type, String shaderCode) {
    final int shaderObjectId = glCreateShader(type);

    if (shaderObjectId == 0) {
        if (LoggerConfig.ON) {
            Log.w(TAG, "Could not create new shader.");
        }
        return 0;
    }

    // Pass in the shader source.
    glShaderSource(shaderObjectId, shaderCode);

    // Compile the shader.
    glCompileShader(shaderObjectId);

    // Get the compilation status.
    final int[] compileStatus = new int[1];
    glGetShaderiv(shaderObjectId, GL_COMPILE_STATUS, compileStatus, 0);
    if (LoggerConfig.ON) {
        // Print the shader info log to the Android log output.
        Log.v(TAG, "Results of compiling source:" + "\n" + shaderCode + "\n:"
                + glGetShaderInfoLog(shaderObjectId));
    }

    // Verify the compile status.
    if (compileStatus[0] == 0) {
        // If it failed, delete the shader object.
        glDeleteShader(shaderObjectId);
        if (LoggerConfig.ON) {
            Log.w(TAG, "Compilation of shader failed.");
        }
        return 0;
    }

    // Return the shader object ID.
    return shaderObjectId;
}
```

## 链接
```Java
public static int linkProgram(int vertexShaderId, int fragmentShaderId) {
    final int programObjectId = glCreateProgram();
    if (programObjectId == 0) {
        if (LoggerConfig.ON) {
            Log.w(TAG, "Could not create new program");
        }
        return 0;
    }

    glAttachShader(programObjectId, vertexShaderId);
    glAttachShader(programObjectId, fragmentShaderId);

    // Link the two shaders together into a program.
    glLinkProgram(programObjectId);

    final int[] linkStatus = new int[1];
    glGetProgramiv(programObjectId, GL_LINK_STATUS, linkStatus, 0);

    if (LoggerConfig.ON) {
        // Print the program info log to the Android log output.
        Log.v(TAG, "Results of linking program:\n"
                + glGetProgramInfoLog(programObjectId));
    }

    if (linkStatus[0] == 0) {
        // If it failed, delete the program object.
        glDeleteProgram(programObjectId);
        if (LoggerConfig.ON) {
            Log.w(TAG, "Linking of program failed.");
        }
        return 0;
    }
    return programObjectId;
}
```

## 验证
```Java
public static boolean validateProgram(int programObjectId) {
    glValidateProgram(programObjectId);
    final int[] validateStatus = new int[1];
    glGetProgramiv(programObjectId, GL_VALIDATE_STATUS, validateStatus, 0);
    Log.v(TAG, "Results of validating program: " + validateStatus[0]
            + "\nLog:" + glGetProgramInfoLog(programObjectId));
    return validateStatus[0] != 0;
}
```

## 使用
至此，可以便可以在 `Renderer` 中的 `onSurfaceCreated` 函数中载入着色器。

```Java
int vertexShader = compileShader(GL_VERTEX_SHADER, vertexShaderSource);
int fragmentShader = compileFragmentShader(GL_FRAGMENT_SHADER, fragmentShaderSource);

program = ShaderHelper.linkProgram(vertexShader, fragmentShader);

// Validate the OpenGL program object
if (LoggerConfig.ON) {
    validateProgram(program);
}

glUseProgram(program);
```

待续...

# 坐标

# 绘制顶点
GL_POINTS
GL_LINES
GL_LINE_LOOP
GL_LINE_STRIP
GL_TRIANGLES
GL_TRIANGLE_STRIP
GL_TRIANGLE_FAN
# 添加颜色

# 添加纹理

# 触摸反馈

# 天空盒

