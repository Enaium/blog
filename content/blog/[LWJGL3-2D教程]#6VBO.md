---
title: "[LWJGL3 2D教程]#6VBO"
data: 2021-2-28 21:33
categories: lwjgl3
---

创建一个`Model`类

```java
public class Model {
    private int draw_count;
    private int v_id;
    private int t_id;

    public Model(float[] vertices, float[] tex_coords) {
        draw_count = vertices.length / 3;

        v_id = glGenBuffers();
        glBindBuffer(GL_ARRAY_BUFFER, v_id);//绑定
        glBufferData(GL_ARRAY_BUFFER, createFloatBuffer(vertices), GL_STATIC_DRAW);//静态绘制

        t_id = glGenBuffers();
        glBindBuffer(GL_ARRAY_BUFFER, t_id);//绑定
        glBufferData(GL_ARRAY_BUFFER, createFloatBuffer(tex_coords), GL_STATIC_DRAW);//静态绘制


        glBindBuffer(GL_ARRAY_BUFFER, 0);//取消绑定
    }

    public void render() {
        glEnableClientState(GL_VERTEX_ARRAY);//启用顶点数组
        glEnableClientState(GL_TEXTURE_COORD_ARRAY);//启用坐标数组

        glBindBuffer(GL_ARRAY_BUFFER, v_id);//绑定
        glVertexPointer(3, GL_FLOAT, 0, 0);//顶点

        glBindBuffer(GL_ARRAY_BUFFER, t_id);//绑定
        glTexCoordPointer(2, GL_FLOAT, 0, 0);// 坐标

        glDrawArrays(GL_TRIANGLES, 0, draw_count);//绘制数组

        glBindBuffer(GL_ARRAY_BUFFER, 0);//取消绑定

        glDisableClientState(GL_VERTEX_ARRAY);//禁用顶点数组
        glDisableClientState(GL_TEXTURE_COORD_ARRAY);//禁用坐标数组
    }


    //创建缓冲区
    private FloatBuffer createFloatBuffer(float[] data) {
        FloatBuffer buffer = BufferUtils.createFloatBuffer(data.length);

        buffer.put(data);

        buffer.flip();
        return buffer;
    }
}
```

```java
Texture texture = new Texture("/hero.png");

        //2个三角
        float[] vertices = new float[]{
                -0.5f, 0.5f, 0,//左上
                0.5f, 0.5f, 0,//右上
                0.5f, -0.5f, 0,//右下

                0.5f, -0.5f, 0,//右下
                -0.5f, -0.5f, 0,//左下
                -0.5f, 0.5f, 0,//左上
        };

        float[] textures = new float[]{
                0, 0,
                1, 0,
                1, 1,

                1, 1,
                0, 1,
                0, 0
        };

        Model model = new Model(vertices, textures);
```

绘制
```java
texture.bind();
model.render();
```