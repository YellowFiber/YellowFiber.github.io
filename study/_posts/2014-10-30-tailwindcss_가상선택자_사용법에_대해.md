---
layout: post
title: tailwindcss 가상선택자에 대한 고찰
sitemap: false
tags: [css]
---

tailwind를 얼마 써보지 못한 상태에서 작성한 글입니다.<br>
혹시나 틀린 부분이나 더 좋은 방법있으시면 말씀 부탁드립니다!
{:.note}

## Introduction

<img src="/assets/img/blog/241030.jpg">
예시이미지
{:.figcaption}
예시이미지 처럼 가상선택자를 사용하여 그라데이션을 깔아줄려고 합니다.<br>
tailwind는 가상선택자도 지원해줘서 사용할 때 `after:`, `before:`를 쓰고 사용할 요소를 입력 해주면 됩니다.<br>
하지만 코드가 길어져 편리하게 쓸 수 있는 방법이 있는지 찾아보고 정리해봤습니다.

<hr>

## 기존 방법

```html
<!-- file: `Test.html` -->
<ul class="p-10 box-border">
  <li class="relative w-[800px] h-auto overflow-hidden
  after:absolute after:top-0 after:left-0 after:w-full after:h-full
  after:bg-gradient-to-b after:from-transparent after:from-70% after:to-gray-950 rounded-3xl">
    <img src="test.jpg" alt="" class="w-full h-full" />
  </li>
</ul>
```

공식 문서에서 지원하는 방법입니다.<br>
가상요소를 쓰면 자동으로 `content:""`가 설정되어 있어서 따로 적어주지 않아도 되는게 편합니다.<br>
단점은 코드가 기네요. `li`태그 한곳에만 하면 상관없겠지만 여러개 있으면..<br>
그래서 다른 방법으로 해봤습니다.

<hr>

## 개선 방법

```javascript
//  file: `tailwind.config.js`
module.exports = {
  content: ["./src/**/*.{html,js}"],
  theme: {
    extend: {
    },
  },
  plugins: [
    function ({ addUtilities }) {
      addUtilities({
        ".after-gradient": {
          position: "relative",
          "&::after": {
            content: '""',
            position: "absolute",
            top: 0,
            left: 0,
            right: 0,
            bottom: 0,
            backgroundImage:
              "linear-gradient(to bottom, transparent 70%, black 100%)",
            zIndex: 1,
          },
        },
      });
    },
  ],
};

```

```html
<!-- file: `Test.html` -->
<ul class="p-10 box-border">
 <li class="after-gradient w-[800px] h-auto overflow-hidden">
   <img src="test.jpg" alt="" class="w-full h-full" />
 </li>
</ul>
```

제가 찾은 개선방법은 addUtilities를 사용해 커스텀 클래스를 만드는 것입니다.<br>
`.after-gradient` 클래스를 생성하고 안에 스타일들을 넣어주면 끝입니다.<br>
기존 방법에 비해 코드가 많이 줄어들어 보기 좋네요.

<hr>

## 개인적인 느낌

tailwind는 커스텀 클래스를 많이 사용하는 것을 권장하고 있지 않습니다.<br>
그렇게 쓰면 차라리 그냥 css를 쓰는게 나을테니까요.<br>
하지만 자주 쓰는 내용이라면 클래스로 묶어서 쓰는 편이 더 좋다고 생각합니다.<br>
실제로 중고나라 사이트에서 tailwind를 사용하는데 가상선택자를 클래스로 묶어서 사용하고 있고요.<br>
아직 개인으로 해서 정리가 안되는 부분이 있지만 팀 프로젝트에 사용할 때는 어떤걸 클래스로 지정할지 협의가 필요한거 같습니다.
