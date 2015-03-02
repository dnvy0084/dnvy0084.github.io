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

행렬을 아래와 같이 가정하고 풀이를 보면,


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

<!-- 
\begin{bmatrix}
1 & 0 & 0 & 0 & b \\
0 & 1 & 0 & 0 & b \\
0 & 0 & 1 & 0 & b \\
0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 
\end{}
 -->

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A1%20%26%200%20%26%200%20%26%200%20%26%20b%20%5C%5C%0A0%20%26%201%20%26%200%20%26%200%20%26%20b%20%5C%5C%0A0%20%26%200%20%26%201%20%26%200%20%26%20b%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

Red 값만 살펴보면

$R' = 1 \times R + 0 \times G + 0 \times B + 1 \times A + 1 \times b$
가 되어 결과적으로 

$R + b$ 만 남게 됩니다. 

이때 b값을 각각 따로 주면 선택한 색상만 밝기 조절도 가능합니다.


Invert
===

<!-- 
\begin{bmatrix}
-1 & 0 & 0 & 0 & 255 \\
0 & -1 & 0 & 0 & 255 \\
0 & 0 & -1 & 0 & 255 \\
0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 
\end{}
  -->

이미지 반전은 $255 - r, 255 - g, 255 - b$로 구할 수 있으니 $-1 \times r + 255$로 보고 다음과 같이 정리할 수 있습니다.

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A-1%20%26%200%20%26%200%20%26%200%20%26%20255%20%5C%5C%0A0%20%26%20-1%20%26%200%20%26%200%20%26%20255%20%5C%5C%0A0%20%26%200%20%26%20-1%20%26%200%20%26%20255%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

여기서 $t(0 <= t <= 1)$로 이미지 반전 정도를 정할 수 있도록 하려면, 단위 행렬 I, 이미지 반전 행렬 M이 있을때, $M' = I + t( M - I )$로 무게중심을 이용해 구할 수도 있고, $R' = R + t( invertedR - R )$처럼 RGB 3개 원소에 대한 계산을 Matrix로 만들어 곱해주는 방법이 있습니다. 후자를 정리해 보자면, 

$R' = R + t( invertedR - R )$

$R' = R + t( 255 - R - R )$

$R' = R + t \cdot 255 - t \cdot 2 \cdot R$

$R' = ( 1 - 2 \cdot t ) \cdot R + 255 \cdot t$

$y = ax + b$의 형태로 바꿔,


<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A1-2%5Ccdot%20t%20%26%200%20%26%200%20%26%200%20%26%20255%20%5Ccdot%20t%20%5C%5C%0A0%20%26%201-2%5Ccdot%20t%20%26%200%20%26%200%20%26%20255%20%5Ccdot%20t%20%5C%5C%0A0%20%26%200%20%26%201-2%5Ccdot%20t%20%26%200%20%26%20255%20%5Ccdot%20t%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D" />

위와 같은 행렬이 됩니다. t를 변경하면 이미지 반전 정도를 조절할 수 있습니다. 


<!-- 
\begin{bmatrix}
1-2\cdot t & 0 & 0 & 0 & 255 \cdot t \\
0 & 1-2\cdot t & 0 & 0 & 255 \cdot t \\
0 & 0 & 1-2\cdot t & 0 & 255 \cdot t \\
0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 
\end{}
 -->


Grayscale
===

grayscale은 인간의 눈이 3원색에 민감한 정도에 따라 고정된 가중치를 적용하여 구하는데요, 각각 0.21, 0.72, 0.07을 곱해줍니다. 따라서 아래와 같이 정리할 수 있습니다. 

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

여기서도 t를 통해 grayscale 정도를 조절하려면 

$R' = R + t( GrayscaleR - R )$ 를 $ax + b$의 형태로 바꿔 행렬로 적용할 수 있습니다. 

$R' = R + t( 0.21 \cdot R + 0.72 \cdot G + 0.07 \cdot B - R )$

$R' = R + 0.21 \cdot t \cdot R + 0.72 \cdot t \cdot G + 0.07 \cdot t \cdot B - t \cdot R$

$R' = ( 1 - t + 0.21 \cdot t ) R + 0.72 \cdot G + 0.07 \cdot B$가 되어( G,B도 각각 정리하면 )

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A1-t%2B0.21t%20%26%200.72t%20%26%200.07t%20%26%200%20%26%200%20%5C%5C%0A0.21t%20%26%201-t%2B0.72t%20%26%200.07t%20%26%200%20%26%200%20%5C%5C%0A0.21t%20%26%200.72t%20%26%201-t%2B0.07t%20%26%200%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D"/>

위와 같은 행렬로 만들 수 있습니다. 

<!-- 
\begin{bmatrix}
1-t+0.21t & 0.72t & 0.07t & 0 & 0 \\
0.21t & 1-t+0.72t & 0.07t & 0 & 0 \\
0.21t & 0.72t & 1-t+0.07t & 0 & 0 \\
0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 1 
\end{}
 -->


Black & White
===

흑백 이미지는 밝기( grayscale )가 0.5 이상이면 1 밑이면 0으로 구할 수 있습니다. 

```javascript
if( grayscale >= 128 ) return 255
else                   return 0
```

이라고 할 수 있는데요, 행렬에서는 if문 표현이 불가능하니 0~255 범위를 $-128$해서 -128 ~ +128로 바꾸고 $\times 128 + 128$하면 0이나 255로 바꿀 수 있습니다. 

$(0.21\times R + 0.72\times G + 0.07\times B - 128)\times 128 + 128$

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A0.21%20*%20128%20%26%200.72%20*%20128%20%26%200.07%20*%20128%20%26%200%20%26%20128%20-%20128%5E2%20%5C%5C%0A0.21%20*%20128%20%26%200.72%20*%20128%20%26%200.07%20*%20128%20%26%200%20%26%20128%20-%20128%5E2%20%5C%5C%0A0.21%20*%20128%20%26%200.72%20*%20128%20%26%200.07%20*%20128%20%26%200%20%26%20128%20-%20128%5E2%20%5C%5C%0A%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0A%5Cend%7B%7D"/>

<!-- 
\begin{bmatrix}
0.21 * 128 & 0.72 * 128 & 0.07 * 128 & 0 & 128 - 128^2 \\
0.21 * 128 & 0.72 * 128 & 0.07 * 128 & 0 & 128 - 128^2 \\
0.21 * 128 & 0.72 * 128 & 0.07 * 128 & 0 & 128 - 128^2 \\
\end{}
 -->

t까지 붙여서 정리하면 다음과 같습니다. 

$f=1-t,$ 

$scaleR=0.21 * 128 * t, scaleG = 0.72 * 128 * t, scaleB = 0.07 * 128 * t,$ 

$translate = 128 - 128^2$

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0Af%20%2B%20scaleR%20%26%20scaleG%20%26%20scaleB%20%26%200%20%26%20translate%20%5C%5C%0AscaleR%20%26%20f%20%2B%20scaleG%20%26%20scaleB%20%26%200%20%26%20translate%20%5C%5C%0AscaleR%20%26%20scaleG%20%26%20f%20%2B%20scaleB%20%26%200%20%26%20translate%20%5C%5C%0A%5Cend%7B%7D" />

Contrast
===

contrast는 $contrast \times ( p - 128 ) + 128$이니 $contrast \times p + 128 \times ( 1 - contrast )$로 정리하여 행렬로 바꿀 수 있습니다. 

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0Ac%20%26%200%20%26%200%20%26%200%20%26%20128%5Ctimes%20(1-c)%20%5C%5C%0A0%20%26%20c%20%26%200%20%26%200%20%26%20128%5Ctimes%20(1-c)%20%5C%5C%0A0%20%26%200%20%26%20c%20%26%200%20%26%20128%5Ctimes%20(1-c)%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%5C%5C%0A%5Cend%7B%7D" />
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

$c= contrast$


Saturation
===

채도는 grayscale의 반대입니다. $1-t+0.21t$에서 t값을 -로 주면 1보다 커져서 해당 색원소값이 가중치보다 커지게 됩니다. grayscale이 이미지에서 색상을 빼고 밝기만 남긴 형태이니 그 반대는 해당 픽셀이 원색에 가깝게 바뀌게 됩니다. 

<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0A1-t%2B0.21t%20%26%200.72t%20%26%200.07t%20%26%200%20%26%200%20%5C%5C%0A0.21t%20%26%201-t%2B0.72t%20%26%200.07t%20%26%200%20%26%200%20%5C%5C%0A0.21t%20%26%200.72t%20%26%201-t%2B0.07t%20%26%200%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%201%20%26%200%20%5C%5C%0A0%20%26%200%20%26%200%20%26%200%20%26%201%20%0A%5Cend%7B%7D"/>
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%20%5C%5C%0AG%20%5C%5C%0AB%20%5C%5C%0AA%20%5C%5C%0A1%20%0A%5Cend%7B%7D" />
$=$
<img src="http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%0AR%27%20%5C%5C%0AG%27%20%5C%5C%0AB%27%20%5C%5C%0AA%27%20%5C%5C%0A1%20%0A%5Cend%7B%7D"/>

-- 예제 작성 중 --


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
