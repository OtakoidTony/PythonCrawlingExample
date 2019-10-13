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
