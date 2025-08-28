---
layout: single
title: "[github.io] 블로그 구글 검색 등록하기(SEO 설정)"
date: 2023-03-12 15:00:00 +0900
last_modified_at: 2023-03-14 22:37:00:00 +0900
categories: blog
---

![search-result1](/assets/images/2023/03/12/search-result1.png)

현재 제 블로그를 구글에 검색하면 전혀 나오지 않습니다\...

원인은 구글 웹 크롤러가 제 블로그를 크롤링 하지 못하는 것인데요. 블로그가 일기장이 되기 전에 빠르게 구글에 등록해 보겠습니다.

<br/>

## Google Search Console 등록

먼저 [Google Search Console](https://search.google.com/search-console)에 접속해 줍니다.

![google-search-console-1](/assets/images/2023/03/12/google-search-console-1.png)

URL 접두어에 블로그 주소를 적어주세요!

![google-search-console-2](/assets/images/2023/03/12/google-search-console-2.png)

google ~~ .html 파일을 다운로드 받고 github repository에 업로드 해줍시다.

repo의 action에서 배포가 성공적으로 된걸 확인 후 확인 버튼을 누르면!

![google-search-console-3](/assets/images/2023/03/12/google-search-console-3.png)

소유권이 확인되었다고 합니다~ 그럼 속성으로 이동해 볼게요.

> 인증 후에도 repo에서 인증 html파일을 삭제하시면 안됩니다!!
> 혹시라도 삭제하거나 잃어버렸다면 [구글 웹마스터](https://www.google.com/webmasters/verification) 로 가서 자기 블로그 클릭 후 HTML 파일 상세정보 누르면 재 다운로드 가능하니 참고하세요!

![google-search-console-4](/assets/images/2023/03/12/google-search-console-4.png)

이제 `Search Console`에서 색인생성 -> Sitemaps로 들어가 새 사이트맵 추가를 해야합니다.

<br/>

## 사이트맵(sitemap.xml) 생성

사이트맵은 `sitemap.xml`로 google 인증 html 파일처럼 레포의 root 폴더에 바로 추가하시면 됩니다!

```xml
{% raw %}
---
layout: null
---
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9 http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd"
  xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  {% for post in site.posts %}
    <url>
      <loc>{{ site.url }}{{ post.url }}</loc>
      {% if post.last_modified_at == null %}
        <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>
      {% else %}
        <lastmod>{{ post.last_modified_at | date_to_xmlschema }}</lastmod>
      {% endif %}

      {% if post.sitemap.changefreq == null %}
        <changefreq>weekly</changefreq>
      {% else %}
        <changefreq>{{ post.sitemap.changefreq }}</changefreq>
      {% endif %}

      {% if post.sitemap.priority == null %}
          <priority>0.5</priority>
      {% else %}
        <priority>{{ post.sitemap.priority }}</priority>
      {% endif %}

    </url>
  {% endfor %}
</urlset>
{% endraw %}
```

위 코드로 `sitemap.xml`을 생성하여 github repo에 추가해주세요!

post들의 글을 전부 들고와 `url`과 `마지막 수정 날짜`, `changefreq`, `priority`를 추가해줍니다.

`jekyll`에서 수정 날짜는 `last_modified_at`으로 되어있기 때문에 `last_modified_at`으로 `lastmod`를 설정하였습니다.

- post의 `markdown`에 `last_modified_at`이 없으면 등록 날짜로 `lastmod`를 부여합니다.

- 마찬가지로 `sitemap.changefreq`와 `sitemap.priority`가 없으면 `default` 값을 부여합니다.

```md
---
layout: single
title: "[github.io] 블로그 구글 검색 등록하기(SEO 설정)"
date: 2023-03-12 15:00:00
last_modified_at: 2023-03-12 15:00:00
catagories: blog
sitemap:
  changefreq: weekly
  priority: 0.5
---
```

이 포스팅의 `Yaml front matter`입니다! `jekyll` 같은 정적 사이트 생성기에서 사용하는 메타데이터 형식으로 페이지 정보를 저장할 수 있습니다. 여기에 정의 된 값들로 sitemap.xml이 만들어집니다.

<br/>

## robots.txt 생성

sitemap.xml을 만들었다고 다가 아닙니다. `robots.txt`를 만들어서 웹 크롤러에게 무엇을 크롤링하게 할지 정해 줘야합니다.

```
User-agent: *
Allow: /
Sitemap: {{ '/sitemap.xml' | relative_url | prepend: site.url }}
```

- `User-agent` \*의 의미는 AdsBots을 제외한 모든 크롤러를 허용한다는 뜻입니다.
- `Allow`는 허용할 페이지 위치를 지정합니다. /를 입력하여 제 블로그 전체를 지정하였습니다.
- `Sitemap`에 `Liquid 필터`를 이용하여 `sitemap.xml`의 위치를 명시해주었습니다.

이제 robot.txt를 github repo의 root 위치에 추가해주세요!

<br/>

## 마지막 Google Search Console에 sitemap.xml 제출

마지막 단계입니다! `Search Console`에서 색인생성 -> Sitemaps을 다시 켜줍니다.

![success](/assets/images/2023/03/12/success.png)

sitemap.xml을 입력하고 제출해줍니다. 상태가 성공이 되면 성공입니다!

> 처음 등록시 가져올 수 없음이라고 나왔는데 사이트맵을 누르니 성공이라 나오더라고요. 다시 제출하니 바로 성공이라 뜨는게 단순 오류인거 같습니다.
> 혹시 가져올 수 없음이라 뜨시면 다시 제출 눌러보시길 바랍니다!

<br/>

## 진짜 성공\...?

검색 되는데 시간이 조금 걸린다고 합니다.

~~나중에 성공하게 되면 결과 사진 추가할게요!~~

![seo-success](/assets/images/2023/03/12/seo-success.png)

짠! 따로 제 블로그 이름인 jagaldol을 검색어에 넣지 않아도 제목 키워드를 잘 넣으니 상단에 검색 되는 모습을 볼 수 있습니다!

첫번째에 들어간다면 더할 나위 없지만 저 분이 더 깔끔하게 정리해서 올라갔나보네요. 포스팅하는 요령도 익혀야겠습니다.

구글 검색 등록 되는게 보통 1주일, 길면 2주까지도 간다고 합니다.

<br/>
