---
layout: post
title:  "Prime Number"
date:   2015-03-20 10:00:00
categories: update
---

[10001st Prime]
===

$[2,3,5,7,11,13 \cdots ]$의 소수 배열이 있을 때, 6번째 소수는 13입니다. 그럼 10001번째 소수는 무엇일까요? <small>([project euler]라는 알고리즘 사이트에 문제입니다.)</small> 

간단히 생각해보면, `bool isPrimeNum( unsigned int n )`이란 함수를 만들고 10001번째 소수가 발견될 때까지 for loop를 돌면 될 것 같습니다. 

```javascript

function isPrimeNum( n )
{
  // some code 
  return true | false;
}

var pcount = 0;

for( var i = 2; true; i++ )
{
  if( ! isPrimeNum( i ) ) continue;

  if( ++pcount == 10001 ) return i;
}
```

for loop는 위 코드 만으로도 된것 같으니 isPrimeNum을 만들어야 겠네요. 

소수란 1과 자기 자신을 제외한 약수가 없는 수를 말합니다. n을 나눴을 때 나머지가 0이 되는 수가 있다면 소수가 아니라고 할 수 있으니 return false 하면 됩니다.

```javascript
fucntion isPrimeNum( n )
{
  for( var i = 2; i < n; i++ )
  {
    if( n % i == 0 ) return false;  
  }

  return true;
}
```
위 코드는 for loop를 n까지 확인하고 있는데, $\sqrt{n}$ 보다 큰 수는 for loop 앞쪽에서 이미 return 했을테니 확인할 필요가 없습니다. 

가령 n이 20일 경우 약수는 $1, 2, 4, 5, 10, 20$인데 1과 20, 2와 10, 4와 5는 짝을 이루고 있기 때문에 앞수를 찾아내면(n % i == 0) 찾아낸 앞수로 나눠서(n / i) 뒤의 수를 알 수 있습니다. 그래서 for loop를 n이 아니라 $floor(\sqrt{n})$까지만 검사를 해도 소수 여부를 알 수 있습니다. <small>(n이 완전 제곱수일 경우도 있으니 비교는 $(<=)$여야 합니다.)</small>


```javascript

function isPrimeNum( n )
{
  var limit = parseInt( Math.sqrt(n) );

  for( var i = 2; i <= limit; i++ )
  {
    if( n % i == 0 ) return false;
  }

  return true;
}
```

위와 같이 바꿀 수 있습니다. 여기서 조금더 최적화를 하자면, 

```javascript
function isPrimeNum( n )
{
  // 1은 소수가 아니다.
  if( n == 1 )          return false;
  // 1을 제외한 4보다 작은 수는(2,3) 소수이다.
  else if( n < 4 )      return true;
  // 2를 제외한 2로 나누어 떨어지는 모든 수는 소수가 아니다. 
  else if( n % 2 == 0 ) return false;
  // 2, 3을 제외한 9보다 작은 수 중에 홀수는 모두 소수이다. 
  else if( n < 9 )      return true;
  // 9이상의 수중에 3으로 나누어 떨어지는 수는 소수가 아니다.
  else if( n % 3 == 0 ) return false;
  else
  {
    var limit = Math.floor( Math.sqrt( n ) );

    for( var i = 5; i < limit; i += 6 )
    {
      if( n % i == 0 ) return false;
      if( n % ( i + 2 ) == 0 ) return false;
    }
  }

  return true;
}
```

코드가 길고 복잡해 졌는데요, for loop전에 if else로 걸러주는 이유는 5보다 큰 소수는 $6k \pm 1$로 나타낼 수 있어서 입니다. 

왜냐하면 2의 배수는 $4,6,8,10,12\cdots$ 짝수로 모두 소수가 아니며, 3의 배수 $3,6,9,12,15,18,21\cdots$에서 한번씩 건너 나오는 $9,15,21\cdots$의 홀수 또한 소수가 아닙니다. 그럼 남은 수는 $6,12,18\cdots$ 앞뒤의 홀수인 $(5,7),(11,13),(17,19)\cdots$인데 이 수들은 $6k\pm1$로 나타낼 수 있으며, 이 수들만 모듈러 연산을 통해 체크하면 소수 여부를 알 수 있습니다. 

[10001st Prime]: https://projecteuler.net/problem=7
[project euler]: https://projecteuler.net