.# 비검사 경고를 제거하라
## 비검사 경고
>  컴파일러 경고 / unchecked warning

- 안전하다고 검증된 비검사 경고를 (숨기지 않고)그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다.
- 비검사unchecked는 Java 컴파일러가 type-safe를 보장하기 위해 필요한 타입 정보가 충분히 존재하지 않는다는 의미
## 결론
-  @SuppressWarnings("unchecked")을 붙이는 이유에 대해서 주석으로 설명 필수! 
- 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거! 
## 출처
https://github.com/Meet-Coder-Study/book-effective-java/blob/main/5%EC%9E%A5/27_%EB%B9%84%EA%B2%80%EC%82%AC_%EA%B2%BD%EA%B3%A0%EB%A5%BC_%EC%A0%9C%EA%B1%B0%ED%95%98%EB%9D%BC_%EB%B0%95%EA%B2%BD%EC%B2%A0.md