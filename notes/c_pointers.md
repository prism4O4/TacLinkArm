# 빌드로그 #1 — C 포인터 기초 (2026-10-01)

> TacLinkArm V1 · S1 1주차 · 분야: 기초 이론(C)
> 왜 배우나: 2주차 C++ 참조(`&`)를 배우기 전에 알아야 하고, Arduino 라이브러리 코드(`begin(&Wire)` 등)를 읽을 때 필요하다. 10주차 링버퍼·RMS 윈도 계산의 기반이기도 하다.

## 오늘 한 것
- 09-30: 주소 연산자 `&`와 역참조 연산자 `*` 이해
- 10-01: 포인터 연산(`p + 1`)과 배열·포인터의 관계(`arr[i] == *(arr + i)`)를 실행해서 확인, swap 함수로 값 전달과 포인터 전달 비교

## 코드 1 — `&`와 `*` (09-30)

```c
#include <stdio.h>

int main(void) {
    int number = 10;
    int *p = &number;      // &number: number의 주소

    printf("number 주소 = %p\n", (void*)&number);
    printf("p 값        = %p\n", (void*)p);   // 같은 주소
    printf("*p          = %d\n", *p);         // p가 가리키는 값 → 10

    *p = 20;               // 포인터로 원래 변수 값을 바꿈
    printf("number      = %d\n", number);     // 20
    return 0;
}
```

**한 줄 설명:** `&`는 변수의 주소를 꺼내고, 포인터 앞에 붙인 `*`는 그 주소에 있는 값을 꺼낸다.

## 코드 2 — 포인터 연산과 배열 (10-01)

```c
#include <stdio.h>

int main(void) {
    long arr[3] = {10, 20, 30};
    long *p = arr;                 // arr은 첫 원소의 주소 &arr[0]

    // 1. 타입 크기
    printf("[1] sizeof(long) = %zu\n\n", sizeof(long));

    // 2. p와 p+1의 주소
    printf("[2] p     = %p\n", (void*)p);
    printf("    p + 1 = %p\n", (void*)(p + 1));
    printf("    주소 차이 = %td 바이트\n\n", (char*)(p + 1) - (char*)p);

    // 3. arr[i] 와 *(arr+i) 비교
    printf("[3] i | arr[i] | *(arr+i) | *(p+i) | &arr[i]\n");
    for (int i = 0; i < 3; i++) {
        printf("    %d |   %ld   |    %ld    |   %ld   | %p\n",
               i, arr[i], *(arr + i), *(p + i), (void*)&arr[i]);
    }
    return 0;
}
```

실행 결과 (64비트 Linux gcc 기준. 주소값은 실행할 때마다 다름):

```
[1] sizeof(long) = 8

[2] p     = 0x7ffe31287880
    p + 1 = 0x7ffe31287888
    주소 차이 = 8 바이트

[3] i | arr[i] | *(arr+i) | *(p+i) | &arr[i]
    0 |   10   |    10    |   10   | 0x7ffe31287880
    1 |   20   |    20    |   20   | 0x7ffe31287888
    2 |   30   |    30    |   30   | 0x7ffe31287890
```

**한 줄 설명:** 포인터에 1을 더하면(`p + 1`, `p++`) 주소가 1바이트가 아니라 가리키는 자료형 크기(`sizeof(*p)`)만큼 이동한다. 그래서 `arr[i]`, `*(arr + i)`, `*(p + i)`는 모두 같은 값이다.

## 코드 3 — swap: 값 전달 vs 포인터 전달 (10-01)

```c
#include <stdio.h>

void swap_value(int a, int b) {      // a, b는 x, y의 복사본
    int t = a; a = b; b = t;         // 복사본끼리만 바뀜
}

void swap_ptr(int *a, int *b) {      // a, b는 x, y의 주소
    int t = *a; *a = *b; *b = t;     // 그 주소에 있는 원본을 바꿈
}

int main(void) {
    int x = 1, y = 2;

    swap_value(x, y);
    printf("값 전달 후:     x=%d, y=%d\n", x, y);   // x=1, y=2 (그대로)

    swap_ptr(&x, &y);
    printf("포인터 전달 후: x=%d, y=%d\n", x, y);   // x=2, y=1 (바뀜)
    return 0;
}
```

**한 줄 설명:** 값으로 넘기면 함수는 복사본을 받기 때문에 원본이 그대로이고, 주소로 넘기면 함수가 그 주소로 찾아가 원본을 직접 바꾼다.

비유: 값 전달은 서류의 **복사본**을 건네는 것이다. 내용은 똑같이 다 들어 있지만 복사본에 낙서해도 원본은 그대로다. 포인터 전달은 원본이 있는 **집 주소**를 건네는 것이라, 그 주소로 찾아가 원본을 직접 고칠 수 있다.

## 오늘 알게 된 것·고친 것
- 처음 코드에서는 `long` 값을 `%d`로 출력했다. `long`은 `%ld`로 출력해야 한다(`gcc -Wall`로 컴파일하면 경고가 나온다). 첫 줄 라벨도 `sizeof(int)`에서 `sizeof(long)`으로 고쳤다.
- `long`의 크기는 환경마다 다르다. 64비트 Linux에서는 8바이트이고, Windows에서는 4바이트다. Arduino Nano(AVR)에서는 `int`가 2바이트, `long`이 4바이트다. → MCU 코드에서는 크기가 정해진 `int16_t`, `uint32_t` 같은 타입을 쓰는 이유.

## 아직 모르는 것 / 다음 할 일
- 스택과 힙, MCU(SRAM 2KB)에서 `malloc` 대신 정적 할당을 쓰는 이유 → 10-02 AI 튜터 대화에서 정리
- `arr`과 `&arr`은 값(주소)이 같은데 무엇이 다른가?