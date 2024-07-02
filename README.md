# MyUtilsDrawer

<a name="table-of-contents"></a>
## Table of Contents:
> [ColorsFunctions:](#colors-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Colorspace conversion: sRGB to Linear and vice versa](#srgb-2-linear)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Layer Color](#layer-color)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Gradient Shader](#gradient-shader)<br>
> [ParabolicFunctions:](#parabolic-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Calculate parabolic jump height and time](#parabolic-jump)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Calculate parabolic shot height, distance and time](#parabolic-shot)<br>

<a name="colors-functions"></a>
## Colors functions:

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
<a name="parabolic-functions"></a>
## Parabolic functions:

<a name="parabolic-jump"></a>
### Calculate parabolic jump height and time

[https://www.desmos.com/calculator/vdkdg6z9pg?lang=es]

Input Parameters:
* ParabolaHeight
* ImpactTime

Derivated Parameters:
* GravityScale
* InitialVelocity

```cpp
static void CalculeJumpGravityAndInitialVelocity(double Height, double Time, double &GravityScale, double &InitialVelocity) {
	Time /= 2.0;
	GravityScale = 2.0 * Height / (Time * Time);
	InitialVelocity = FMath::Sqrt(2.0 * GravityScale * Height);
}
```
---
<a name="parabolic-shot"></a>
### Calculate parabolic shot height, distance and time

If you want to transform this in a parabolic shot and you want to add time parameter you'll need to do this:

Input Parameters:
* ParabolaDistance
* ParabolaHeight
* ImpactTime

Derivated Parameters:
* GravityScale
* InitialVelocity
* TimeDilation

Auxiliar variables:
* AccumulatedTime

```cpp
OnStart(){
	CalculeJumpGravityAndInitialVelocity(ParabolaHeight, ParabolaDistance, GravityScale, InitialVelocity);
	TimeDilation = ImpactTime / ParabolaDistance;
}

OnTick(float DeltaTime){
	AccumulatedTime += DeltaTime;

	float TimeDilated = AccumulatedTime / TimeDilation;
	float ZPosition = InitialVelocity * TimeDilated - 0.5f * GravityScale * TimeDilated * TimeDilated;

	SubsceneComponent->SetRelativeLocation(FVector(TimeDilated, 0, ZPosition));
}
```
