---
title:  "bufferGeometry 사용 시 buffer memory 부족 문제"
excerpt: "bufferGeometry로 원 6만개 그리기"
toc: true
toc_sticky: true
header:
  teaser: /assets/images/bio-photo-keyboard-teaser.jpg

categories:
  - Blog
tags:
  - threejs
last_modified_at: 
---

# 문제 상황

1. bufferGeometry를 사용하여 사이즈가 애니메이션되는 원을 6만개 그렸다.
2. 동 페이지에 이외에도 애니메이션 요소와 bufferGeometry 다수 사용하였는데 1번이 가장 많은 연산을 한다.
3. cpu i7, memory 16G 인 본인 컴퓨터에서는 잘 돌아간다.
4. 안돌아가는 컴퓨터가 있는데 다른 요소는 다 나오고 1번만 안 나오는데 cpu, memory 사양은 비슷하다.
5. 에러 메세지는 vertex buffer is not big enough for draw call.

# 해결 방안

### buffer memory가 부족하니까 buffer memory 늘리기
  - buffer memory를 늘리는 법 WebGLRenderer의 capabilities 속성값을 이리저리 조정해보았지만 실패.

### buffer memory를 적게 쓰던지

 -  요소를 나눠서 그린다: 6만개를 한번에 만들지 않고 나눠서 그리면 한번에 buffer memory가 overflow되지 않을 것같다.
 
 - **shader를 다시 짠다.**
    - 현재 하나의 shader를 두가지의 buffer memory에 나눠 쓰고 있었는데 이것을 분리 한다.
    - 분리하는 효과 : 연산 횟수를 줄인다.
    - 현재는 position(4), opacity(1), color(3), size(1)를 넘기는데 * 괄호 안 숫자는 1개 점당 넘기는 데이터 수
    - 1번 bufferGeometry는 position, color, size만 사용하므로 opacity는 모든 점이 동일하고 다른 bufferGeometry는 size를 사용하지 않아서 두개의 shader를 만들어서 각각 필요한 속성값만 전달하기로하였다.
    - 디자인이 변경될 여지가 있기 때문에 모든 옵션이 수정되게 남겨 놓는게 좋을 것 같다고 초반에 판단 했었다.

# 결론

- shader를 분리하는 방법으로 문제는 해결되었다. 하지만 더 낮은 사양의 컴퓨터가 있을지 모르기 때문에 근본적인 문제를 해결했다고 보기는 어렵다.
- 항상 브라우저나 운영체제만 생각했었는데 3D를 웹에서 서비스 하기 위해서는 하드웨어의 변수에 대해 생각할 필요가 있다는 것을 알았고 모든 컴퓨터에서 제공할 수 있는 서비스를 만드는 것이 가장 좋겠지만, 요구사항은 늘 끝이 없으니 서비스 할 수 있는 PC의 사양을 정할 필요가 있겠다. 
- IE에서는 chrom에서 작동하는 PC에서도 buffer memory가 부족하다는 에러 메세지가 뜨는데 각 브라우저의 GPU 사용 방식의 차이를 확인해봐야 겠다.
