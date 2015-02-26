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

Canvas는 [ImageData]라는 객체를 이용해 픽셀 조작을 할 수 있는데요, ImageData의 data라는 변수에 [TypedArray]인 [Uint8ClampedArray]형태로 저장되어 있습니다. TypedArray는 지정된 타입의 변수만 저장 가능한 strict한 배열입니다. as의 Vector.\<T\>와 같다고 볼 수 있으며, Uint8ClampedArray는 Vector.\<Uint8\> 으로 가정할 수 있는데, 당연히 as에는 Uint8 타입이 없습니다. C로 보면 unsigned char data[] 정도가 되겟네요. 

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

아래는 ImageData를 이용한 pixel manipulation 예제 입니다. 이미지 위에서 죄우로 드래그하면 값을 변경 할 수 있습니다. 

Brightness
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=brightness" frameborder="0" allowfullscreen></iframe>
</div>

```javascript
/**
* src: 소스 픽셀 버퍼( Uint8ClampedArray )
* dest: 대상 픽셀 버퍼( Uint8ClampedArray )
* value: 변경 값
**/
function manipulate( src, dest, value ) 
{
  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    dest[i  ] = src[i  ] + value;
    dest[i+1] = src[i+1] + value;
    dest[i+2] = src[i+2] + value;
  }
}
```
밝기는 단순하게 $+, -$만 해주면 됩니다. clamped라 $(0<=pixel<=255)$를 체크할 필요도 없습니다. 

Grayscale
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=grayscale" frameborder="0" allowfullscreen></iframe>
</div>

```javascript
/**
* src: 소스 픽셀 버퍼( Uint8ClampedArray )
* dest: 대상 픽셀 버퍼( Uint8ClampedArray )
* value: 변경 값
**/
function manipulate( src, dest, value ) 
{
  var r, g, b, v; // r,g,b는 rgb값 v는 계산된 grayscale

  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    r = src[i], g = src[i+1], b = src[i+2];
    v = 0.21 * r + 0.72 * g + 0.07 * b;

    dest[i  ] = r + value * ( v - r ),
    dest[i+1] = g + value * ( v - g ),
    dest[i+2] = b + value * ( v - b );
  }
}
```
grayscale은 rgb 3개 원소의 평균값입니다. ${(r + g + b) \over 3}$로 구하면 될 것 같지만, 사람의 망막은 green을 느끼는 세포가 많다고 하네요. 그래서 각각 $r = 0.21, g = 0.72, b = 0.07$의 $factor$를 곱해주어 계산합니다. 

Invert
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=invert" frameborder="0" allowfullscreen></iframe>
</div>

```javascript
/**
* src: 소스 픽셀 버퍼( Uint8ClampedArray )
* dest: 대상 픽셀 버퍼( Uint8ClampedArray )
* value: 변경 값
**/
function manipulate( src, dest, value ) 
{
  var r, g, b;

  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    r = src[i], g = src[i+1], b = src[i+2];

    dest[i  ] = r + value * ( 255 - r - r ),
    dest[i+1] = g + value * ( 255 - g - g ),
    dest[i+2] = b + value * ( 255 - b - b );
  }
}
```

invert는 색을 반전 시킵니다. 간단하게 $255 - pixel$로 계산됩니다. 다만 위 코드에서 

$pixel = pixel + value * ( (255 - pixel) - pixel )$로 

$p = a + t * ( b - a )$처럼 내분점( 무게중심 )을 이용하고 있는데요, 

$pixel = pixel + value * 255 - 2 * pixel * value$로 풀어서 

$pixel = pixel * ( 1 - 2 * value ) + 255 * value$ 정리하면 

$( 1 - 2 * value )$와 $255 * value$는 value가 매개변수이기 때문에 for loop 바깥에서 계산할 수 있습니다. 

```javascript
/**
* src: 소스 픽셀 버퍼( Uint8ClampedArray )
* dest: 대상 픽셀 버퍼( Uint8ClampedArray )
* value: 변경 값
**/
function manipulate( src, dest, value ) 
{
  var v = 255 * value, t = 1 - 2 * value;

  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    dest[i  ] = t * src[i  ] + v;
    dest[i+1] = t * src[i+1] + v;
    dest[i+2] = t * src[i+2] + v;
  }
}
```

다음과 같이 바꿀 수 있는데요, 연산이 조금 줄어들었습니다. for loop에서 식이 복잡하거나 길때는 전개했다가 i값 증감과 상관없는 계산은 밖으로 빼는게 성능 최적화의 한 방법입니다. 

Black & White
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=bnw" frameborder="0" allowfullscreen></iframe>
</div>

```javascript
/**
* src: 소스 픽셀 버퍼( Uint8ClampedArray )
* dest: 대상 픽셀 버퍼( Uint8ClampedArray )
* value: 변경 값
**/
function manipulate( src, dest, value ) 
{
  var r, g, b, v;

  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    r = src[i], g = src[i+1], b = src[i+2];
    v = Math.round(( 0.21 * r + 0.72 * g + 0.07 * b ) / 255) * 255;

    dest[i  ] = r + value * ( v - r ),
    dest[i+1] = g + value * ( v - g ),
    dest[i+2] = b + value * ( v - b );
  }
}
```

흑백 이미지는 grayscale + contrast라고 생각할 수 있습니다. 픽셀의 grayscale값이 128 이상이면 255, 이하면 0 두가지 값만 가지도록 하면 됩니다. 

0~1로 만들어서 반올림한 후에 * 255로 0 or 255로 만들었는데요, 이럴때는 그냥 if문 쓰는게 훨씬 빠릅니다. 

```javascript
if( v < 128 ) v = 0
else          v = 255
```


Contrast
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=contratstWithLookupTable" frameborder="0" allowfullscreen></iframe>
</div>

```javascript
/**
* src: 소스 픽셀 버퍼( Uint8ClampedArray )
* dest: 대상 픽셀 버퍼( Uint8ClampedArray )
* value: 변경 값
**/
function manipulate( src, dest, value )
{
  var t = ( 1 - value ) * 128;

  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    dest[i  ] = value * src[i  ] + t;
    dest[i+1] = value * src[i+1] + t;
    dest[i+2] = value * src[i+2] + t;
  }
}
```

contrast는 밝은 픽셀은 더 밝게, 어두운 픽셀은 더 어둡게하면 됩니다. $p = (r,g,b)$일 때, $p - 128$로 -128에서 128까지의 범위로 옮기고 $p\times contrast$를 하면 contrast가 2일 경우 -256 에서 256까지 범위로 확장( scale )됩니다. 마지막으로 $p+128$ 해주면 원래 위치로 돌아오는데 0을 중점으로 확장했기 때문에 밝은 곳은 더 밝게, 어두운 곳은 더 어둡게 바뀝니다.

$new p = contrast \times ( p - 128 ) + 128$

$new p = contrast \times p - contrast \times 128 + 128$

$new p = contrast \times p + 128 \times ( 1 - contrast )$

$new p = contrast \times p + t, (t=128 \times ( 1 - contrast ))$

로 위의 소스처럼 정리 할 수 있습니다. 이렇게 정리하는 이유는 invert와 마찬가지로 최적화의 의미도 있지만, 여러개의 효과를 주기 위해서는 5x5 ColorMatrix를 사용해야 되는데, 첫번째 식은 matrix 곱으로 표현하기 힘들기 때문입니다. 

$p = a \times x + b$ 형태라면 matrix 곱으로 표현할 수 있습니다. 

여기서 더 최적화를 하자면, 특정 contrast에 대한 결과값은 0 -> 0, 1 -> 0 ... 128 -> x 등 각 단계마다 동일하며 전체 픽셀 변경전에 0~255까지 계산해 놓는다면( lookup table ) 실제 for loop에서는 lookup table 값만 가져다 넣을 수 있습니다. 

```javascript
/**
* 특정 contrast에 대한 0-255 lookup table 생성. 
**/
contrast_lookup = function( value )
{
  var table = new Uint8ClampedArray( 256 ),
      t = 128 * ( 1 - value );

  for( var i = 0; i < 256; i++ )
  {
    table[i] = i * value + t;
  }

  return table;
};

/**
* lookup table에서 값 세팅. 
**/
contratstWithLookupTable = function( src, dest, value )
{
  var table = contrast_lookup( value );

  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    dest[i  ] = table[ src[i  ] ];
    dest[i+1] = table[ src[i+1] ];
    dest[i+2] = table[ src[i+2] ];
  }
};

```


Saturation
===

<div>
  <iframe width="100%" height="210" src="http://dnvy0084.github.io/example/pixel-maipulation.html?mode=saturation" frameborder="0" allowfullscreen></iframe>
</div>

```javascript
/**
* src: 소스 픽셀 버퍼( Uint8ClampedArray )
* dest: 대상 픽셀 버퍼( Uint8ClampedArray )
* value: 변경 값
**/
function manipulate( src, dest, value ) 
{
  var r, g, b,
      sr = ( 1 - value ) * 0.21, 
      sg = ( 1 - value ) * 0.72, 
      sb = ( 1 - value ) * 0.07;

  for( var i = 0, l = src.length; i < l; i+=4 )
  {
    r = src[i], g = src[i+1], b = src[i+2];

    dest[i  ] = r * ( sr + value ) + g * sg + b * sb;
    dest[i+1] = r * sr + g * ( sg + value ) + b * sb;
    dest[i+2] = r * sr + g * sg + b * ( sb + value );
  }
}
```

saturation은 grayscale factor를 사용합니다. 다만 각 원소마다 해당하는 원소에( r일때 r, g일때 g ... ) value만큼 더하여 색상을 더 진하게 표현합니다. 

[소스보기]


[가산혼합]: http://ko.wikipedia.org/wiki/RGB_%EA%B0%80%EC%82%B0%ED%98%BC%ED%95%A9
[빛의 삼원색]: http://ko.wikipedia.org/wiki/%EC%9B%90%EC%83%89
[ImageData]: https://developer.mozilla.org/en-US/docs/Web/API/ImageData
[TypedArray]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays
[Uint8ClampedArray]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray
[소스보기]: https://github.com/dnvy0084/dnvy0084.github.io/blob/master/example/pixel-maipulation.html