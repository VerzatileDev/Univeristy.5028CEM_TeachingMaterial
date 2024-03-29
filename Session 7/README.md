# Session 7 - Advanced Topics One

#### Table of Contents
1. [Skybox](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%207#Skybox)
2. [Integrate into your project](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%207#Integrate-into-your-project)
3. [Look around camera](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%207#Look-around-camera)
5. [Add a Room](https://github.coventry.ac.uk/ac7020/5028CEM_TeachingMaterial/tree/master/Session%207#add-a-room)
6. [Your own project](https://github.coventry.ac.uk/ac7020/5028CEM_TeachingMaterial/tree/master/Session%207#your-own-project)

Welcome to Week 7! 

> In this week, we will learn how to create a Skybox and create a look-around camera.


## Skybox

Since a skybox is by itself just a cubemap, loading a skybox isn't too different from what we've seen at the start of this chapter. To load the skybox we're going to use the following function that accepts a vector of 6 texture locations:

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%207/Readme%20Pictures/Skybox.png)

To implement a skybox is quite simple. We simply unwrap a cube into its UV Map. Apply a texture to each face of the cube and render the cube in the middle of the scene.

### Loading a skybox

* Download the base project. Always to Compile option to "x64".  Open Skybox.cpp

* Since a skybox is by itself just a cubemap, loading a skybox is to accept a vector of 6 texture locations.

A cubemap is a texture like any other texture, so to create one we generate a texture and bind it to the proper texture target 
before we do any further texture operations. This time binding it to GL_TEXTURE_CUBE_MAP:

Because a cubemap contains 6 textures, one for each face, we have to call glTexImage2D six times with their parameters set. 
We have to set the texture target parameter to match a specific face of the cubemap, telling OpenGL which side of the cubemap we're creating a texture for. 
This means we have to call glTexImage2D once for each face of the cubemap.

Since we have 6 faces OpenGL gives us 6 special texture targets for targeting a face of the cubemap:

Texture target	Orientation
* GL_TEXTURE_CUBE_MAP_POSITIVE_X	Right
* GL_TEXTURE_CUBE_MAP_NEGATIVE_X	Left
* GL_TEXTURE_CUBE_MAP_POSITIVE_Y	Top
* GL_TEXTURE_CUBE_MAP_NEGATIVE_Y	Bottom
* GL_TEXTURE_CUBE_MAP_POSITIVE_Z	Back
* GL_TEXTURE_CUBE_MAP_NEGATIVE_Z	Front

Like many of OpenGL's enums, their behind-the-scenes int value is linearly incremented, 
so if we were to have an array or vector of texture locations we could loop over them by starting with GL_TEXTURE_CUBE_MAP_POSITIVE_X and
 incrementing the enum by 1 each iteration, effectively looping through all the texture targets (following source codes are for demo only, no need to be added into the project):

```C++
for (unsigned int i = 0; i < 6; i++)
 glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data); 
```

Because a cubemap is a texture like any other texture, we will also specify its wrapping and filtering methods (following source codes are for demo only, no need to be added into the project)

```C++
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE); 
```
Don't be scared by the GL_TEXTURE_WRAP_R, this simply sets the wrapping method for the texture's R coordinate 
which corresponds to the texture's 3rd dimension (like z for positions). We set the wrapping method to GL_CLAMP_TO_EDGE 
since texture coordinates that are exactly between two faces may not hit an exact face (due to some hardware limitations) so 
by using GL_CLAMP_TO_EDGE OpenGL always returns their edge values whenever we sample between faces.

* Add following codes to Skybox::InitialiseCubeMap().  

```C++
void Skybox::InitialiseCubeMap()
{
	std::string skyboxTextures[] = 
	{	
		"Textures/SkyboxRight.jpg",
		"Textures/SkyboxLeft.jpg",
		"Textures/SkyboxTop.jpg",
		"Textures/SkyboxBottom.jpg",
		"Textures/SkyboxFront.jpg",
		"Textures/SkyboxBack.jpg"
	};
	glUseProgram(programId);
	glGenTextures(1, &textureID);
	glActiveTexture(GL_TEXTURE0 + textureID);
	myTextureIDs[0] = textureID;
	glBindTexture(GL_TEXTURE_CUBE_MAP, textureID);

	int width, height;
	for (unsigned int i = 0; i < 6; i++)
	{
		unsigned char *data = SOIL_load_image(skyboxTextures[i].c_str(), &width, &height, 0, SOIL_LOAD_RGBA);
		glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
		SOIL_free_image_data(data);
	}
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);
	glGenerateMipmap(GL_TEXTURE_2D);

	++textureID;
}
```


### Render Skybox

* Rendering the skybox is easy now that we have a cubemap texture, 
we simply bind the cubemap texture and the skybox sampler is automatically filled with the skybox cubemap.
 To draw the skybox we're going to draw it as the first object in the scene and disable depth writing. 
 This way the skybox will always be drawn at the background of all the other objects 
 since the unit cube is most likely smaller than the rest of the scene.


```C++
void Skybox::Draw()
{
	glDepthFunc(GL_LEQUAL);  // change depth function so depth test passes when values are equal to depth buffer's content
	glUseProgram(programId);
	// skybox cube
	glBindVertexArray(skyboxVAO);
	int pos =glGetUniformLocation(programId, "skyboxTexture");
	glUniform1i(glGetUniformLocation(programId, "skyboxTexture"), myTextureIDs[0]);
	glDrawArrays(GL_TRIANGLES, 0, 36);
	glBindVertexArray(0);
	glDepthFunc(GL_LESS); // set depth function back to default
}
```

* Finally it should look like this (Always to Compile option to "x64" )

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%207/Readme%20Pictures/SkyScreenshot.JPG)


## Integrate into your project

### This is an optional task. If you are developing a pool game, you might use the skybox as the room (please change the texture images).
The Skybox can not directly into your project (not just a copy-paste job!). The Skybox example project uses a stand-alone shader architecture.

There are two ways you can make the skybox working in your project. One is convert your model class (or any other drawing classes) into stand-alone shader architecture.
This requires some advanced programming skills which deeply understand the modern OpenGL drawing pipelines

The relative easy way to emerge skybox shader codes with your existing shader codes and also modify the skybox class.
Here is some instructions for how to integrate skybox into your project. Of course, they will not solve every problem you are facing.
However they will provide you a good starting point.

### Step one: Modify Skybox class  
Remove CreateShader function inside Skybox class. You have to remove both CreateShader defintion in header file and remove
implementation in cpp file. We are going to merge the skybox shader codes with main shaders.

Remove variables definition. We only need VAO and VBO. so, only variables left are

```C++
private:
	unsigned int skyboxVAO,	skyboxVBO;
```

Do rememeber to delete related codes inside Skybox class.

Change InitialiseSkybox definition and implementation. The function takes VAO and VBO from the gameEngine or main.

```C++
void InitialiseSkybox(unsigned int vao, unsigned int vbo);
```

The implementation.

```C++
void Skybox::InitialiseSkybox(unsigned int vao, unsigned int vbo)
{
	float skyboxVertices[] =
	{
		-300.0f,  300.0f, -300.0f,
		-300.0f, -300.0f, -300.0f,
		 300.0f, -300.0f, -300.0f,
		 300.0f, -300.0f, -300.0f,
		 300.0f,  300.0f, -300.0f,
		-300.0f,  300.0f, -300.0f,

		-300.0f, -300.0f,  300.0f,
		-300.0f, -300.0f, -300.0f,
		-300.0f,  300.0f, -300.0f,
		-300.0f,  300.0f, -300.0f,
		-300.0f,  300.0f,  300.0f,
		-300.0f, -300.0f,  300.0f,

		 300.0f, -300.0f, -300.0f,
		 300.0f, -300.0f,  300.0f,
		 300.0f,  300.0f,  300.0f,
		 300.0f,  300.0f,  300.0f,
		 300.0f,  300.0f, -300.0f,
		 300.0f, -300.0f, -300.0f,

		-300.0f, -300.0f,  300.0f,
		-300.0f,  300.0f,  300.0f,
		 300.0f,  300.0f,  300.0f,
		 300.0f,  300.0f,  300.0f,
		 300.0f, -300.0f,  300.0f,
		-300.0f, -300.0f,  300.0f,

		-300.0f,  300.0f, -300.0f,
		 300.0f,  300.0f, -300.0f,
		 300.0f,  300.0f,  300.0f,
		 300.0f,  300.0f,  300.0f,
		-300.0f,  300.0f,  300.0f,
		-300.0f,  300.0f, -300.0f,

		-300.0f, -300.0f, -300.0f,
		-300.0f, -300.0f,  300.0f,
		 300.0f, -300.0f, -300.0f,
		 300.0f, -300.0f, -300.0f,
		-300.0f, -300.0f,  300.0f,
		 300.0f, -300.0f,  300.0f
	};
	skyboxVAO = vao;
	skyboxVBO = vbo;
	glBindVertexArray(skyboxVAO);
	glBindBuffer(GL_ARRAY_BUFFER, skyboxVBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(skyboxVertices), &skyboxVertices, GL_STATIC_DRAW);
	glEnableVertexAttribArray(0);
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
}
```

Change InitialiseSkybox definition and implementation.

```C++
void InitialiseCubeMap(unsigned int programId, unsigned int textureID);
```

```C++
void Skybox::InitialiseCubeMap(unsigned int programId, unsigned int textureID)
{
	std::string skyboxTextures[] =
	{
		"Textures/SkyboxRight.jpg",
		"Textures/SkyboxLeft.jpg",
		"Textures/SkyboxTop.jpg",
		"Textures/SkyboxBottom.jpg",
		"Textures/SkyboxFront.jpg",
		"Textures/SkyboxBack.jpg"
	};

	glActiveTexture(GL_TEXTURE0 + textureID-1);
	glBindTexture(GL_TEXTURE_CUBE_MAP, textureID);

	int width, height;
	for (unsigned int i = 0; i < 6; i++)
	{
		unsigned char* data = SOIL_load_image(skyboxTextures[i].c_str(), &width, &height, 0, SOIL_LOAD_RGBA);
		glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
		SOIL_free_image_data(data);
	}
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
	glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);
	glGenerateMipmap(GL_TEXTURE_2D);

	unsigned int skyTexLoc = glGetUniformLocation(programId, "skyboxTexture");
	glUniform1i(skyTexLoc, textureID-1); //send texture to shader
}
```

Change Draw definition and implementation.

```C++
void Draw(unsigned int programId);
```

```C++
void Skybox::Draw(unsigned int programId)
{
	glDepthFunc(GL_LEQUAL);  // change depth function so depth test passes when values are equal to depth buffer's content
	glBindVertexArray(skyboxVAO);
	glDrawArrays(GL_TRIANGLES, 0, 36);
	glBindVertexArray(0);
	glDepthFunc(GL_LESS); // set depth function back to default
}
```

Change SetViewMatrix definition and implementation.

```C++
void SetViewMatrix(unsigned int modelViewMatLoc, glm::mat4 modelViewMat);
```

```C++
void Skybox::SetViewMatrix(unsigned int modelViewMatLoc, glm::mat4 modelViewMat)
{
	glUniformMatrix4fv(modelViewMatLoc, 1, GL_FALSE, value_ptr(modelViewMat));
}
```

Delete rest of functions inside Skybox class.

### Step two: Merge shader codes

For vertex shader, first define SKYBOX object and add import and output data. additional codes 

```C++
#define SKYBOX 1

layout(location=0) in vec3 skyCoords;

out vec3 SkytexCoordsExport;
```

Please not vec3 SkytexCoordsExport is the texture coordinates for Skybox (it is vec3 not vec2)

Add coordinate codes
```C++
    if (object == SKYBOX)
    {
        SkytexCoordsExport = skyCoords;
        coords = vec4(skyCoords, 1.0);
    }
```

For Fragment shader, first define SKYBOX object and add import data. additional codes 

```C++
#define SKYBOX 1

in vec3 SkytexCoordsExport;

uniform samplerCube skyboxTexture;
```

Finally, color calculation codes.

```C++
    if (object == SKYBOX) {
    colorsOut = texture(skyboxTexture, SkytexCoordsExport);
    }
```

### Step three: Add codes into your game engine or main.

Make sure, you increase the size of your both VAO and VBO to accommodate Skybox.
Also, increase the size texture id array and increase number in glGenTextures(3, texture); 

Make your viewing frustum is big enough to accommodate the Skybox. So, place your project matrix defintion with
 
```C++
   projMat = perspective(radians(60.0), 1.0, 0.1, 1000.0);
```

Replace skybox initialization codes in setup function wth

```C++
   glGenVertexArrays(1, &vao[SKYBOX]);
   glGenBuffers(1, &buffer[SKYBOX_VERTICES]);
   skybox.InitialiseSkybox(vao[SKYBOX], buffer[SKYBOX_VERTICES]);
   skybox.InitialiseCubeMap(programId, texture[3]);
```

The exact index number for texture[3] should be adjusted according to your project.

In draw function, please place skybox drawing codes right after modelview matrix codes. For example,

 ```C++
   // Calculate and update modelview matrix.
   modelViewMat = mat4(1.0);
   modelViewMat = lookAt(vec3(0.0, 10.0, 15.0), vec3(0.0 + d, 10.0, 0.0), vec3(0.0, 1.0, 0.0));
   glUniformMatrix4fv(modelViewMatLoc, 1, GL_FALSE, value_ptr(modelViewMat)); 

   //Draw SkyBox
   glUniform1ui(objectLoc, SKYBOX);  //if (object == SKYBOX)
   skybox.SetViewMatrix(modelViewMatLoc, modelViewMat);
   skybox.Draw(programId);
```

If you have done all changes, you should be able to add Skybox into the scene. 
There is a completed example project with model class added. The project (SkyBoxModel.zip) can be downloaed in this folder.
The example of output is shown below.

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%207/Readme%20Pictures/SkyModelScreenshot.JPG)

## Look around camera

In this section, you will learn how to Add a camera which can be controlled by key pressing to look around the scene.



### Basic theory. 

To look around the scene we have to change the cameraForward vector based on the input of the keyboard. However, changing the direction vector is a little complicated and requires some trigonometry. 
If you do not understand the trigonometry, don't worry, you can just skip to the code sections and paste them in your code; you can always come back later if you want to know more.. 


* Euler angles 

Euler angles are 3 values that can represent any rotation in 3D, defined by Leonhard Euler somewhere in the 1700s. 
There are 3 Euler angles: pitch, yaw and roll. The following image gives them a visual meaning:

![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%207/Readme%20Pictures/Euler.png)

The pitch is the angle that depicts how much we're looking up or down as seen in the first image. 
The second image shows the yaw value which represents the magnitude we're looking to the left or to the right. 
The roll represents how much we roll as mostly used in space-flight cameras. 
Each of the Euler angles are represented by a single value and with the combination of all 3 of them we can calculate any rotation vector in 3D.

For our camera system we only care about the yaw and pitch values so we won't discuss the roll value here. 
Given a pitch and a yaw value we can convert them into a 3D vector that represents a new direction vector. 
The process of converting yaw and pitch values to a direction vector requires a bit of trigonometry.


```C++
cameraForward.x = cos(glm::radians(yaw)) * cos(glm::radians(pitch));
cameraForward.y = sin(glm::radians(pitch));
cameraForward.z = sin(glm::radians(yaw)) * cos(glm::radians(pitch));
```

### Implmentation codes.

* Add Yaw and Pitch definitions (global variables)

```C++
float cameraYaw = 90;
float cameraPitch;
```

* Create UpdateCamera function based trigonometry

```C++
void UpdateCamera()
{
	if (cameraPitch < -89)
	{
		cameraPitch = -89;
	}
	if (cameraPitch > 89)
	{
		cameraPitch = 89;
	}
	glm::vec3 eye = glm::vec3(0,0,0);
	eye.x = glm::cos(glm::radians(cameraPitch)) * -glm::cos(glm::radians(cameraYaw));
	eye.y = glm::sin(glm::radians(cameraPitch));
	eye.z = glm::cos(glm::radians(cameraPitch)) * glm::sin(glm::radians(cameraYaw));

	cameraForward = glm::normalize(eye);

	glm::mat4 modelViewMat = glm::lookAt(cameraForward, glm::vec3(0), glm::vec3(0, 1, 0));
	modelViewMat = glm::translate(modelViewMat, cameraPosition);
	skybox.SetViewMatrix(modelViewMat);
}
```

* Add Keyboard handling codes

w move forward. s move backword. a move left. d move right. e move up. q move down.


```C++
void KeyInputCallback(unsigned char key, int x, int y)
{
	switch (key)
	{
	case 'w':
	{
		cameraPosition += cameraForward;
		UpdateCamera();
	}break;
	case 's':
	{
		cameraPosition += -cameraForward;
		UpdateCamera();
	}break;
	case 'a':
	{
		cameraPosition += -glm::normalize(glm::cross(cameraForward, glm::vec3(0, 1, 0)));
		UpdateCamera();
	}break;
	case 'd':
	{
		cameraPosition += glm::normalize(cross(cameraForward, glm::vec3(0, 1, 0)));
		UpdateCamera();
	}break;
	case 'e':
	{
		cameraPosition += -glm::vec3(0, 1, 0);
		UpdateCamera();
	}break;
	case 'q':
	{
		cameraPosition += glm::vec3(0, 1, 0);
		UpdateCamera();
	}break;
	case 27:
	{
		exit(0);
	}break;
	}
}
```

## Add a Room
### This is only for the coursework of developing a pool game.
There are three ways to add a room into the scene.

(1)Create a room (basically just a textured cube) in 3DS Max or blender. Then loading it into the OpenGL game.
Make sure room is big enough to accommodate the pool table and small enough to fit within the viewing frustum defined by the projection matrix.
Make sure the camera is inside the room. The repeatable texture such as wood is a good choice.

(2)Another way is to create a OpenGL cube from scratch. There is a tutorial in week 3. But you need to modify the vertex data to include the texture coordinates.
You can use repeatable texture such as wood to decorate the room. 

(3) Use the Skybox tutorial to create a room and replace skybox texture images with some wood texture images.

## Your own project

Now, you should be able to work on your own project 

For example, 

* Add skybox into your project (optional), if your game requires an outdoor environment.

* Add look around camera into your project. You have to modify the model class to allow input modelview matrix from camera setup.
change void Model::updateModelMatrix(unsigned int modelViewMatLoc,float d,float scale,float ZPos) function (both input parameters and codes)

* Add mouse control camera codes (optional, advanced level)
Tutorials: https://learnopengl.com/Getting-started/Camera






