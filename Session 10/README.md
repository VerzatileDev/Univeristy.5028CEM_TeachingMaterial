# Session 10 - Toon Shader


Welcome to Week 10! 

> In this week, we will learn how to create a Toon Shader.

## Toon Shader

Toon shading is probably the simplest non-photorealistic shader we can write. 
It uses very few colors, usually tones, hence it changes abruptly from tone to tone, yet it provides a sense of 3D to the model. 
The following image shows what we’re trying to achieve..

 ![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%2010/Readme%20Pictures/Example.JPG)

The tones in the teapot above are selected based on diffuse light calculation.

So if we have a normal that is close to the light’s direction, 
then we’ll use the brightest tone. As the angle between the normal and the light’s direction increases darker tones will be used.
 In other words, the cosine of the angle provides an intensity for the tone.


### Implementation

* Download the base project (ToonShader.zip). Always to Compile option to "x64".  Open fragmentShader.glsl

we will do the toon shader effect per fragment. Therefore, we only need to modify the fragment shader.

Modify the color calculation codes based on the intensity value which is calculated as dot product between the light vector and the normal vector.
Do not forget to define the intensity variable before the main function in the fragment shader.

```C++
   if (object == SPHERE) {
    normal = normalize(normalExport);
	lightDirection = normalize(vec3(light0.coords));
	intensity = dot(lightDirection,normal);
	fAndBDif = max(dot(normal, lightDirection), 0.0f) * (light0.difCols * sphereFandB.difRefl); 
    if (intensity > 0.85)
		colorsOut =  0.85f*fAndBDif;
	else if (intensity > 0.6)
		colorsOut =  0.6f*fAndBDif;
	else if (intensity > 0.4)
		colorsOut =  0.4f*fAndBDif;
	else if (intensity > 0.2)
		colorsOut =  0.2f*fAndBDif;
	else
		colorsOut =  0.1f*fAndBDif;
   }
```


It should look this

 ![Tex1 picture](https://github.coventry.ac.uk/ac7020/212CR_TeachingMaterial/blob/master/Session%2010/Readme%20Pictures/Result.JPG)
 
 * You can adjust intensity values to achieve better results. 










