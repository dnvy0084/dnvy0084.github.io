---
layout: post
title:  "Pixel Manipulation"
date:   2015-02-25 10:00:00
categories: update
---

색 좌표계
===

디지털 환경에서는 색을 저장하기 위해 **RGB**라는 색 좌표계를 이용합니다. 각각 Red, Green, Blue의 [빛의 삼원색]을 나타내며 [가산혼합]으로 색을 표현합니다. 

프로그래밍에서 R, G, B를 나타내는 혹은 저장하는 방법에는 여러가지가 있겠지만, 흔히 부호없는 정수형 타입을 많이 사용합니다 `Unsigned Integer 32`. 32개 비트에서 앞에서부터 차례대로 8개씩 Alpah, Red, Green, Blue 값으로 사용하며, 16진수로 0xAARRGGBB로 표기할 수 있습니다. 각각의 색 AA, RR, GG, BB는 8개 비트 $2^8$인 256단계를 표현(저장)할 수 있습니다. 

하나의 변수에 4개 값이 담겨져 있기 때문에 색을 바꾸기 위해서는 비트 연산을 주로 이용하게 됩니다. 

color라는 변수가 있을 때 

```javascript
color = 0xffccddee;

alpha = color >> 24, // 0xff
red = color >> 16 & 0xff, // 0xcc
green = color >> 8 & 0xff,  // 0xdd
blue = color & 0xff; // 0xee
```
이렇게 각 원소를 분리 할 수 있습니다. 다시 하나로 저장하려면 아래와 같습니다. 

```javascript
color = alpha << 24 | red << 16 | green << 8 | blue;
```

ImageData를 통한 픽셀 조작
===

Canvas는 [ImageData]라는 객체를 이용해 픽셀 조작을 할 수 있는데요, ImageData의 data라는 변수에 [TypedArray]인 [Uint8ClampedArray]형태로 저장되어 있습니다. TypedArray는 지정된 타입의 변수만 저장 가능한 strict한 배열입니다. as의 Vector.\<T\>와 같다고 볼 수 있으며, Uint8ClampedArray는 Vector.\<Uint8\> 으로 가정할 수 있는데, 당연히 as에는 Uint8 타입이 없습니다. 

js의 TypedArray중에는 Uint8ClampedArray와 함께 Uint8Array도 있습니다. 둘다 Uint8 <small>-부호없는 정수형 8비트 저장공간-</small> 타입만 저장 가능하며 8비트 보다 큰 값 257을 저장하면 Uint8Array는 1로 Uint8ClampedArray는 255로 저장됩니다. Uint8Array는 들어온 값의 마지막 8개 비트만 읽어서 저장하며 257은 이진수로 `1 0000 0001`이니 `0000 0001`로 저장되고, Uint8Clamped는 자동으로 clamp `Math.min( 0, Math.max( 255, 257 ) )` 됩니다.

ImageData.data는 항상 픽셀 개수 * 4의 길이를 가지는 데요, 한개의 픽셀을 r, g, b, a 4개의 원소로 저장하고 있기 때문입니다. 그래서 위 처럼 색 변환에 비트연산을 사용할 필요 없이 

```javascript
for( var i = 0, l = imageData.data.length; i < l; i += 4 )
{
  var data = imageData.data,

      r = data[i],
      g = data[i+1],
      b = data[i+2],
      a = data[i+3];

  // manipulation code

  data[i] = r,
  data[i+1] = g,
  data[i+2] = b,
  data[i+3] = a;
}
```
위 처럼 for loop를 통해 전체 픽셀을 조작할 수 있습니다.

Brightness
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=brightness" frameborder="0" allowfullscreen></iframe>
</div>

Grayscale
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=grayscale" frameborder="0" allowfullscreen></iframe>
</div>

Invert
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=invert" frameborder="0" allowfullscreen></iframe>
</div>

Black & White
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=bnw" frameborder="0" allowfullscreen></iframe>
</div>

Contrast
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=contrast" frameborder="0" allowfullscreen></iframe>
</div>

Saturation
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=saturation" frameborder="0" allowfullscreen></iframe>
</div>


[가산혼합]: http://ko.wikipedia.org/wiki/RGB_%EA%B0%80%EC%82%B0%ED%98%BC%ED%95%A9
[빛의 삼원색]: http://ko.wikipedia.org/wiki/%EC%9B%90%EC%83%89
[ImageData]: https://developer.mozilla.org/en-US/docs/Web/API/ImageData
[TypedArray]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays