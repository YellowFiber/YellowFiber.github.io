---
layout: post
title: MongoDB로 todoList 만들기
sitemap: false
tags: [js, mongo, node]
categories: [devlog]
---

1. this ordered seed list will be replaced by the toc
{:toc}

## Introduction

<img src="/assets/img/blog/241129.gif">
<br>
MongoDB를 배우면서 실제로는 어떻게 사용되는지 알고싶어서 todolist를 만들어 봤습니다.


## 폴더 구조

```
project/
├── node_modules/
├── public/
│   ├── index.html
│   ├── main.css
│   ├── main.js
├── routes/
│   ├── list.js
├── config/
│   ├── db.js
├── .env
├── .gitignore
├── package.json
├── app.js

```

- `routes/list.js` : 할 일 목록 관련 작업을 처리하는 라우터 파일
- `config/db.js` : MongoDB 연결을 설정하는 파일
- `app.js` : express 서버 초기화하고 위의 모듈을 통합
- `public/main.js` : 클라이언트와 서버 간의 상호작용 처리
- `.env` : 환경변수(계정정보) 파일

<hr>

### db.js

```javascript
// file: `db.js`

require("dotenv").config();
const mongoose = require("mongoose");

const myId = process.env.MONGO_USER;
const myPassword = process.env.MONGO_PASSWORD;

const connectDB  = async () => {
  const mongoURI = `mongodb+srv://${myId}:${myPassword}@나머지주소`;
  try {
    await mongoose.connect(mongoURI, {
      dbName: "TodoList"
    });
  } catch(err){
    console.log('에러 발생 : ', err);
  }
}

module.exports = connectDB;

```

- MongoDB에 생성할 db로 `TodoList`를 만들어줬습니다.
- .env를 이용하여 제 계정정보가 나타나지 않도록 설정했습니다.<br>
<a href="/study/2024-11-28-texs/" target="_blank" title="새창열림">env에 대한 정리글</a>

<hr>

### list.js

```javascript
// file: `list.js`

const express = require("express");
const router = express.Router();
const mongoose = require("mongoose");

const listSchema = new mongoose.Schema({
  content: {
    type: String,
    required: true,
  },
  date: {
    type: Date,
    default: Date.now
  }
});
const List = mongoose.model("List", listSchema);

```

- 스키마 내용 정의
  - `content` : 할일 내용이 들어갈 데이터입니다. `required : true`를 넣어 필수로 들어가게 합니다.
  - `date` : 등록된 날짜가 나옵니다.
- `List` 모델로 스키마를 등록하여 데이터를 조작할 수 있게 합니다.
<br>

```javascript
// file: `list.js`

// 할 일 가져오기
router.get("/", async (req, res) => {
  try {
    const items = await List.find({});
    res.json(items);
  } catch (err) {
    res.status(500).send("불러오기 오류");
  }
});

// 할 일 추가
router.post("/", async (req, res) => {
  const { contents } = req.body;

  try {
    const newList = new List({ contents, date: new Date() });
    await newList.save();
    res.status(201).json(newList);
  } catch (err) {
    res.status(500).send("추가 오류");
  }
});

// 할 일 삭제
router.delete("/:id", async (req, res) => {
  const { id } = req.params;

  try {
    await List.findByIdAndDelete(id);
    res.status(200).send('선택한 할 일 삭제');
  } catch(err) {
    res.status(500).send('삭제 오류');
  }
});

module.exports = router;

```

- express router를 사용하여 조회, 추가, 삭제 기능을 구현했습니다.
- `status code` 정리
  - `200` : 성공적으로 처리
  - `201` : 요청이 성공적으로 처리 되어 리소스가 생성
  - `500` : 서버에 에러가 발생

<hr>

### main.js

```javascript
// file: `main.js`

const listForm = document.querySelector("#list-form");
const listInput = document.querySelector("#list-input");
const listWrap = document.querySelector("#list-wrap");

const LIST_URL = "/list";

async function fetchList() {
  try {
    const response = await fetch(LIST_URL);
    const lists = await response.json();
    renderList(lists);
  } catch (err) {
    console.log("오류 발생 : ", err);
  }
}

```
- `fetchList()`는 서버에 등록된 할 일들을 가져오는 함수입니다.
- `fetch(LIST_URL)`로 해당 주소에 등록된 데이터를 가져와 JSON 형식으로 반환됩니다.
- `response.json()`으로 자바스크립트 객체로 변환합니다.

<br>

```javascript
// file: `main.js`

function renderList(lists) {
  listWrap.innerHTML = "";
  lists.forEach((list) => {
    const li = document.createElement("li");

    const listDate = document.createElement('strong');
    listDate.classList.add('list-date');
    listDate.textContent = new Date(list.date).toLocaleDateString();

    const listContent = document.createElement('div');
    const listText = document.createElement('p');
    listText.classList.add('list-text');
    listText.textContent = list.contents;

    const deleteButton = document.createElement("button");
    deleteButton.textContent = "삭제";
    deleteButton.addEventListener("click", () => deleteList(list._id));

    listContent.appendChild(listText);
    listContent.appendChild(deleteButton);

    li.appendChild(listDate);
    li.appendChild(listContent);
    listWrap.appendChild(li);
  });
}

```

- `renderList(lists)`는 할 일을 등록할 때 사용되는 함수입니다.
- `new Date(list.date)`로 등록된 날짜 가져온 뒤 `toLocaleDateString()`으로 년도월일만 나오도록 했습니다.
- `deleteButton` 클릭시 list의 id를 확인하여 삭제하도록 구현했습니다.

<br>

```javascript
// file: `main.js`

listForm.addEventListener("submit", async (e) => {
  e.preventDefault();

  const listContents = listInput.value;

  try {
    const response = await fetch(LIST_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ contents: listContents }),
    });

    if (response.ok) {
      listInput.value = "";
      fetchList();
    }
  } catch (err) {
    console.log("에러 발생 : ", err);
  }
});

async function deleteList(id) {
  try {
    const response = await fetch(`${LIST_URL}/${id}`, {
      method: "DELETE",
    });

    if (response.ok) {
      fetchList();
    }
  } catch (err) {
    console.log("에러 발생 : ", err);
  }
}

fetchList();

```

- `listForm` 버튼을 클릭하면 입력한 내용이 할 일 목록에 등록이 됩니다.
- 입력한 내용은 자바스크립트 객체이므로 `JSON.stringify()`로 JSON 객체로 변환해서 서버에 넣습니다.
- `deleteList(id)`는 삭제 버튼을 클릭 시 실행되는 함수입니다.
  - 삭제할 목록의 주소와 아이디를 찾아서 실행합니다.

<hr>

### app.js

```javascript
// file: `app.js`

require("dotenv").config();
const express = require("express");
const path = require("path");
const connectDB = require("./config/db");
const ListRouter = require("./routes/list");

const app = express();
const PORT = 3000;

connectDB();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));

app.use('/list', ListRouter);

app.listen(PORT, () => {
  console.log('서버 실행');
})

```

- `app.js`는 서버 초기화·실행 설정, 환경변수 관리, 데이터베이스 연결, 미들웨어 설정, 라우터 등록하는 역할을 합니다.
- `app.use(express.static(path.join(__dirname, 'public')))` 는 `public` 폴더 안에 있는 파일들을 불러올 수 있도록 해줍니다.

<hr>

## 후기

"그냥 MongoDB로 연결하고 데이터 주고받으면 끝이겠지" 라고 생각했는데 그 과정이 쉽지않네요. 구글링 해보면서 여러 글을 읽고 만든 프로젝트라 아직 정리가 더 필요한거 같습니다. 그래도 제가 가장 알고 싶었던 `프론트엔드와 백엔드가 어떻게 데이터를 주고 받는가?` 에 대해 이해가 조금 되어서 좋았습니다. 만들고 나니 더 추가하고 싶은 기능도 있고 개선하고 싶은 부분도 있어서 조금씩 고쳐볼려고 합니다.