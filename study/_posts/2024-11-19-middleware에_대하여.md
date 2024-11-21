---
layout: post
title: middleware에 대하여
sitemap: false
tags: [node]
categories: [study]
---

1. this ordered seed list will be replaced by the toc
{:toc}

Express의 middleware가 잘 이해되지 않아서 정리한 글
{:.note}


## 📌Middlware란?

- req(요청), res(응답), next를 인자로 가지는 함수
- next는 다음 미들웨어가 작업을 처리할 수 있도록 순서를 넘김
- 미들웨어는 res, next(), next(err)중에 하나를 반드시 호출

<img src="/assets/img/blog/241120_middleware.jpg" alt="미들웨어 기본구조">
미들웨어 기본구조
{:.figcaption}

<hr>

## 📌app.use()

- 지정된 경로에 미들웨어 호출하는 역할
- url이 없어도 사용 가능

#### 경로가 없는 경우

```javascript

const app = express();

app.use((req, res, next) => {
  console.log('요청 실행');
  next();
});

```

#### 경로가 지정되어 있는 경우
```javascript

app.use('/user/:id', (req, res, next) => {
  console.log('요청 실행');
  next();
});

```

- `/user/:id` 경로에 대한 모든 유형의 http 요청이 실행된다.

<hr>

## 📌에러 핸들링

```javascript

app.use((err, req, res, next) => {
    console.err(err);
});

```

- 미들웨어로도 에러 처리가 가능하며 반복적인 에러 핸들링 처리를 위해 공통된 작업을 모듈화 하는 용도
- 4개의 인자를 받으며 인자를 다 쓰지않아도 꼭 적어야 한다.

#### 에러 핸들링 주의점

작성 할 때 에러 핸들링은 맨 밑에 작성해야 한다.

##### 에러 핸들링 위에 작성한 경우

```javascript

app.use(function(err,req, res, next) {
  console.log(2)
  console.error(err);
  res.status(500).send('오류 발생 2');
});

app.get('/',(req,res)=>{
  console.log(1);
  throw new Error('오류 발생 1')
  res.send("hi")
});

```
<img src="/assets/img/blog/241121_1.jpg">

- 위에 적었을 경우 터미널에는 1만 나오며 화면에 에러 메세지가 그대로 출력된다.

##### 에러 핸들링을 마지막에 작성한 경우
```javascript

app.get('/',(req,res)=>{
  console.log(1);
  throw new Error('오류 발생 1')
  res.send("hi")
});

app.use(function(err,req, res, next) {
  console.log(2)
  console.error(err);
  res.status(500).send('오류 발생 2');
});

```
<img src="/assets/img/blog/241121_2.jpg">
<img src="/assets/img/blog/241121_3.jpg">

- 마지막에 적으면 화면에는 `오류 발생 2`라는 문구만 출력이 되고 터미널에도 정상적으로 작동 확인

## 📖참고문헌
- <a href="https://expressjs.com/ko/guide/writing-middleware.html" target="_blank" title="새창열림">https://expressjs.com/ko/guide/writing-middleware.html</a>
- <a href="" target="_blank" title="새창열림">https://inpa.tistory.com/entry/EXPRESS-%F0%9F%93%9A-%EB%AF%B8%EB%93%A4%EC%9B%A8%EC%96%B4-%F0%9F%92%AF-%EC%9D%B4%ED%95%B4-%EC%A0%95%EB%A6%AC</a>
