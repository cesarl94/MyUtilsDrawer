# MyUtilsDrawer

In this shared document, I plan to include all the functions that have helped me solve tasks more than once. It has become a common problem for me to remember in which project I did "this or that" and, on top of that, to find precisely where I did it. That's why I decided to start compiling everything in this document that can help solve a specific task, regardless of the project it is part of.

If you think any of these functions are useful to you, feel free to use them, but I encourage us to read and understand the code we're using, rather than just copying and pasting. These functions could be in any programming language, because the idea is the logic, not the sintaxys.

Some functions I created myself, others I borrowed from places with their respective links, and others I don't even remember where I got them from, so if you know the origin of some of these, let me know in my email: lorenzon.cesar@hotmail.com

<a name="table-of-contents"></a>
## Table of Contents:
> [Material Functions:](#material-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Colorspace conversion: sRGB to Linear and vice versa](#srgb-2-linear)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Layer Color](#layer-color)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Gradient Shader](#gradient-shader)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Resolve Decal Artifacts](#decal-artifacts)<br>
> [Parabolic Functions:](#parabolic-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Parabolic motion 1D and 2D:](#parabolic-motion)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Calculate parabolic jump from height and duration](#parabolic-jump)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Calculate parabolic shot from height, distance and duration](#parabolic-shot)<br>

<a name="material-functions"></a>
## Material functions:

<a name="srgb-2-linear"></a>
### Colorspace conversion: sRGB to Linear and vice versa
```javascript
function srgbToLinear(c) {
    if (c <= 0.04045) {
        return c / 12.92;
    } else {
        return Math.pow((c + 0.055) / 1.055, 2.4);
    }
}

function linearToSrgb(c) {
    if (c <= 0.0031308) {
        return 12.92 * c;
    } else {
        return 1.055 * Math.pow(c, 1 / 2.4) - 0.055;
    }
}

function srgbToLinearRgb(srgb) {
    return srgb.map(c => srgbToLinear(c));
}

function linearToSrgbRgb(linearRgb) {
    return linearRgb.map(c => linearToSrgb(c));
}

// Example:
const srgbColor = [0.25, 0.5, 0.75];
const linearRgbColor = srgbToLinearRgb(srgbColor);
console.log("sRGB to Linear RGB:", linearRgbColor);

const linearRgbColor2 = [0.05, 0.25, 0.5];
const srgbColor2 = linearToSrgbRgb(linearRgbColor2);
console.log("Linear RGB to sRGB:", srgbColor2);
```
---
<a name="layer-color"></a>
### Layer Color
This is the equivalent to put an color above other, supossing that we are using colors with alpha channel

```glsl
vec4 layerColor(vec4 topColor, vec4 bottomColor){
    vec3 finalColor = topColor.rgb * topColor.a + bottomColor.rgb * bottomColor.a * (1.0 - topColor.a);
    float finalAlpha = bottomColor.a + topColor.a * (1.0 - bottomColor.a);
    return vec4(finalColor, finalAlpha);
}
```
---
<a name="gradient-shader"></a>
### Gradient Shader
Piece of code of a fragment shader that receives two colors and two gradient points to create a interpolation
```glsl
varying vec2 vUvs;

uniform sampler2D uSampler;
uniform vec4 colorA;
uniform vec4 colorB;
uniform vec2 gradientPointA;
uniform vec2 gradientPointB;

void main() {
    vec4 mainColor = texture2D(uSampler, vUvs);

     // Storing vector A->P
    vec2 a_to_p = vec2(vUvs.x - gradientPointA.x, vUvs.y - gradientPointA.y);
    // Storing vector A->B
    vec2 a_to_b = vec2(gradientPointB.x - gradientPointA.x, gradientPointB.y - gradientPointA.y);

    //Basically finding the squared magnitude of a_to_b
    float atb2 = a_to_b.x * a_to_b.x + a_to_b.y*a_to_b.y; 
    //The dot product of a_to_p and a_to_b
    float atp_dot_atb = a_to_p.x * a_to_b.x + a_to_p.y * a_to_b.y;

    float t = atp_dot_atb / atb2;

    mainColor.rgb = mix(colorA.rgb,colorB.rgb,vec3(t));
    gl_FragColor = mainColor;
    gl_FragColor *=  mainColor.a;
}
```
---
<a name="decal-artifacts"></a>
### Resolve Decal Artifacts:


| Before | After |
| --- | --- |
| <img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhe6Q42EAgEh1jn6QfbXDKI9rI0UdNXoSq2L3o4lullGLDPg5re5uGQtSfIvSEzKO3ofYU6b2QtYRNP-gnbZ4DT7CvecciIdT5-nRlE-oPbp6NSle7ZmSU5_VtZO_NHCtVQXVpxwQl9YtI/s320/ScreenShot00004.png"> | <img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEht1cZMyhpkr2WJly7RKdExOHQZNnZLRGuPnPlqfbnI8dFDa3HMBeg1K3pI9kbvypwUjfM_o_f4f_9qhm0_gmD9ozYu0bqr9uirnNKSYNosZIeGuHLrNmNKlNm3Ub-ybOFOTo_jkvTXG7c/s320/ScreenShot00005.png"> |

You need to enter in this link: https://grephicsnerd.blogspot.com/2017/07/clip-stretching-of-deferred-decal-in-ue4.html<br>
<img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhzLYNGgdkAKjbnzXcf5G5R1zXRk0808CbNPaVQt1MdmI4XuWefUNUIcq9qYmodz2NZxmZrmai2aw9aLsjy1q5fE6d086s1sZoIAg4Qiona5Ceq-vbxj4FO1AgTysyGLSg6QMtnog0KJ-Y/s1600/worldnormal_without_gbuffer.PNG">
* The first node is Absolute World Position
* Don't forget to convert the result. The last "TransformVector" node is a positive vector [0,1], not a unit vector [-1,1]

---
<a name="parabolic-functions"></a>
## Parabolic functions:

<a name="parabolic-motion"></a>
### Parabolic motion 1D and 2D:

```
y(t) = y0 + v0 * t + 1/2 * g * t^2
```
```
y(t) = y0 + v0 * sin(a) * t + 1/2 * g * t^2
x(t) = x0 + v0 * cos(a) * t
```

Where ``y0`` or ``x0`` are the initial position, ``v0`` is the initial velocity, ``g`` is gravity acceleration, ``a`` is the launch angle, and ``t`` is the time in seconds

---
<a name="parabolic-jump"></a>
### Calculate parabolic jump from height and duration

https://www.desmos.com/calculator/vdkdg6z9pg?lang=es

Input Parameters:
* Height
* Duration

Derivated Parameters:
* Gravity
* InitialVelocity

```cpp
static void CalculeJumpGravityAndInitialVelocity(double Height, double Duration, double &Gravity, double &InitialVelocity) {
	Duration /= 2.0;
	Gravity = 2.0 * Height / (Duration * Duration);
	InitialVelocity = FMath::Sqrt(2.0 * Gravity * Height);
}
```

And then you can use it in your character like this (this will fall for the eternity) 
```cpp
void OnStart(){
    CalculeJumpGravityAndInitialVelocity(Height, Duration, Gravity, InitialVelocity);
}

void OnJump(){
    Velocity.Z = InitialVelocity;
}

void OnTick(float DeltaTime){
    Velocity.Z += Gravity * DeltaTime;
    Position.Z += Velocity.Z * DeltaTime;
}
```
---
<a name="parabolic-shot"></a>
### Calculate parabolic shot from height, distance and duration

If you want to make a parabolic shot and you want to add time parameter to the previous function you'll need to do this:

Input Parameters:
* ParabolaDistance
* ParabolaHeight
* ParabolaDuration

Derivated Parameters:
* Gravity
* InitialVelocity
* TimeDilation


```cpp
void OnStart(){
    CalculeJumpGravityAndInitialVelocity(ParabolaHeight, ParabolaDistance, Gravity, InitialVelocity);
    TimeDilation = ParabolaDuration / ParabolaDistance;
}

void OnTick(float DeltaTime){
    AccumulatedTime += DeltaTime;

    float TimeDilated = AccumulatedTime / TimeDilation;
    float ZPosition = InitialVelocity * TimeDilated - 0.5f * Gravity * TimeDilated * TimeDilated;

    SubsceneComponent->SetRelativeLocation(FVector(TimeDilated, 0, ZPosition));
}
```
