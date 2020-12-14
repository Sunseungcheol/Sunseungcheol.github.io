---
layout: post
categories: javaScript
tags: [확장프로그램]
---

블로그에 공부한 것을 정리하는데 코드정리를 위해 깔아놓은 prettier이 마크다운 문서도 건드려 상당히 귀찮았다.
여기저기 검색해 본 결과 루트디렉토리에 .prettierignore 파일을 만든 후 *.md 를 적어주면 된다는데..안됐다..
prettier 자체 설정에 Prettier: Disable Languages 에서 .md md Markdown 다 넣어도 안되서 한참을 헤맸는데 prettier 내역을 출력해주는 부분에 "parser": "markdown" 으로 되있어서 혹시나 하고 넣어보니 잘된다..