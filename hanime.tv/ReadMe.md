---
layout: default
---
## Hanime.tv 크롤링
### 목표
애니메이션 제목을 입력하면 해당 애니메이션의 타이틀, 장르, 줄거리, 제작사, 커버를 딕셔너리의 형태로 출력하는 함수를 구현한다.

### 필요한 패키지
* requests  
* BeautifulSoup  

### 구현방법
#### 패키지 불어오기
```python
import requests
from bs4 import BeautifulSoup
```
* requests는 http에 요청을 보낼 때 사용하는 패키지이며, 이를 통해 `https://hanime.tv/videos/hentai/(애니메이션 제목)` 의 html소스를 가져올 것이다. 이 때 사용할 방식은 GET요청을 사용한다.  
* BeautifulSoup은 파이썬을 사용하여 웹 크롤링을 할 때 주로 사용되는 패키지이며, requests를 이용해 얻은 html소스로부터 얻고자하는 정보를 추출해낸다.

#### 입력값 보정하기
입력값을 title이라고 하자. 보통 제목을 입력할 때 제목에 띄어쓰기가 있다면 사용자 입장에서는 대다수가 -대신에 띄어쓰기로 입력할 것이다. 예로 사용자가 dokidoki little ooyasan 2를 입력했다고 가정하자.  
그러면 title="dokidoki little ooyasan 2"이므로 요청할 주소는 다음과 같이 될 것이다.
```
https://hanime.tv/videos/hentai/dokidoki%20little%20ooyasan%202
```
그러나 hanime.tv에서 dokidoki little ooyasan 2 주소는 다음과 같다.
```
https://hanime.tv/videos/hentai/dokidoki-little-ooyasan-2
```
따라서 title="dokidoki-little-ooyasan-2"이여야만 정상적으로 동작할 것이라는 것을 알 수 있다. 이 문제는 간단하게 파이썬의 내장함수 `.replace()`를 이용해 다음과 같이 해결할 수 있다.
```python
title = title.replace(' ', '-')
input_url = 'https://hanime.tv/videos/hentai/'+title
```
이 때, `title.replace(' ', '-')`는 입력값 title에서 띄어쓰기를 -으로 바꿔준다.

#### 크롤링 방지 우회하기
저자는 처음에 실행할 때, goodbyeDPI를 실행함으로서 SNI 도청을 우회를 했으니 당연히 될 것이라 생각했었으나, 결과는 404를 출력했었다.
그래서 goodbyeDPI가 python에는 영향을 안미치는 것인가 싶었지만, 알고보니 hanime.tv에서 크롤링을 방지해서 실패했던 것이다.
다행히게도 hanime.tv의 크롤링 방지는 다음과 같이 헤더를 추가함으로서 우회할 수 있다.
```python
headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36'}
```

#### html 가져오기
```python
res = requests.get(input_url, headers=headers)
soup = BeautifulSoup(res.content, 'html.parser')
```
`requests.get(input_url, headers=headers)`를 이용하여 input_url에 get 요청을 보내고, `BeautifulSoup(res.content, 'html.parser')`를 이용하여 html 소스를 파싱한다.

#### 정보 추출하기
이제 남은 것은 정보를 추출하기만 하면 된다. `https://hanime.tv/videos/hentai/dokidoki-little-ooyasan-2`의 html 소스를 보면 다음과 같은 내용이 있을 것이다.
```html
<div class="action-bar flex column">
    <div class="htv-video-page-action-bar flex wrap">
        <div class="title-views flex column">
            <h1 class="tv-title">Dokidoki Little Ooyasan 2</h1>
            <div class="tv-views grey--text">1,629,353 views</div>
        </div>
        <div class="actions flex row justify-right align-right"><span class="tooltip tooltip--top"><div class="tooltip__content" style="left:0px;max-width:auto;opacity:0;top:12px;z-index:0;display:none;"> <span>Like</span></div><span><div class="hvpab-btn flex justify-center align-center primary-color-hover"><i aria-hidden="true" class="icon mdi mdi-heart grey--text"></i> <span class="hvpabb-text ">6K</span></div>
    </span>
```
그런데 감사하게도 제목은 쉽게 추출해낼 수 있을 것 같다.  
왜냐하면 h1태그의 tv-title 클래스에 해당되는 값이 `<h1 class="tv-title">Dokidoki Little Ooyasan 2</h1>` 뿐이기 때문인데, 만약에 다른 요소도 있었다면, 적절히 태그와 클래스를 선택해야만 했을 것이다.
태그와 클래스를 이용하여 정보를 추출하는 것은 다음과 같이 할 수 있다.
```python
title = soup.find('h1', attrs={'class': 'tv-title'}).text
```
우선 soup라는 객체에는 html을 파싱한 내용물이 있을 것인데, `soup.find('h1', attrs={'class': 'tv-title'})`를 이용하여 `<h1 class="tv-title">Dokidoki Little Ooyasan 2</h1>`를 추출해낸다. 그리고 이 코드에서 `Dokidoki Little Ooyasan 2`만을 추출해내기 위해서 .text를 이용하면 해당 애니메이션의 제목을 얻어낼 수 있다.  

| |태그|클래스|  
|------|---|---|   
|Title|h1|tv-title|  
|Brand|a|hvpimbc-text|  
|Description|div|mt-3 mb-0 hvpist-description|  
|Tags Parent|div|hvpi-summary|  
|Tags|a| |  
|Cover Parent|div|hvpi-cover-container|  
|Cover|img| |  

위 표를 보면 Parent가 있는 경우가 있다. 이 경우는 아래와 같은 경우를 표시한 것이다.
```html
<div class="hvpi-summary">
    <div class="hvpis-text grey--text text--lighten-1">
        <a href="/browse/tags/plot" rel="nofollow" alt="Explore more videos that have the plot tag." title="Explore more videos that have the plot tag." class="ml-0 mr-3 btn btn--outline btn--depressed btn--router grey--text">
            <div class="btn__content">
                plot
            </div>
        </a>
        <a href="/browse/tags/ahegao" rel="nofollow" alt="Explore more videos that have the ahegao tag." title="Explore more videos that have the ahegao tag." class="ml-0 mr-3 btn btn--outline btn--depressed btn--router grey--text">
            <div class="btn__content">
                ahegao
            </div>
        </a>
        <a href="/browse/tags/hand job" rel="nofollow" alt="Explore more videos that have the hand job tag." title="Explore more videos that have the hand job tag." class="ml-0 mr-3 btn btn--outline btn--depressed btn--router grey--text">
            <div class="btn__content">
                hand job
            </div>
        </a>
        <a href="/browse/tags/censored" rel="nofollow" alt="Explore more videos that have the censored tag." title="Explore more videos that have the censored tag." class="ml-0 mr-3 btn btn--outline btn--depressed btn--router grey--text">
            <div class="btn__content">
                censored
            </div>
        </a>
        <a href="/browse/tags/creampie" rel="nofollow" alt="Explore more videos that have the creampie tag." title="Explore more videos that have the creampie tag." class="ml-0 mr-3 btn btn--outline btn--depressed btn--router grey--text">
            <div class="btn__content">
                creampie
            </div>
        </a>
        <a href="/browse/tags/hd" rel="nofollow" alt="Explore more videos that have the hd tag." title="Explore more videos that have the hd tag." class="ml-0 mr-3 btn btn--outline btn--depressed btn--router grey--text">
            <div class="btn__content">
                hd
            </div>
        </a>
        <a href="/browse/tags/loli" rel="nofollow" alt="Explore more videos that have the loli tag." title="Explore more videos that have the loli tag." class="ml-0 mr-3 btn btn--outline btn--depressed btn--router grey--text">
            <div class="btn__content">
                loli
            </div>
        </a>
        <div class="mt-3 mb-0 hvpist-description">
            Miyuri is a landlord who has sex with the tenant of one of her beat-up apartments, Daisuke. Miyuri ends up borrowing Daisuke’s shower and has sex with him in her sexy bathrobe! Miyuri uses her slutty hip thrusts to squeeze everything out from Daisuke! Getting all sweaty, they take a shower together. “My pussy is going to be full of all your cum at this rate!” Miyuri becomes a body sponge and washes up, down, back, front, every single corner! In return, Daisuke washes her as well! His fat, hard dick cleans every inch of her pussy!
        </div>
    </div>
```
위 소스를 보면 장르의 태그는 a이고 상위 클래스가 hvpi-summary라는 것을 알 수 있다. 따라서 장르를 추출할 때는 다음과 같이 할 수 있다.
```python
soup = soup.find('div', attrs={'class': "hvpi-summary"})
tags_a = soup.find_all('a')
```
그러나 문제는 이렇게 얻은 장르 데이터를 어떻게 쓸 수 있는가인데, 왜냐하면 soup.find_all('a')는 List Iterator이기 때문에 바로 사용하는 것이 불가능하기 때문이다. 따라서 자유로운 조작을 위해 다음과 같이 작성해야 한다.
```python
tags = []
for i in tags_a:
    tags.append(i.text)
```
위와 같이 작성하면, soup.find_all('a')의 결과물이 List Iterator이며 각 요소들이 html형식이라는 점을 해결할 수 있다.
마지막으로 커버 이미지를 추출해보자. 커버에 대한 정보는 다음과 같다.
```html
<div class="hvpi-cover-container">
    <img src="https://i0.wp.com/htvassets.club/images/covers/dokidoki-little-ooyasan-2.png?quality=100" alt="Dokidoki Little Ooyasan 2 dvd blu-ray video cover art" class="hvpi-cover">
</div>
```
따라서 다음과 같이 이미지를 추출할 수 있다.
```python
soup = BeautifulSoup(res.content, 'html.parser')
cover_div = soup.find('div', attrs={'class': "hvpi-cover-container"})
cover_img = cover_div.find('img')
cover = cover_img.get('src')
```
위의 코드를 실행하면, cover="https://i0.wp.com/htvassets.club/images/covers/dokidoki-little-ooyasan-2.png?quality=100"이 된다.

### 소스코드
```python
import requests
from bs4 import BeautifulSoup


def hanime(title):
    title = title.replace(' ', '-')
    input_url = 'https://hanime.tv/videos/hentai/'+title
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36'}
    res = requests.get(input_url, headers=headers)
    soup = BeautifulSoup(res.content, 'html.parser')
    cover_div = soup.find('div', attrs={'class': "hvpi-cover-container"})
    cover_img = cover_div.find('img')
    cover = cover_img.get('src')
    title = soup.find('h1', attrs={'class': 'tv-title'}).text
    brand = soup.find('a', attrs={'class': "hvpimbc-text"}).text
    desc = soup.find('div', attrs={'class': 'mt-3 mb-0 hvpist-description'}).text
    desc = desc.replace('\n', '')
    soup = soup.find('div', attrs={'class': "hvpi-summary"})
    tags_a = soup.find_all('a')
    tags = []
    for i in tags_a:
        tags.append(i.text)
    output_dict = {
        'title': title,
        'desc': desc,
        'tags': tags,
        'brand': brand,
        'cover': cover
    }
    return output_dict


print(hanime("a size classmate 2")) # 취향입니다. 존중해주세요. 8ㅁ8
```
