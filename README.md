# MyUtilsDrawer

In this shared document, I plan to include all the functions that have helped me solve tasks more than once. It has become a common problem for me to remember in which project I did "this or that" and, on top of that, to find precisely where I did it. That's why I decided to start compiling everything in this document that can help solve a specific task, regardless of the project it is part of.

If you think any of these functions are useful to you, feel free to use them, but I encourage us to read and understand the code we're using, rather than just copying and pasting. These functions could be in any programming language, because the idea is the logic, not the sintaxys.

Some functions I created myself, others I borrowed from places with their respective links, and others I don't even remember where I got them from, so if you know the origin of some of these, let me know in my email: lorenzon.cesar@hotmail.com

<a name="table-of-contents"></a>
## Table of Contents:
> [Maths Functions:](#maths-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Linear Step](#linear-step)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Rule of Five](#rule-of-five)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Line Intersection](#line-intersection)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Reflect Ray](#reflect-ray)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Normalize Degrees](#normalize-degrees)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Normalize Radians](#normalize-radians)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Difference in Degrees](#difference-in-degrees)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Difference in Radians](#difference-in-radians)<br>
> [Material Functions:](#material-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Colorspace conversion: sRGB to Linear and vice versa](#srgb-2-linear)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Layer Color](#layer-color)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Gradient Shader](#gradient-shader)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Resolve Decal Artifacts](#decal-artifacts)<br>
> [Parabolic Functions:](#parabolic-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Parabolic motion 1D and 2D:](#parabolic-motion)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Calculate parabolic jump from height and duration](#parabolic-jump)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Calculate parabolic shot from height, distance and duration](#parabolic-shot)<br>
> [Unreal Functions:](#unreal-functions)<br>
> &nbsp;&nbsp;&nbsp;&nbsp;[Capture variables in Lambda functions:](#lambda-capture)<br>

<a name="maths-functions"></a>
## Maths Functions:

<a name="linear-step"></a>
### Linear Step:
```cpp
/**
 * AKA InverseLerp
 * if you give me an factor0Param I return an 0.
 * if you give me an factor1Param I return an 1.
 * then you can give me any input and I will return a value between 0 and 1 according to the values passed before.
 */
UFUNCTION(BlueprintCallable, Category = "MathUtils")
static double LinearStep(double factor0Param, double factor1Param, double input, bool clamp) {
	const double denom = factor1Param - factor0Param;
	if (denom == 0) {
		return input < factor0Param ? 0 : 1;
	}
	const double t = (input - factor0Param) / denom;
	return clamp ? FMath::Clamp(t, 0, 1) : t;
}
```

<a name="rule-of-five"></a>
### Rule of Five:
```cpp
/**
 * if you give me an inputDataA I return an outputDataA.
 * if you give me an inputDataB I return an outputDataB.
 * then you can give me any input and I will return a value according to the values passed before.
 * If clamp is true, the returns values will be held between outputDataA and outputDataB
 *
 * If inputDataA is 0 and inputDataB is 1, you should use lerp instead this.
 * If outputDataA is 0 and outputDataB is 1, you should use linearstep instead this.
 */
UFUNCTION(BlueprintCallable, Category = "MathUtils")
static double RuleOfFive(double inputDataA, double outputDataA, double inputDataB, double outputDataB, double input, bool clamp) {
	const double t = LinearStep(inputDataA, inputDataB, input, clamp);
	return FMath::Lerp(outputDataA, outputDataB, t);
}
```

<a name="line-intersection"></a>
### Line intersection:
Computes the intersection point between two lines or segments.<br>
bInfiniteLines: If true, treats the inputs as infinite lines and returns any intersection, if false, ensures the intersection is within the given segments.
```cpp
static bool GetLineIntersection(double ax, double ay, double bx, double by, double cx, double cy, double dx, double dy, bool bInfiniteLines, double &rvx, double &rvy) {
	const double ux = bx - ax;
	const double uy = by - ay;
	const double vx = dx - cx;
	const double vy = dy - cy;
	const double div = ux * vy - uy * vx;
	// div 0 == parallel lines
	if (div == 0) {
		return false;
	}

	const double hx = cx - ax;
	const double hy = cy - ay;

	if (bInfiniteLines) {
		const double cross = hx * uy - hy * ux;
		const double t = cross / div;
		rvx = vx * t + cx;
		rvy = vy * t + cy;
		return true;
	}

	const double s = (hx * vy - hy * vx) / div;
	const double t = (hx * uy - hy * ux) / div;

	// Check if intersection is within both segments
	if (s < 0 || s > 1 || t < 0 || t > 1) {
		return false;
	}

	rvx = ax + s * ux;
	rvy = ay + s * uy;
	return true;
}
```

<a name="reflect-ray"></a>
### Reflect Ray:
Given the normal of a plane, and a ray, returns the direction of the reflected ray<br>
```cpp
static FVector ReflectRay(const FVector &PlaneNormal, const FVector &RayNormal) {
	FVector NormalizedPlaneNormal = PlaneNormal.GetSafeNormal();
	FVector NormalizedRayNormal = RayNormal.GetSafeNormal();
	return NormalizedRayNormal - 2.f * (NormalizedRayNormal | NormalizedPlaneNormal) * NormalizedPlaneNormal;
}
```

<a name="normalize-degrees"></a>
### Normalize Degrees:
Normalizes an angle in degrees to an equivalent angle within the range [0, 360)<br>
```cpp
static double NormalizeDegrees(double Degrees) {
	if (Degrees >= 0) {
		return FGenericPlatformMath::Fmod(Degrees, 360.0);
	}
	return Degrees + FGenericPlatformMath::CeilToDouble(Degrees / -360.0) * 360.0;
}
```

<a name="normalize-radians"></a>
### Normalize Radians:
Normalizes an angle in radians to an equivalent angle within the range [-PI, PI]<br>
```cpp
static double NormalizeRadians(double Radians) {
	return Radians - TWO_PI * FGenericPlatformMath::FloorToDouble((Radians + PI) / TWO_PI);
}
```

<a name="difference-in-degrees"></a>
### Difference in Degrees:
Calculates the minimum difference between two angles in degrees. The angles don't have to be normalized.<br>
The result could be used in 3 ways:<br>
* To approximate Origin to Target: If you add the ReturnedValue to Origin (totally or partially), you'll get closer to the Target angle<br>
* To know the aperture between both angles: Calculate Abs(ReturnedValue) and you'll get the aperture<br>
* To know the clockwise of the difference: The sign of the ReturnedValue can give you the clue if you need turn clockwise or counterclockwise<br>

Origin: Given Angle in degrees<br>
Target: The angle in degrees which Origin wishes to be or approximate<br>

Returns the signed difference between two angles, in the range [-180, 180). If you add this to Origin you'll have the same equivalent angle<br>

```cpp
static double DifferenceInDegrees(double Origin, double Target) {
	double diff = FGenericPlatformMath::Fmod((Target - Origin + 180.0), 360.0) - 180.0;
	return diff < -180.0 ? diff + 360.0 : diff;
}
```

<a name="difference-in-radians"></a>
### Difference in Radians:
Calculates the minimum difference between two angles in radians. The angles don't have to be normalized.<br>
The result could be used in 3 ways:<br>
* To approximate Origin to Target: If you add the ReturnedValue to Origin (totally or partially), you'll get closer to the Target angle<br>
* To know the aperture between both angles: Calculate Abs(ReturnedValue) and you'll get the aperture<br>
* To know the clockwise of the difference: The sign of the ReturnedValue can give you the clue if you need turn clockwise or counterclockwise<br>

Origin: Given Angle in radians<br>
Target: The angle in radians which Origin wishes to be or approximate<br>

Returns the signed difference between two angles, in the range (-PI, PI]. If you add this to Origin you'll have the same equivalent angle<br>

```cpp
static double DifferenceInRadians(double Origin, double Target) {
	// Simply normalize the raw difference
	return NormalizeRadians(Target - Origin);
}
```

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

<a name="unreal-functions"></a>
## Unreal functions:

<a name="lambda-capture"></a>
### Capture variables in Lambda functions:

Read this document to understand how Lambda functions work in C++: https://learn.microsoft.com/en-us/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170

I found it quite confusing regarding capturing variables by copy or by reference. If we are working with pointers, it would logically seem that since we are using referenced data, we should pass the values as references (with a "&" before the variable name). However, we still need to pass the values by copy. In any case, what we will do is create a copy of the pointer.

Practical usage example with a Lambda function inside another Lambda function:
```cpp
// A GameState with the data of the duration of both Timers
ACustomGameState *GameState = Cast<ACustomGameState>(UGameplayStatics::GetGameState(this));
GetWorld()->GetTimerManager().SetTimer(TimerHandle,
    [this, GameState]() { // It's ok! But if I use "[this, &GameState]" I will receive an error!
        SlimeCollider->SetCollisionEnabled(ECollisionEnabled::NoCollision); // This doesn't matter
        K2_StartRemoval(); // This doesn't matter

        // We start another Lambda function with the data read from our custom GameState
        GetWorld()->GetTimerManager().SetTimer(TimerHandle,
            [this]() { Destroy(); }, // "this" is still valid!
            GameState->GameData->SlimeDisappearAnimationDuration, false);
    },
    GameState->GameData->SlimeDuration, false);
```
