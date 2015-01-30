---
layout: post
title:  "Hello World"
date:   2015-01-28 18:51:08
categories: update
---
[**Github Pages**][github-pages] 서비스와 [**Jekyll**][jekyll]을 이용한 블로그 포스팅입니다. 

Github는 git 호스팅 뿐만 아니라 소셜 기능이 많아 개발자들의 **_sns_** 라고 불릴 만큼 많은 개발자들이 활동하고 있습니다. 그 중 pages는 github에서 제공하는 웹사이트 호스팅 서비스로 개인용 혹은 프로젝트용으로 웹사이트를 개설할 수 있는데요, 재밌는건 github repository에서 바로 호스팅을 하기에 웹사이트 소스를 git으로 관리해야 합니다. 

다만 프로젝트 용 pages는 repository setting에서 자동 생성 기능을 이용해 README.md 파일을 불러 올 수 있으며 준비된 template 선택만으로 간단하게 만들 수 있습니다. 사용자 용 pages는 [__Markdown__][markdown] 텍스트를 웹사이트나 블로그로 변환해주는 tool인 jekyll을 이용해 쉽고 빠르게 생성할 수 있습니다.

jekyll은 기본적인 markdown 형식이나 code snippet 뿐만 아니라 

* jekyll code snippet - javascript
{% highlight javascript %}
(function(){
  //some self invoked anonymous function
  for( var i = 0; i < len; i++ )
  {
    console.log( i, colorBuffer[i] );
  };

  function test( a )
  {
    /** code **/
  }
})()
{% endhighlight %}

* jekyll code snippet - python

```python
def test( a ):
  print a
  arr = range( 10 )

  for i, j in enumerate( arr ):
    print i, j
```
* jekyll code snippet - c / c++


```c
void main( int argc, char **argv )
{
  for( int i = 0; i < len; i++ )
  {
    printf( "%d", i );
  }

  return 0;
}
```

mathjax 라이브러리를 이용하면 수학 공식이나 방정식도 보기 좋게 표현 할 수 있습니다.

$$(y-ay) = m( x - ax )$$
\\[y = mx - ma_x+ a _y\\]
$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$

[mathjax cheat sheet][1]


호스팅 했을 때 기본 도메인은 _http://아이디.github.io/_ 혹은 _http://아이디.github.io/프로젝트명_
 이지만 [**custom domain**][gh-help-customdomain]을 이용하면 자신만의 도메인을 부여하는 것도 가능합니다.

[jekyll]:      http://jekyllrb.com
[github-pages]: https://pages.github.com/
[markdown]: http://daringfireball.net/projects/markdown/
[gh-help-customdomain]: https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/
[1]: http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference