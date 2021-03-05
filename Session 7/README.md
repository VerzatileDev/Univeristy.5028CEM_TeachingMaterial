# Session 7 - Advanced Topics One

#### Table of Contents
1. [Skybox](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%207#Skybox)
2. [Integrate into your project](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%207#Integrate-into-your-project)
3. [Look around camera](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%207#Look-around-camera)
4. [Your own project](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/tree/master/Session%207#Your-own-project)

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
 incrementing the enum by 1 each iteration, effectively looping through all the texture targets:

```C++
for (unsigned int i = 0; i < 6; i++)
 glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data); 
```

Because a cubemap is a texture like any other texture, we will also specify its wrapping and filtering methods

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

### Try to add a skybox into your project (Optional)

* Now, you can try to add the skybox into your own project. Following instructions were just for guidance (probably not able to solve every problem you have)

* add skybox texture map into your texture folder.  The list is

```C++
"Textures/SkyboxRight.jpg",
"Textures/SkyboxLeft.jpg",
"Textures/SkyboxTop.jpg",
"Textures/SkyboxBottom.jpg",
"Textures/SkyboxFront.jpg",
"Textures/SkyboxBack.jpg"
```

* Copy Both Skybox.cpp and Skybox.h into your project folder

* Copy Both SkyboxFragmentShader.glsl and SkyboxVertexShader.glsl into your project.
Both shaders are stand-alone shader, you do not need to add them into your main shaders.

* Add Skybox initialization codes into your main.cpp. Please refer to example project main.cpp to find out where to put them.

```C++
Skybox skybox;

skybox.InitialiseCubeMap();	
skybox.InitialiseSkybox();
skybox.CreateShader("SkyboxVertexShader.glsl", "SkyboxFragmentShader.glsl");
skybox.SetViewMatrix(modelViewMat);	


skybox.Bind();
skybox.Draw();

	
```

* Also, you need to be careful with textureID in Skybox class, which gives you the information which textureID has been used.
So, you should use skybox texture as the first textureID and add your other textures after it.

## Integrate into your project

This is an optional task.
The Skybox can not directly into your project (not just copy paste job!). The Skybox example project uses a stand-alone shader architecture.


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

## Your own project

Now, you should be able to work on your own project 

For example, 

* Add skybox into your project (optional)

* Add look around camera into your project. You have to modify the model class to allow input modelview matrix from camera setup.
change void Model::updateModelMatrix(unsigned int modelViewMatLoc,float d,float scale,float ZPos) function (both input parameters and codes)

* Add mouse control camera codes (optional, advanced level)
Tutorials: https://learnopengl.com/Getting-started/Camera






