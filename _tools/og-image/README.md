# OG 이미지 생성

`assets/images/og-image.png`(1200×630) 생성용 소스.

## 재생성 방법

1. 저장소 루트에서 정적 서버 실행:

   ```sh
   python3 -m http.server 8899
   ```

2. 브라우저(또는 Playwright)에서 뷰포트를 **1200×630**으로 맞추고
   `http://localhost:8899/_tools/og-image/index.html` 접속

3. 뷰포트 전체를 PNG로 캡처해 `assets/images/og-image.png`에 저장

문구·색·배치는 `index.html`의 CSS에서 수정한다.
캐릭터 이미지는 `assets/images/profile.png`를 참조한다.
