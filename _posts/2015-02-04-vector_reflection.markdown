---
layout: post
title:  "Vector Reflection"
date:   2015-02-04 10:00:00
categories: update
---

# Vector Reflection

매 프레임당 어떤 물체의 $x, y$에 $vx, vy$를 더해주면 움직임을 만들 수 있습니다. 거기다 $vy = vy + G$ 해주면 그럴듯한 중력도 구현할 수 있습니다. 이러다 보면 물체가 지면이나 벽에 부딪혀 튕겨 나오는 경우도 있는데요, 그때 벡터를 이용하면 반사벡터를 쉽게 구할 수 있습니다. 

벡터는 흔히 **`크기와 방향을 가지는 단위이다`** 라고 정의하는데, 정작 $\vec{A}$ 라고 표기하며 $( x, y, z... )$ 처럼 숫자들로 이루어져 있습니다. 저 숫자 집합에서 크기와 방향을 알아내려면, 피타고라스 정의와 삼각함수를 이용한 계산이 따로 필요합니다. 그래서 단순히 숫자 집합이며 그게 2차원이라면 $(x,y)$, 3차원이라면 $(x,y,z)$를 원소로 갖는 단위$(Class)$라고 이해하는게 편합니다. 프로그래밍이라면 더욱 그렇겠죠. 

새로운 단위가 생기면 가장 처음으로 하는일은 사칙연산이 가능한가, 교환, 분배, 결합 법칙이 적용되느냐를 알아보는데 벡터는 더하기, 빼기는 같고 곱하기는 내적과 외적으로 나눠 적용되며 나누기는 의미가 없다고 하네요. 그리고 교환, 결합 법칙이 적용되며 항등원, 역원도 존재 합니다. [-wiki]

위에서 말한 $vx, vy$가 벡터로 표현될 수 있는데요, 연산을 통해 특정 각도에 대한 $vx, vy$의 반사 벡터를 계산 해보겠습니다.

![alt text][1]

그림과 같이 $\vec{A}$가 반사된 $\vec{R}$을 구하려면 면에 대한 입사각 $\theta$ 를 구해서 $2 \cdot \pi - \theta$로 반사각을 구하고 $\vec{A}$를 반사각 만큼 돌리면 될 것 같은데요, 물론 이런 방법으로도 구할 수 있겠지만, 각도 때문에 삼각함수를 써야 하니 성능이 좋을것 같지 않네요. 

![alt text][2]

만약 그림처럼 $\vec{N}$을 알 수 있어서, $\vec{A}$에 $\vec{N}$을 두번 빼준다면 $\vec{R}$을 얻을 수 있을것 같은데요. 식으로 표현하면 $\vec{R} = \vec{A} - 2 \cdot \vec{N}$ 가 되겠네요. 여기서 $\vec{N}$은 반사된 벽을 $90$도 돌려서 각도를 구하고, $\vec{A}$를 $\vec{N}$축에 내적하여 길이를 구할 수 있습니다. (3D라면 반사된 면을 이루는 두변을 외적하여 구할 수 있습니다.)

반사된 면을 90도 돌리려면, 우선 면의 끝 벡터 $(x1,y1)$에서 시작 벡터 $(x0,y0)$를 빼 벡터 $(x',y')$으로 만들고 $\theta$ 만큼 돌리기 위해 ![alt text][3] 를 곱해주면 되는데요, 90도(${\pi \over 2 }$)를 돌려야 하니 $\theta = {\pi \over 2}, \cos {\pi \over 2} = 0, \sin {\pi \over 2} = 1$ 이 되서 ![alt text][4]가 되고, 풀어보면 $(-y, x)$가 됩니다. 이렇게 90도 돌린 벡터를 법선 벡터( normal vector )라고 합니다.


```javascript
function Vector2D(x,y){};

// 벡터 빼기
Vector2D.prototype.sub = function( v )
{
	return new Vector2D( this.x - v.x, this.y - v.y );
}

// 노말( 법선 ) 벡터 구하기
Vector2D.prototype.normal = function()
{
	return new Vector2D( -y, x );
}
```

다음은 길이인데, 위에서 말씀드린것 처럼 $\vec{N}$축에 내적하여 길이를 구할 수 있는데요, 여기서 $\vec{N}$축은 $\vec{N}$의 단위 벡터를 가리킵니다. 길이가 1인 벡터를 단위 벡터라고 하는데요, 각 원소를 자신의 길이로 나눠주면 구할 수 있습니다.(정규화 한다고 합니다. ${\vec{V} \over ||\vec{V}|| }$)


```javascript
Vector2D.prototype.length = function()
{
	return Math.sqrt( this.x * this.x + this.y * this.y );
}

Vector2D.prototype.normalize = function()
{
	var len = this.length();

	return new Vector2D( this.x / len, this.y / len );
}
```

정규화 하여 $\vec{N}$축이 나오면 $\vec{A}\cdot \vec{N}$ 내적으로 scalar 값을 구하여 나온 값을 $\vec{N}$에 scalar 곱을 해주면 최종 $\vec{N}$를 구할 수 있습니다. 


```javascript
Vector2D.prototype.dot = function( v )
{
	return this.x * v.x + this.y * v.y;
}

Vector2D.prototype.scale = function( scalar )
{
	return new Vector2D( this.x * scalar, this.y * scalar );
}
```

reflection vector 구하는 코드 입니다. 

```javascript
function getReflectionVector( incidentVec, collisionPlane )
{
	var sub =  collisionPlane.b.sub( collisionPlane.a ),
		normal = sub.normal(),
		normalized = normal.normalize(),
		scalar = incidentVec.dot( normalized ),
		reflect = normalized.scale( 2 * scalar );

	return incidentVec.sub( reflect );
};
```


[Canvas example][6]
======
[*소스보기*][7]

<iframe width="100%" height="400" src="http://dnvy0084.github.io/example/vector-reflection.html" frameborder="0" allowfullscreen></iframe>


[-wiki]: http://ko.wikipedia.org/wiki/%EB%B2%A1%ED%84%B0

[1]: /raw/rvec01.jpg "Vector"
[2]: /raw/rvec02.jpg "Vector"
[3]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D%5Ccos%20%5Ctheta%20%26%20-%5Csin%20%5Ctheta%20%5C%5C%20%5Csin%20%5Ctheta%20%26%20%5Ccos%20%5Ctheta%20%5Cend%7Bbmatrix%7D%20%5Ccdot%20%5Cbegin%7Bbmatrix%7D%20x%20%5C%5C%20y%20%5Cend%7Bbmatrix%7D "rotate"
[4]: http://chart.apis.google.com/chart?cht=tx&chl=%5Cbegin%7Bbmatrix%7D0%20%26%20-1%20%5C%5C%201%20%26%200%20%5Cend%7Bbmatrix%7D%20%5Ccdot%20%5Cbegin%7Bbmatrix%7D%20x%20%5C%5C%20y%20%5Cend%7Bbmatrix%7D "90 rotate"

[6]: {{ site.url }}example/vector-reflection.html
[7]: https://github.com/dnvy0084/math/blob/master/vector-reflection/vector-reflection.html