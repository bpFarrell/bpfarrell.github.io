---
layout: post
title:  "Metroid Prime Visors"
date:   2019-08-18 16:02:34 -0700
categories: jekyll update
---

In the Metroid Prime games Samus has a series of tools to aid her in her adventures. A type of tool she can get are visor upgrades. These visors allow views of the world that are not normals visible, Such as a Scanner Visor, Thermal Visor, and X-Ray Visor. In this post we will be covering and breaking down the effects, how they are achieved, and how to implement them into a modern graphics engine.

<i>Notice: Thhs tutorial was written with Unity3D in mind, but the shaders we write, are very easy to port.</i>

---
<h2>X-Ray Visor</h2>The X-ray Visor has a very high contrast view that is supposed to imitate the images of an actual X-ray scan. There is a couple things to keep in mind when looking at Metroid’s implementation. 

![Xray Example](/Assets/Xray_Example.png)
* Surfaces accumulate ‘brightness’

* Content Darkens as it gets further away.

* The Color Pallet, it’s not quite black and white

With these in mind we can start working on our shader. I started off with a new unlit shader, and there are a few things we need to configure specifically for the transparency. First we need to set the render queue to transparent, and then change the blending mode to be ONE ONE
{% highlight c %}
SubShader
{
    Tags { "Queue" = "Transparent" "RenderType" = "Transparent" "IgnoreProjector" = "True" }
    ZWrite Off
    LOD 100
    Blend One One
    Pass
    {
    ...
{% endhighlight %}

At a low level this takes the pixel that is there, and adds the surface onto it, where normal rendering replaces if the surface is closer to the camera then what is already rendered. This is an effect desirable for lights and fire effects, that we are going to take advantage of.

Let’s Make a material with a texture, and put a few in our scene.

![Additive Example](/Assets/Xray_additiveTest.gif)

The overlapping, and accumulation is exactly what we want. Let’s set clear flags to Solid Color,the Background to be black on the camera, and change the shader to always output a gray.
![Camera](/Assets/CameraBackgroundSettings.png)

{% highlight c++ %}            
fixed4 frag (v2f i) : SV_Target
{
    return fixed4(0.25,0.25,0.25,1);
}
{% endhighlight %}
![Additive Example](/Assets/Xray_additiveTest2.gif)

This effect is starting to look a bit closer to what we want, but now we need to darken based off of the depth of the surface. Inside the shader We need to calculate the ‘clip space’ position for each vertex, and thus fragment. Once calculated we will put the screen position into the varying v2f so our fragment shader can read it. 
{% highlight c++ %}      
struct v2f
{
    ...
    float4 screenPos : TEXCOORD1; //Used to store the screen position for the frag shader
};      
v2f vert (appdata v)
{
    v2f o;
    ...
    o.screenPos = ComputeScreenPos(o.vertex);
    ...
    return o;
}
{% endhighlight %}

Unity has a built in method that calculates screen pose for us. Once we have the clip space, we want to use the z component to scale our color. I am using smoothstep with some hard coded values to calibrate the values.
{% highlight c++ %}            
fixed4 frag(v2f i) : SV_Target
{
    float4 clip = i.screenPos / i.screenPos.w;
    return fixed4(1, 1, 1, 1) * smoothstep(0, .025, clip.z);
}
{% endhighlight %}
![Additive Example](/Assets/Xray_additiveTest3.gif)

Keep in mind when you pass something from the vertex to the fragment shader, it tri-linearly interpolates every varying value, for every rendered pixel. This allows a smooth transition of color, and even uvs to stretch over the surface!

Now we need to introduce the blue into the midtones. There are a few ways to approuch this, you could mathmatically define the color curve, but that can be difficult to match, and can take a decent amount of time. I choose to do a Look Up Texture, or LUT. A LUT is a means to use a value to drive where the shader is looking into a texture. After a short time in Photoshop, I save out a gradient that I will use for the LUT

![Xray LUT](/Assets/xray_LUT.png)

<i>Note the texture I actaully saved out, and am using is 256x2 pixels I scaled it up to be more </i>visible.

- SCreenspace
- Use Lut


---
<h2>Thermal Visor</h2>
Lut the Dot

{% highlight c# %}
v2f vert (appdata v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    o.viewDir = ObjSpaceViewDir(v.vertex);
    o.normal = v.normal;
    UNITY_TRANSFER_FOG(o,o.vertex);
    return o;
}

fixed4 frag (v2f i) : SV_Target
{
    float fresnel = dot(
        normalize(i.viewDir), 
        normalize(i.normal));

    float therm = smoothstep(
        _Heat.x - _Heat.y,
        _Heat.x + _Heat.y,
        fresnel);
    return 
        tex2D(_LUTHeat,
            fixed2(lerp(therm,
            _Heat.z,
            _Heat.w)*(saturate(_LastVisorTime*3)),
            0.5));
}
{% endhighlight %}

---
<h2>Integrating</h2>
Shader replace on key stroke

---
<h2>Transitions</h2>
something with a `*(1-)`

---
<h2>Thoughts</h2>
...profit?

---
my other stuff

![My helpful screenshot](/Assets/OffRegister_test_4.gif)
{% highlight c# %}
void Main(){
	if(int x == 0){
		Console.log("Fun times!");
		return;
	}
}
{% endhighlight %}
