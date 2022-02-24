# Session 8 - Advanced Topics Two

#### Table of Contents
1. [Instancing](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%208#Instancing)
2. [Blending](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%208#Blending)
3. [Billboards](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%208#Billboards)
4. [Home work](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%208#Home-work)

Welcome to Week 8! 

> In this week, we will learn how to add instances of an object, blending the object and add billboards into the scene.

## Instancing

Say you have a scene where you're drawing a lot of models where most of these models contain the same set of vertex data, 
but with different world transformations. Think of a scene filled with grass leaves: each grass leave is a small model that consists of only a few triangles. 
You'll probably want to draw quite a few of them and your scene may end up with thousands or maybe tens of thousands of grass leaves 
that you need to render each frame. Because each leaf is only a few triangles, the leaf is rendered almost instantly. 
However, the thousands of render calls you'll have to make will drastically reduce performance.

If we were to actually render such a large amount of objects it will look a bit like this in code:

```C++
for(unsigned int i = 0; i < amount_of_models_to_draw; i++)
{
    DoSomePreparations(); // bind VAO, bind textures, set uniforms etc.
    glDrawArrays(GL_TRIANGLES, 0, amount_of_vertices);
}
```

When drawing many instances of your model like this you'll quickly reach a performance bottleneck because of the many draw calls. 
Compared to rendering the actual vertices, telling the GPU to render your vertex data with functions like glDrawArrays or 
glDrawElements eats up quite some performance since OpenGL must make necessary preparations before it can draw your vertex data 
(like telling the GPU which buffer to read data from, where to find vertex attributes and all this over the relatively slow CPU to GPU bus).
 So even though rendering your vertices is super fast, giving your GPU the commands to render them isn't.
 
It would be much more convenient if we could send data over to the GPU once, and then tell OpenGL to draw multiple objects using this data with a single drawing call. 
That is instancing.

To render using instancing many times,

* glDrawArrays -> glDrawArraysInstanced
* glDrawElements -> glDrawElementsInstanced 

These instanced versions of the classic rendering functions take an extra parameter called the instance count
 that sets the number of instances we want to render. We sent all the required data to the GPU once, 
 and then tell the GPU how it should draw all these instances with a single call. 
 The GPU then renders all these instances without having to continually communicate with the CPU.

### Implementation

* Download the base project (SphereInstance.zip). Open Sphere.cpp

We just need to modify the drawing function to draw 4 spheres instead one.

Replace

```C++
glDrawElements(GL_TRIANGLE_STRIP, triCount, GL_UNSIGNED_INT, sphereIndices);
```

with 

```C++
glDrawElementsInstanced(GL_TRIANGLE_STRIP, triCount, GL_UNSIGNED_INT, sphereIndices, 4);
```


* Modify Vertex shader codes so that we can see 4 sphere.

Now, all 4 sphere is in the same position. So, you do not see any difference at all. Modify position  

Replace
```C++
   if (object == SPHERE)
   {
      coords = sphereCoords;
      normalExport = sphereNormals;
   }
```

with

```C++
   if (object == SPHERE)
   {
      coords = vec4(sphereCoords.x+(gl_InstanceID*12 - 18),sphereCoords.y,sphereCoords.z,sphereCoords.w);
      normalExport = sphereNormals;
   }
```

Here gl_InstanceID is a built-in variable inside shader which represent the id of instance object. So, it is range from 0 to 3. 
We, move the x coordinate by (gl_InstanceID*12 - 18) for each instance.


* Finally it should look like this

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/Instance.JPG)

### Assign different color to individual instance

All can be done in shaders.
First add InstanceID variable into the vertex shader.

In vertex shader, add 
```C++
flat out int InstanceID; //output InstanceID variable to fragment shader
```

Then change codes so that InstanceID is assigned by  gl_InstanceID
```C++
   if (object == SPHERE)
   {
      InstanceID = gl_InstanceID;
      coords = vec4(sphereCoords.x+(gl_InstanceID*12 - 18),sphereCoords.y,sphereCoords.z,sphereCoords.w);
      normalExport = sphereNormals;
   }
```

In fragment shader, add 
```C++
flat in int InstanceID; 
```

Then assign difference color to the sphere according to the InstanceID
```C++
	if (object == SPHERE) {
   		normal = normalize(normalExport);
		lightDirection = normalize(vec3(light0.coords));
	    if(InstanceID==0)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * vec4(1.0,0.0,0.0,1.0); //red color
		  colorsOut =  vec4(vec3(min(fAndBDif, vec4(1.0))), 1.0);
		}
		if(InstanceID==1)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * vec4(0.0,1.0,0.0,1.0); //green color
		  colorsOut =  vec4(vec3(min(fAndBDif, vec4(1.0))), 1.0);
		}
		if(InstanceID==2)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * vec4(0.0,0.0,1.0,1.0); //blue color 
		  colorsOut =  vec4(vec3(min(fAndBDif, vec4(1.0))), 1.0);
		}
		if(InstanceID==3)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * (light0.difCols * sphereFandB.difRefl); //original color
		  colorsOut =  vec4(vec3(min(fAndBDif, vec4(1.0))), 1.0);
		}

   }
```

* Finally it should look like this

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/InstanceColor.JPG)

### Assign different texture to individual instance

This is the advanced level and it is optional.

Use the same idea to assign different texture to individual instance. Please refer week 4 tutorial for adding texture coordinates into your sphere class.
Specially, you need to re-dsign your setup function in sphere class

```C++
void Sphere::Setup()
{
	int verCount = 121;
	int triCount = 660;
	glBindVertexArray(VAO);
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(VertexWithAll) * verCount, sphereVerticesNor, GL_STATIC_DRAW);
	glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, IBO);
	glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(unsigned int) * triCount, sphereIndices, GL_STATIC_DRAW);
	glVertexAttribPointer(2, 4, GL_FLOAT, GL_FALSE, sizeof(sphereVerticesNor[0]), 0);
	glEnableVertexAttribArray(2);
	glVertexAttribPointer(3, 3, GL_FLOAT, GL_FALSE, sizeof(sphereVerticesNor[0]), (GLvoid*)offsetof(VertexWithAll, normals));
	glEnableVertexAttribArray(3);
	glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, sizeof(sphereVerticesNor[0]), (GLvoid*)offsetof(VertexWithAll, texCoords));
	glEnableVertexAttribArray(1);
}
```

Make sure sphere has coordinates, normals and texture coordinates.

Then add additional codes in vetex shader to export texture coordinates to fragment shader. In this case, we re-use TexCoords layout to receive data.

```C++
   if (object == SPHERE)
   {
      InstanceID = gl_InstanceID;
      coords = vec4(sphereCoords.x+(gl_InstanceID*12 - 18),sphereCoords.y,sphereCoords.z,sphereCoords.w);
      normalExport = sphereNormals;
      texCoordsExport = TexCoords;
   }
```

Then add texture importing codes in main program. There are two additional images (sphere.jpg and earth.jpg) in the texture folder of the base project. 
Do not forget to enlarge texture[2] to texture[4]. 

```C++
  // Load the images.
   std::string TexNames[] = {
		"Textures/grass.bmp",
		"Textures/sky.bmp",
		"Textures/sphere.jpg",
		"Textures/earth.jpg",
   };

   // Create texture ids.
   glGenTextures(4, texture);
```

```C++
   glActiveTexture(GL_TEXTURE2);
   glBindTexture(GL_TEXTURE_2D, texture[2]);

   data = SOIL_load_image(TexNames[2].c_str(), &width, &height, 0, SOIL_LOAD_RGBA);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
   SOIL_free_image_data(data);

   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
   glGenerateMipmap(GL_TEXTURE_2D);
   sphereTexLoc = glGetUniformLocation(programId, "sphereTex");
   glUniform1i(sphereTexLoc, 2); //send texture to shader
   
   glActiveTexture(GL_TEXTURE3);
   glBindTexture(GL_TEXTURE_2D, texture[3]);

   data = SOIL_load_image(TexNames[3].c_str(), &width, &height, 0, SOIL_LOAD_RGBA);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
   SOIL_free_image_data(data);

   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
   glGenerateMipmap(GL_TEXTURE_2D);
   earthTexLoc = glGetUniformLocation(programId, "earthTex");
   glUniform1i(earthTexLoc, 3); //send texture to shader
```

Finally, add codes into fragment shader to handling texture map for spheres.

```C++
uniform sampler2D sphereTex;
uniform sampler2D earthTex;
vec4 sphereTexColor,earthTexColor;
```

Inside the main function of fragment shader
```C++
   sphereTexColor = texture(sphereTex, texCoordsExport);
   earthTexColor = texture(earthTex, texCoordsExport);
   
   if (object == SPHERE) {
   		normal = normalize(normalExport);
		lightDirection = normalize(vec3(light0.coords));
	    if(InstanceID==0)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * vec4(1.0,0.0,0.0,1.0); //red color
		  colorsOut =  vec4(vec3(min(fAndBDif, vec4(1.0))), 1.0)*sphereTexColor;
		}
		if(InstanceID==1)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * vec4(0.0,1.0,0.0,1.0); //green color
		  colorsOut =  vec4(vec3(min(fAndBDif, vec4(1.0))), 1.0)*sphereTexColor;
		}
		if(InstanceID==2)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * vec4(0.0,0.0,1.0,1.0); //blue color 
		  colorsOut = earthTexColor;
		}
		if(InstanceID==3)
	    {
		  fAndBDif = max(dot(normal, lightDirection), 0.0f) * (light0.difCols * sphereFandB.difRefl); //original color
		  colorsOut =  vec4(vec3(min(fAndBDif, vec4(1.0))), 1.0)*sphereTexColor;
		}
   }
```

The final results look like
![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/InstanceTex.JPG)

### Assign different position to individual instance

In the previous example, we modify coordinates of spheres in vertex shader to move all instances of sphere into different position.
In this section, we will use the position data from sphere class to move the ball around the scene so that you can apply physics forces to individual ball.
To do that, first we create position array in the sphere class (sphere.h)
Add following codes in sphere.h (assume we have 16 balls)

```C++
public:
	float PosX[16]; //X of ball position
	float PosY[16]; //Y of ball position
```

Then initialize the ball position in setup() function in main program
```C++
   for (int i = 0; i < 16; i++)
   {
	   testSphere.PosX[i] = i * 12.0 - 18.0;
   }
```

Then in the drawscene function, send position data to the vertex shader. glUniform1fv can send an array of float to the vertex shader.
```C++
   // Draw sphere
   glUniform1fv(glGetUniformLocation(programId, "PosX"), 16, testSphere.PosX);
   testSphere.updateModelMatrix(modelViewMatLoc, d);
   glUniform1ui(objectLoc, SPHERE);  //if (object == SPHERE)
   testSphere.Draw();
```

In vertex shader, we define the position data first.
```C++
uniform float PosX[16];
```

Then in the main function (vertex shader). Use PosX to move each instance into diferent position
```C++
if (object == SPHERE)
   {
      InstanceID = gl_InstanceID;
      //coords = vec4(sphereCoords.x+(gl_InstanceID*12 - 18),sphereCoords.y,sphereCoords.z,sphereCoords.w);
      coords = vec4(sphereCoords.x+PosX[gl_InstanceID],sphereCoords.y,sphereCoords.z,sphereCoords.w);
      normalExport = sphereNormals;
      texCoordsExport = TexCoords;
   }
```


### Add instancing into your own project

* Select a decorated object in your scene such as a rock or ball object. Modify the drawing codes.

* In the vertex shader, change their position. Ideally, along certain pattern such as line, circle and etc.


## Blending

Blending in OpenGL is commonly known as the technique to implement transparency within objects. 
Transparency is all about objects (or parts of them) not having a solid color, 
but having a combination of colors from the object itself and any other object behind it with varying intensity.

The textures we've used so far all consisted of 3 color components: red, green and blue, 
but some textures also have an embedded alpha channel that contains an alpha value per texel (such as PNG image file).

We are going to use a transparent image (trees.png) as the texture map and use it as decorated objects.

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/trees.png)


* Step 1: Add a rectangle plane into your secne 

Use the project in instancing (set the number of instance to 1), open CreateSphere.cpp. Add Both VAO and VBO for new object: Tree.

```C++
static enum object {FIELD, SKY,SPHERE, TREE}; // VAO ids.
static enum buffer {FIELD_VERTICES, SKY_VERTICES,SPHERE_VERTICES, SPHERE_INDICES, TREE_VERTICES}; // VBO ids.
```

Add Tree rectangle vertices.

```C++
static Vertex treeVertices[4] =
{
	{vec4(20.0, 0.0, -20.0, 1.0), vec2(1.0, 0.0)},
	{vec4(20.0, 40.0, -20.0, 1.0), vec2(1.0, 1.0)},
	{vec4(-20.0, 0.0, -20.0, 1.0), vec2(0.0, 0.0)},
	{vec4(-20.0, 40.0, -20.0, 1.0), vec2(0.0, 1.0)}
};
```

* Increase VAO and VBO array size

```C++
   buffer[5], ///add one more object
   vao[4], ///add one more object
```

* Create ID for VAO and VBO

```C++
   //Create VAO and VBO
   glGenVertexArrays(1, &vao[TREE]);
   glGenBuffers(1, &buffer[TREE_VERTICES]);
```

* Binding VAO and VBO

```C++
   glBindVertexArray(vao[TREE]);
   glBindBuffer(GL_ARRAY_BUFFER, buffer[TREE_VERTICES]);
   glBufferData(GL_ARRAY_BUFFER, sizeof(treeVertices), treeVertices, GL_STATIC_DRAW);
   glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, sizeof(treeVertices[0]), 0);  //layout(location=2) in vec4 skyCoords;
   glEnableVertexAttribArray(0);
   glVertexAttribPointer(1, 2, GL_FLOAT, GL_FALSE, sizeof(treeVertices[0]), (void*)(sizeof(treeVertices[0].coords))); //layout(location=3) in vec2 skyTexCoords;
   glEnableVertexAttribArray(1);
```

* Draw tree

```C++
   // Draw Tree.
   glUniform1ui(objectLoc, TREE);  //if (object == TREE)
   glBindVertexArray(vao[TREE]);
   glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
```

* Add codes for vertex shader

```C++
#define TREE 3

   if (object == TREE)
   {
      coords = Coords;
      texCoordsExport = TexCoords;
   }
```

* Add testing codes for fragment shader

For now, we just use sky texture map for testing

```C++
#define TREE 3

 if (object == TREE) colorsOut = skyTexColor;
```

* compile and run

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/StepOne.JPG)

### Add transparent TREE texture map

* Add texture location

```C++
TreeTexLoc,
```

* Add texture filename

```C++
"Textures/trees.png",
```

* Increase array size

```C++
glGenTextures(3, texture);
```

* Texture initialization codes

```C++
   glActiveTexture(GL_TEXTURE2);
   glBindTexture(GL_TEXTURE_2D, texture[2]);

   data = SOIL_load_image(TexNames[2].c_str(), &width, &height, 0, SOIL_LOAD_RGBA);
   glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
   SOIL_free_image_data(data);

   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
   glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
   glGenerateMipmap(GL_TEXTURE_2D);
   TreeTexLoc = glGetUniformLocation(programId, "treeTex");
   glUniform1i(TreeTexLoc, 2); //send texture to shader
```

* Add codes into fragment shader

```C++
uniform sampler2D treeTex;

vec4 fieldTexColor, skyTexColor, treeTexColor;

treeTexColor =  texture(treeTex, texCoordsExport);

 if (object == TREE) colorsOut = treeTexColor;
```

* Compile and run

You will notice there are two problem. One is not transparent. The other one is upside down.

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/StepTwo.JPG)

* Fix transparency

This happens because OpenGL by default does not know what to do with alpha values, nor when to discard them. 
We have to manually do this ourselves. Luckily this is quite easy thanks to the use of shaders. 
GLSL gives us the discard command that (once called) ensures the fragment will not be further processed and thus not end up into the color buffer. 
Thanks to this command we can check whether a fragment has an alpha value below a certain threshold and 
if so, discard the fragment as if it had never been processed:

```C++
   if (object == TREE) 
   {
      if(treeTexColor.a < 0.2)         discard;
      colorsOut = treeTexColor;
   }
```

* Fix upside down. Add codes into vertex shader to flip y axis

```C++
   if (object == TREE)
   {
      coords = Coords;
      coords.y = 40.0 - Coords.y;
      texCoordsExport = TexCoords;
   }
```

* Compile and run

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/StepThree.JPG)

## Billboards

Billboarding is a popular technique used in 3D graphics programming. Billboarding allows an object (usually a quad) to always face a given camera.  

There are two simple techniques (spherical and cylindrical) to do billboarding in the vertex shader.

Both techniques are similar and consist in tweaking the rotational part of the ModelView matrix. 
This matrix is the result of the multiplication of the View matrix (camera) and the Model matrix (object):

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/modelview.jpg)

The rotation matrix included in the ModelView matrix is a 3×3 matrix (green dotted lines). 
This rotation matrix (three columns) can be extracted from the ModelView matrix like this: 

```C++
// Column 0:
ModelView[0][0]  // 0
ModelView[0][1]  // 1
ModelView[0][2]  // 2

// Column 1:
ModelView[1][0]  // 4
ModelView[1][1]  // 5
ModelView[1][2]  // 6

// Column 2:
ModelView[2][0]  // 8
ModelView[2][1]  // 9
ModelView[2][2]  // 10
```

Spherical billboarding makes the object to always face the camera no matter the position of the camera on an imaginary sphere. 
Cylindrical billboarding makes the object to face the camera only when the camera looks at the right or at the left.

To do spherical billboarding, just remove all rotations by setting the identity matrix:

```C++
// Column 0:
ModelView[0][0] = 1;
ModelView[0][1] = 0;
ModelView[0][2] = 0;

// Column 1:
ModelView[1][0] = 0;
ModelView[1][1] = 1;
ModelView[1][2] = 0;

// Column 2:
ModelView[2][0] = 0;
ModelView[2][1] = 0;
ModelView[2][2] = 1;
```

If we don’t touch the second column (column 1), we get the cylindrical billboarding:

```C++
// Column 0:
ModelView[0][0] = 1;
ModelView[0][1] = 0;
ModelView[0][2] = 0;

// Column 2:
ModelView[2][0] = 0;
ModelView[2][1] = 0;
ModelView[2][2] = 1;
```

* Example of cylindrical billboarding implementation in vertex shader

Just replace Final part vertex shader codes 

```C++
   gl_Position = projMat * modelViewMat * coords;
```

with

```C++
   if (object == TREE)
   {
         mat4 modelView = modelViewMat;
          // First colunm.
          modelView[0][0] = 1.0; 
          modelView[0][1] = 0.0; 
          modelView[0][2] = 0.0; 

          // Thrid colunm.
          modelView[2][0] = 0.0; 
          modelView[2][1] = 0.0; 
          modelView[2][2] = 1.0; 

        gl_Position = projMat * modelView * coords;
   }
   else
      gl_Position = projMat * modelViewMat * coords;
```

* Finally, it looks like this (in different view angle) (Use left and right arrow keys to adjust views)

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%208/Readme%20Pictures/StepFour.JPG)

## Home work

Continue to work on your own project 

* Add some instances of decorated objects into the scene.

* Add blending textures such as trees into the scene

* Add billboardings into the scene (optional) 




