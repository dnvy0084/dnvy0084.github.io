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

만약 그림처럼 $\vec{N}$을 알 수 있어서, $\vec{A}$에 $\vec{N}$을 두번 빼준다면 $\vec{R}$을 얻을 수 있을것 같습니다. 식으로 쓰면 $\vec{R} = \vec{A} - 2 \cdot \vec{N}$ 정도가 되겠네요. 여기서 $\vec{N}$은 반사된 벽을 $90$도 돌려서 각도를 구하고, $\vec{A}$를 $\vec{N}$축에 내적하여 길이를 구할 수 있습니다. 


[Canvas example][6]
======
<iframe width="100%" height="500" src="http://dnvy0084.github.io/example/vector-reflection.html" frameborder="0" allowfullscreen></iframe>

[*소스보기*][7]

[-wiki]: http://ko.wikipedia.org/wiki/%EB%B2%A1%ED%84%B0

[1]: /raw/rvec01.jpg "Vector"
[2]: /raw/rvec02.jpg "Vector"
[6]: {{ site.url }}example/vector-reflection.html
[7]: https://github.com/dnvy0084/math/blob/master/vector-reflection/vector-reflection.html