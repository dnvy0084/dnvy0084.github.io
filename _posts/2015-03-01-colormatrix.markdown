---
layout: post
title:  "Color Matrix"
date:   2015-03-01 10:00:00
categories: update
---

[Pixel Manipulation] 성능
===

Canvas에서 color값 변경을 통한 Image Processing은 기본적으로 전체 픽셀에 대한 for loop 안에서 더하거나, 곱하거나 평균값을 내거나 하는 방법으로 진행됩니다. 만약 하나의 사진에 밝기, 명암, 색상을 한꺼번에 적용한다면 for loop 안에서 3개 효과에 해당하는 계산을 수행하거나(이런 경우 동적으로 효과 적용이 힘들어 집니다.), brightness(), contrast(), saturation()을 한번씩 호출하여 총 3번의 for loop로 효과를 적용할 수 있는데요, 당연히 성능은 바닥을 달릴것 같습니다. 

이럴때 Matrix를 이용할수 있습니다. 밝기, 명암, 색상 계산에 대한 각각의 matrix를 만들고 서로 곱하여 하나의 matrix가 되면 for loop안에서 matrix * pixel로 한번에 적용이 가능합니다. 

```javascript 
var brightnessMatrix,   //밝기
    contrastMatrix,     //명암
    saturationMatrix;   //색상

var transformMatrix = brightnessMat * contrastMat * saturationMat;

for( var i = 0, l = src.length; i < l; i += 4 )
{
    dest = transformMatrix * src.subarray( i, i + 4 );
}
```

위 소스코드 처럼 for loop안에서는 $matrix \times pixel(vector)$ 곱으로 여러가지 효과를 한번에 적용할 수 있습니다. 

Color Matrix
===

Color(pixel) 변환용 matrix를 Color Matrix라고 합니다. Action script에서 ColorMatrixFilter와 같다고 보면 됩니다. 2D(x,y) 변환에는 3x3 Matrix가, 3D(x,y,z) 변환에는 4x4 Matrix가 사용되는데요, Color 는 R,G,B,A 4가지 원소를 가지고 있으니 변환용 Matrix도 그보다 하나 많은 5x5 Matrix를 사용합니다. alpha값은 변경하지 않아도 되는 앱이라면 4x4 Matrix 사용으로 약간(!)이나마 성능 향상을 노려볼 수 있겠네요. 

5x5 Matrix와 Color Vector의 곱은 아래와 같습니다. 

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%20%0A1%20%26%200%20%26%200%20%26%200%20%26%200%20%5C%5C%20%0A0%20%26%201%20%26%200%20%26%200%20%26%200%20%5C%5C%20%0A0%20%26%200%20%26%201%20%26%200%20%26%200%20%5C%5C%20%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%20%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%5C%5C%20%0A%5Cend%7Bbmatrix%7D%20%0A" /> 
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%20%0Ar%20%5C%5C%0Ag%20%5C%5C%0Ab%20%5C%5C%0Aa%20%5C%5C%0A1%20%5C%5C%0A%5Cend%7Bbmatrix%7D%20%0A" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%20%0Ar%20%5C%5C%0Ag%20%5C%5C%0Ab%20%5C%5C%0Aa%20%5C%5C%0A1%20%5C%5C%0A%5Cend%7Bbmatrix%7D%20%0A" />

단위 행렬을 곱해서 rgba값에 변화가 없지만 아래 처럼 곱해지게 됩니다. 


<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0Aa%20%26%20b%20%26%20c%20%26%20d%20%26%20translateR%20%5C%5C%0Ae%20%26%20f%20%26%20g%20%26%20h%20%26%20translateG%20%5C%5C%0Ai%20%26%20j%20%26%20k%20%26%20l%20%26%20translateB%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%20translateA%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

$R' = aR + bG + cB + dA + 1 \cdot translateR$

$G' = eR + fG + gB + hA + 1 \cdot translateG$

$B' = iR + jG + kB + lA + 1 \cdot translateB$

$A' = 0\cdot R + 0\cdot G + 0\cdot B + 0\cdot A + 1 \cdot translateA$

$1 = 0\cdot R + 0\cdot G + 0\cdot B + 0\cdot A + 1 \cdot 1$

복잡해 보이지만 3x3 행렬 $\times$ 1x3행렬(x,y,1)과 같습니다. 


Brightness Matrix
===

color matrix를 이용해 brightness 값을 변경하는 방법은 간단합니다. transformation에서 tx, ty를 이용해 이동해주는 것 처럼 translateRed, translateBlue ... 를 통해 +, - 해주면 됩니다. 

$b = brightness$

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A1%20%26%200%20%26%200%20%26%200%20%26%20b%20%5C%5C%0A1%20%26%201%20%26%200%20%26%200%20%26%20b%20%5C%5C%0A1%20%26%200%20%26%201%20%26%200%20%26%20b%20%5C%5C%0A1%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A1%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

이때 b값을 각각 따로 주면 선택한 색상만 밝기 조절도 가능합니다. 


Grayscale
===

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

<!-- 
\begin{bmatrix}
0.21 & 0.72 & 0.07 & 0 & 0 \\
0.21 & 0.72 & 0.07 & 0 & 0 \\
0.21 & 0.72 & 0.07 & 0 & 0 \\
0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 
\end{}
  -->

Invert
===

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A-1%20%26%200%20%26%200%20%26%200%20%26%20255%20%5C%5C%0A0%20%26%20-1%20%26%200%20%26%200%20%26%20255%20%5C%5C%0A0%20%26%200%20%26%20-1%20%26%200%20%26%20255%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

<!-- 
\begin{bmatrix}
-1 & 0 & 0 & 0 & 255 \\
0 & -1 & 0 & 0 & 255 \\
0 & 0 & -1 & 0 & 255 \\
0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 
\end{}
  -->

Black & White
===

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A1%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A1%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>


Contrast
===

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A1%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A1%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>


Saturation
===

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A0.21%20%26%200.72%20%26%200.07%20%26%200%20%26%200%20%5C%5C%0A1%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A1%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

-- 작성 중.


[Pixel Manipulation]: http://dnvy0084.github.io/update/2015/02/25/pixel_manipulation.html

<!-- 
\begin{bmatrix}
1 & 0 & 0 & 0 & b \\
1 & 1 & 0 & 0 & b \\
1 & 0 & 1 & 0 & b \\
1 & 0 & 0 & 1 & 0 \\
1 & 0 & 0 & 0 & 1 
\end{} 
-->