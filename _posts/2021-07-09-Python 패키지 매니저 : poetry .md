---
layout: post
title: "Python Web Framework 차이점: Django, Flask, Fastapi"
# cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/poetry.jpg
# share-img: /assets/img/path.jpg
tags: [web, flask, django]
comments: true
---



# Poetry

 `poetry`는 파이썬 의존성 패키지 관리 툴입니다. `poetry.lock`을 사용해서 프로젝트의 의존성을 다른 환경에서도 동일하게 유지할 수 있습니다.

 또한 `build`와 `publish`까지 지원해주고 있어서, 실제로 프로젝트를 만들고 저장소에 배포까지 하고자 하는 사람에게 유용합니다.

<a text-align="center" href="https://github.com/python-poetry">

​	<img href="../assets/img/poetry_github.png"/>

</a>

깃헙의 [python-poetry](https://github.com/python-poetry) 라는 계정에서 관리되는데, poetry 하나를 위해서 따로 만든 프로젝트가 있을 정도로 꽤나 정성스럽게 만든 프로젝트입니다.

또한 사용법도 기존에 npm을 사용해 본적이 있다면 아주 친숙한 명령어들로 구성되어 있습니다.

단점이라면 새로 커맨드를 익혀야 하는 문제가 있으며, 기존의 프로젝트에 적용하기에는 약간 부담스러울 수 있다는 것입니다.

개인적으로는 새로만드는 프로젝트라면 주저없이 `poetry`를 써보라고 추천하고 싶습니다.

본 문서는 `poetry`의 설치 및 기본적인 사용법을 정리해서 조금이라도 많은 분이 좋은 프로젝트 환경을 구축하도록 돕기위한 글입니다. 시간이 지나면 본 문서의 내용과 달라질 수 있으므로 최대한 [공식 사이트의 문서](https://python-poetry.org/docs) 를 참고하도록 합시다.
