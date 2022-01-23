# Embedded-C-Code-Optimization
#### 임베디드 시스템 개발 환경

- 네이티브 개발환경

  > 개발하는 시스템의 프로세서와 프로그램이 실행될 시스템의 프로세서가 같은 개발환경

- 크로스 컴파일 개발환경

  > - 임베디드 소프트웨어 개발환경은 개발하는 시스템의 프로세서와 실행될 시스템의 프로세서가 같지 않은 환경
  >
  > - 생성되는 개발 환경을 다른 컴파일러를 사용해야 한다는 뜻이 된다.
  > - 즉 임베디드 소프트웨어를 개발하려면 다른 프로세서용 기계코드를 생성해주는 컴파일러가 필요하며 이를 크로스 컴파일러라 한다. (arm-linux-gcc, ADC 등)
  > - 크로스 컴파일러, 크로스 어셈블러, 링커/로케이터 등으로 구성된 소프트웨어 개발도구를 "툴체인"이라 한다.

- 입출력 장치 제어 방법

  - 입출력 장치 제어 방법에는 두가지 방법이 있다.

  > - 메모리 맵I/O7
  >
  >   => 입출력 장치들이 사용하는 메모리가 따로 정해져 있지 않아서 장치가 사용할 메모리를 개발자가 직접 지정하여 사용하는 방법
  >
  > - I/O 맵 방법
  >
  >   => I/O는 입출력 장치들이 사용하는 메모리가 미리 정해져 있어서 지정된 메모리를 사용하는 방법

- 하드웨어 제어 동작 예시

  > LED 3000_0000번지의 5,6,7번 비트에 값을 쓰는 방법은 다음과 같다.
  >
  > ```c
  > char *p = (char*)0x30000000;
  > //5,6,7번 비트를 1로 set
  > *p |= (0x1<<5) + (0x1<<6) + (0x1<<7);
  > //5,6,7번 비트를 0으로 set
  > *p &= ~((0x1<<5) + (0x1<<6) + (0x1<<7));
  > //특정 비트 반전
  > *p &^= 0x1<<5;
  > ```

- 비트 연산을 통한 활용

  > 어떤 수에 대한 값을 더했을 때 가장 가까운 4의 배수에 해당하는 수를 만드려면 어떻게 해야하는가
  >
  > - 그냥 C code
  >
  >   ```c
  >   start = 100;
  >   scanf("%d", &x);
  >   start += x;
  >   if((start % 4) == 1)
  >       start += 3;
  >   else if ((start % 4) == 2)
  >       start += 2;
  >   else if((start % 4) == 3)
  >       start += 1;
  >   ```
  >
  > - 비트 연산을 사용한 C code
  >
  >   ```c
  >   start = 100;
  >   scanf("%d", &x);
  >   start += x;
  >   start += 0x3; //1에서 3을 더해도, 2에서 3을 더해도, 3에서 3을 더해도, 4에서 3을 더해도 
  >   				// 0bxxxxx...1xx 형태에 해당하는 비트 값을 가진다.
  >   start &= ~(0x3); // 0bxxxxx.....1xx & 0bxxxxxxx...100 의 값은 0b100이 되기에 4의 배수에 해당하는 값이 된다.
  >   ```

- 비트 연산을 활용한 매크로 정리

  > - 한 비트 클리어(예> 5번 비트)
  >
  >   `a &= ~(0x1<<5);`
  >
  > - 연속된 여러 비트 클리어(예> 5,4,3,번 비트)
  >
  >   `a &= ~(0x7<<3);`
  >
  > - 떨어져 있는 여러 비트 클리어 (예> 5,3,2번 비트)
  >
  >   `a &= ~((0x1<<5) + (0x3<<2));`
  >
  > 
  >
  > - 한 비트 설정(예> 5번 비트) 
  >
  >   `a |= (0x1<<5); `
  >
  > - 연속된 여러 비트 설정 (예> 5, 4, 3번 비트) 
  >
  >   `a |= (0x7<<3);`
  >
  > - 떨어져 있는 여러 비트 설정 (예> 5,3,2번 비트)
  >
  >   `a |= (0x1<<5) + (0x3<<2);`
  >
  > 
  >
  > - 한 비트 반전
  >
  >   `a ^= (0x1<<5);`
  >
  > - 연속된 여러 비트 반전
  >
  >   `a ^= (0x7<<3);`
  >
  > - 떨어져 있는 여러 비트 반전 (예> 5, 4, 3번 비트) 
  >
  >   `a ^= (0x1<<5) + (0x3<<2);`
  >
  >   
  >
  > - 비트 검사 (예> 5번 비트)
  >
  >   `a & (0x1 <<5);`
  >
  >   
  >
  > - 비트 추출 (예 > 6, 5, 4번 비트)
  >
  >   `b = (a>>4) & 0x7;`
  
- 비트 연산은 보통 매크로를 통해 구현하는 것이 가장 좋은 방식이다. (매크로의 경우 가로를 쳐야만 개발자가 의도한 코드 대로 동작된다.)

  ```c
  //한 비트 클리어
  #define clear_bit(data, loc) ((data) &= ~(0x1<<(loc)))
  //연속된 여러 비트 클리어
  #define clear_bits(data, area, loc) ((data) &= ~((area)<<(loc)))
  
  //한 비트 설정
  #define set_bit(data, loc) ((data) |= (0x1<<loc)))
  
  //한 비트 반전
  #define invert_bit(data, loc) ((data) ^= (0x1<<(loc)))
  ```

  

- 임베디드 환경에서는 메모리 직접 접근이 가능하다. 아래와 같은 방식이 포인터 배열을 선언해서 할당 한 뒤 매번 접근하는 것 보다 빠르게 활성화 할 수 있다.

  ```c
  //LED의 주소 값이 0x30000000이며 해당 주소에 100을 넣으려면
  *(volatile unsigned int*)0x30000000 = 100;
  //과 같이 정의할 수 있다. 하지만 근본적으로 매번 이렇게 접근할 수 없으므로 보통은 아래와 같이 전처리기를 사용한다.
  
  #define PA *(volatile unsigned int*)0x30000000 = 100
  
  void init(void){
      PA |= (0x7 << 5);
  }
  ```

   

- 임베디드 환경에서는 Switch를 기반으로한 코드는 메모리 접근 내역에서 좋지 않다. 따라서 다음과 같이 정의해 주는 것이 좋다.

  - 임베디드 시스템에서 사용되는 메모리는 HDD 없이 ROM, RAM으로 구성되어 있어서 실행코드를 최소화하거나, 작업 공간을 최소화하는 방법으로 효율적으로 사용해야 한다.

  ```c
  //올바르지 않은 예
  void display(int digit){
      switch(digit){
          case 1:
              PA &= ~(0x1 << 5);
             	break;
          case 2 :
              PA &= ~(0x2 << 5);
              break;
      }
  }
  ```

  

  ```c
  void display(int digit){
      //들어온 이진수의 일의 자리가 1이면 LED3을 ON 아니면 Off
      (digit & 0x1) ? (PA &= ~(0x1 << 5) ) : (PA |= (0x1 << 5) );
      //들어온 이진수의 십의 자리가 1이면 LED2를 ON 아니면 Off
      (digit & 0x2) ? (PA &= ~(0x1 << 6) ) : (PA |= (0x1 << 6) );
      //들어온 이진수의 백의 자리가 1이면 LED1를 ON 아니면 Off
      (digit & 0x4) ? (PA &= ~(0x1 << 7) ) : (PA |= (0x1 << 7) );
  }
  ```



- `volatile`의 기능적 의미는 캐시사용안함(no-cache)이다. 

  > 보통 프로그램이 실행될 때 속도를 위해 필요한 데이터를 메모리에서 직접 읽어오지 않고 캐시로부터 읽어온다. 하지만, 하드웨어에 의해서 변경되는 값들은 캐시에 즉각적으로 반영되지 않으므로 데이터를 캐시로부터 읽어오지 말고 주 메모리에서 직접 읽어오도록 해야한다. 이러한 특성 때문에 하드웨어가 사용하는 메모리는 volatile로 선언해야 하드웨어에 의해 변경된 값들이 프로그램에 제대로 반영된다.
  >
  > 
  >
  > - 추가적으로 volatile은 캐시를 사용하지 않는 특성 외에 컴파일러 최적화가 임의로 코드를 변경하는 것을 막을 수 있기 때문이다.

- `volatile`의 추가적인 기능은 컴파일러가 main에서 호출되지 않는 함수나 delay를 위한 BusyWaiting 코드의 생략을 막을 수 있다.

  ```c
  int time;
  
  //하드웨어에 의해 호출되고 메인에는 없는 코드
  void interrupt_handler(void){
      time++;
  }
  
  //비프음이 들리지 않는다. 
  //왜냐하면 컴파일러 입장에서 main에는 호출될 일 없는 interrupt_handler 함수는 최적화 대상이기 때문에 삭제시킨다.
  //따라서 int time;을 volatile int time;으로 선언해야 개선이 가능하다.
  //또한 busy waiting을 통해 1초를 기다리는 for문 역시도 컴파일러 입장에서는 연산만하고 내부가 아무것도 수행되지 않는 문이므로 초기화 대상이다.
  //따라서 volatile int j = 0;으로 선언해야 원하는 데로 수행된다.
  void main(){
      time = 0;
      int j = 0;
      while(1){
          if((time%10) == 0 && time != 0) beep();
          for(j = 0; j < 100000; j++);
      }
  }
  ```




#### 운영체제 설정

- 데이터 구조의 영역을 먼저 이해하는게 좋다.

  > ![image](https://user-images.githubusercontent.com/18729679/148637128-e1f03f80-443a-4543-8371-efc055d2691a.png)

- main()이 호출되는 과정은 OS의 유무에 따라 두 가지로 나눌 수 있다.

  - OS의 유(리눅스)

  > 1. 쉘에서 프로그램 수행 : ./test
  > 2. fork로 새로운 프로세스 복사
  > 3. execve() 호출(시스템 콜) -> sys_execve() 호출(사용자 모드에서 커널 모드로 전환)
  > 4. do_execve() -> open_exec()가 파일 정보를 읽어 들여 적합한 binary handler를 실행
  > 5. flush_old_exec()가 기존 프로세스 정보를 삭제하고, 현재 프로세스를 "current"로 설정
  > 6. 새로운 프로세스에서 사용할 메모리 레이아웃 설정. text, data, bss, stack 등의 세그먼트를 선형 메모리에 맵핑
  > 7. 독적 링커 메모리에 로딩, elf 포맷이면 load_elf_interp()가 /lib/ld_linux.so.2를 메모리에 로딩
  > 8. start_thread() -> elf 인터프리터(elf_interpreter) 실행
  > 9. sys_execve() 종료(커널 모드에서 사용자 모드 전환)
  > 10. reschedule() / 문맥교환
  > 11. _start 코드에 의해 main() 호출
  - OS 무
    - 하드웨어에 대한 초기화나 클록 설정 등은 부트 코드에서 처리, C 프로그램이 작동될 수 있는 환경을 만들어 주는 과정은 스타트업 코드에서 처리한다.

  > 1. 인터럽트 사용불가(disable)로 설정한다.(와치독도 마찬가지. 스타트업 코드 실행 중에 인터럽트 발생시 어떤 일이 발생할 지 모름)
  >
  > 2. Rw_data를 ROM 에서 RAM으로 복사(data 세그먼트 복사. 보통 임베디드 시스템에서 실행 파일은 ROM에 저장되는데 **data 세그먼트는 초기화된 전역 변수가 저장되는 곳**으로 프로그램이 실행될 때 값이 변할 수 있다.)
  >
  >    `보통 임베디드 시스템에서 실행 파일은 ROM에 저장되는데 data 세그먼트는 초기화된 전역 변수가 저장되는 곳으로 프로그램이 실행될 때 값이 변할 수 있다. 그러므로 이러한 값의 변동을 적용시키려면 RAM으로 복사해서 RAM에 있는 전역 데이터를 사용해야 한다.`
  >
  >    **`이 때 초기화되지 않은 전역 변수가 저장된 bss 세그먼트를 0으로 초기화한다. 이 과정 때문에 초기화 되지 않은 전역 변수는 0으로 초기화되는 것이다.`**
  >
  > 3. ZI(bss)영역을 0으로 클리어
  >
  > 4. 모드별 stack 생성
  >
  > 5. 힙 생성
  >
  > 6. 인터럽트 사용가능(enable)으로 설정
  >
  > 7. main() 호출



#### 포인터에 대한 오해

- 메모리에 대해 직접 접근은 포인터를 통해서만 가능하다?

  > 아니다. 위에도 나와 있듯이 직접 메모리 접근도 가능하다.
  >
  > ```c
  > *(volatile unsigned int*)0x30000000 = 100;
  > ```

- 포인터와 포인터 연산은 +만 가능하다?

  >  아니다. 오히려 +, - 둘 다 가능하지만 +의 경우 컴파일러가 protect 시키는 경우가 더 많다.
  >
  >  ```c
  >  int *p, *q;
  >  p = 104;
  >  q = 100;
  >  ```
  >
  >  일 경우 가능한 것은
  >
  >  | 정수와의 +, - | 단항 연산 | 포인터 - 포인터 | 대입   | 비교        |
  >  | ------------- | --------- | --------------- | ------ | ----------- |
  >  | p+1; p-1;     | p++; p--; | p - q;          | p = q; | if(p > q){} |
  >
  >  불가능한 것은
  >
  >  | 실수와 +, - 정수와 *,/ | 포인터끼리 +,*,/     |
  >  | ---------------------- | -------------------- |
  >  | p + 3.1; p - 3.1;      | p + q; p * q; p / q; |

- 배열의 이름은 포인터이다?

  > **배열의 이름은 포인터가 아니다!**
  >
  > 포인터의 특성은 값이 바뀔 수 있어야 하는데, 배열 이름은 상수이기 때문에 값이 바뀌지 않는다. 마찬가지로 &i도 주소를 나타내기는 하지만 포인터는 아니다. 배열 이름과 &i는 주소를 저장하거나 변경할 수 없기 때문이다.
  >
  > 만약 같다면, 아래와 같은 코드는 오류가 날 이유가 없다.
  >
  > ```c
  > int a[] = {1,2,3};
  > int c = 100;
  > int *b = &c;
  > a = b;
  > ```

- a[] 배열의 a와 &a는 같은 의미이다?

  > **a는 배열의 첫 요소에 대한 주소를 나타낸다. 그래서 한 단위는 배열 한 칸을 의미한다. &a는 배열의 시작 주소를 나타내며, 한 단위는 배열 전체가 된다.**
  >
  > | 실 주소 | 값의 표현      |
  > | ------- | -------------- |
  > | 0x100   | a[0] == *a     |
  > | 0x104   | a[1] == *(a+1) |
  > | 0x108   | a[2] == *(a+2) |
  >
  > 위의 표에서
  >
  > 1) a, &a의 값은 각각 얼마인가? => 0x100, 0x100
  > 2) a+1, &a + 1의 값은 각각 얼마인가? => 0x104, 0x10c



- 2차원 배열의 경우 다음과 같은 특징이 있다.

  > ![image](https://user-images.githubusercontent.com/18729679/149645736-80b852b8-351e-4c10-b212-990446887b56.png)
  >
  > 
  >
  > - 2차원 `a[2][3] = { {1,2,3}, {4,5,6} };` 에서 a, &a. a[0] 의 값이 어떻게 되는지는 상단의 그림과 같다.
  >
  > - 이를 근거로 아래와 같은 질문을 생각해보면 다음과 같다.
  >
  >   1. a[0], a[0] + 1의 값은?
  >   2. a, a + 1의 값은?
  >   3. &a, &a + 1의 값은?
  >
  >   ```wiki
  >   위의 그림을 보면 a라는 큰 배열 안에 a[0], a[1]이라는 작은 배열이 있다.
  >   여기서 a[1][0]의 주소는 a[1]이라는 것을 알 수 있고, a[0][0]의 주소는 a[0]이라는 것을 알 수 있다.
  >         
  >   그렇다면, a[0][1]을 포인터로 표현하면 어떻게 되는가.
  >   => a[0][0]의 주소에서 한 칸 이동한 것과 같다. a[0][0]는 a[0]의 주소와 같으며, a[0]에서 한칸 이동한 a[0] + 1이 a[0][1]의 주소이다.
  >   따라서, a[0][1]의 주소의 값을 접근하려면 *(a[0] + 1)이 된다.
  >   이 중에서 a[0]의 값은 a가 가지는 주소의 값에 해당하므로 *(*a + 1)이 a[0][1]과 같다.
  >   ```
  >
  >   정리하면 다음과 같다.
  >
  >   a[0] = 0x100, a[0] + 1 = 0x104
  >
  >   a = 0x100, a + 1 = 0x10C
  >
  >   &a = 0x100, &a + 1 = 0x118
  >
  >   
  >
  >   이를 근거로 하면 더 신기한 내용도 찾을 수 있다. 예를 들면 배열의 첨자를 음수로 하여 `a[1][-1]` 이라는 수도 내역을 찾을 수 있다.
  >
  >   배열의 음수 값은 그만큼의 이전 수를 나타내기 때문에 배열 사이즈를 정확히 안다면 사용해서 접근도 가능하다. `a[1][-1]`는 `a[1][0]`의 전 주소인 `a[0][2]`를 참조하게 된다.
  >
  >   ```c
  >   #include <stdio.h>
  >   
  >   void main(){
  >       int a[2][3] = { {1,2,3}, {4,5,6} };
  >       
  >       printf("a[1][-1] = %d", a[1][-1]);
  >       //3이 출력된다.
  >   }
  >   ```



#### 포인터를 쉽게 해석하는 방법

- 변수 이름을 기준으로 해서 가장 가까이 있고, 변수의 뒤에 있는 표현부터 해석 (가장 가까이 있는 뒤의 수식부터 해석해야 한다.)

  `int (*p[3])(int);`를 해석해보면 다음과 같다.

  > p를 기준으로 보면 가장 가깝게 변수의 뒤에 있는 표현부터 해석하면 [3]이라는 내역이므로 p는 배열임을 알 수 있다.



#### 문자열 포인터

> ```c
> #include <stdio.h>
> 
> void main(){
>     char a[] = "rose";
>     char *p = "grace";
>     
>     a[0] = 'n';
>     p[0] = 't';
>     
>     printf("a = %s\n", a);
>     printf("p = %s\n", p);
> }
> ```
>
> => 위의 코드에서 배열에 저장된 문제는 오류 없이 출력되지만, p에 대한 출력은 오류가 나거나 원하는데로 동작되지 않을 것이다.
>
> C 프로그램에서 사용하는 데이터는 종류에 따라 저장되는 위치가 달라진다. 텍스트 영역에는 프로그램 코드와 상수가 저장되며, 데이터 영역에는 정적(static) 변수와 전역 변수가 저장된다. 그리고, 스택은 함수가 사용하는 메모리로, 지역 변수나 함수의 인자, 리턴 값 등이 저장된다. 힙 영역은 malloc()과 같은 함수로 메모리를 할당 받아 사용할 수 있으며, 포인터로만 접근이 가능한 메모리이다.

- 텍스트 문자열을 배열에 넣는 것과 포인터의 넣는 것의 차이는 다음과 같다.

  > 보통 C언어의 메모리 영역에서 "rose", "grace"와 같이 텍스트로 선언한 것은 텍스트 메모리에 저장되며 해당 메모리는 수정이 불가능하다.
  >
  > 이 때 배열을 선언한 뒤 텍스트 메모리의 것을 저장하게 된다면 배열 하나하나에 텍스트 원소가 복사되어 넣어지는 것이기 때문에 다른 메모리에 할당된다. => 즉, 수정이 가능하다.
  >
  > 하지만 포인터로 선언하는 것은 텍스트 메모리의 주소를 직접 확인하여 넣는 것이다. => 즉, 텍스트 메모리를 접근하여 수정하게 됨으로 수정이 불가능하며 오류가 발생하거나, 그대로 참조된다.

- printf에 넣는 "%s" 나 "%d\n"도 텍스트 메모리 중 일부이다.

  > 따라서 다음과 같은 코드도 사용 가능하다.
  >
  > ```c
  > #include <stdio.h>
  > void error_print(int x);
  > 
  > void main(void){
  >     int a, i;
  >     char b[3][10] = {"test1","test2","test3"};
  >     char *p = "%s\n";
  >     
  >     for(i=0; i<3; i++)
  >         printf(p, b[i]);
  >   
  >     printf(p, "1~4사이의 숫자를 입력하시오");
  >     scanf("%d", &a);
  >     error_print(a);
  > }
  > 
  > void error_print(int x){
  >     char *a[5] = {"error number : %s\n", "200", "300", "400", "500"};
  >     printf(a[0], a[x]);
  > }
  > ```



#### void 포인터

- void 포인터는 데이터 타입이 없는 포인터를 말한다. 따라서 void 포인터는 저장은 가능하지만 연산에 사용할 수는 없다. 하지만, 모든 주소를 저장할 수 있다는 장점을 가지고 있다.

- 하나의 프로세서의 주소는 32비트로 표현된다. 따라서 4바이트가 동일하게 사용된다.

- 그러나 데이터 타입이 지정되지 않아 증감, 데이터의 시작 주소로부터 몇 바이트를 읽어와야 할지 결정하지 못한다.

  ```c
  #include <stdio.h>
  
  void add(void *p, void *q, void *s, int op);
  
  void main(void){
      int a = 1, b = 2, sum_i;
      float x = 1.5, y = 2.5, sum_f;
      
      add(&a, &b, &sum_i, 1);
      printf("int의 합 = %d\n", sum_i);
      //int의 합 = 3
  }
  
  void add(void *p, void *q, void *s, int op){
      if(op == 1)
          *(int*)s = *(int*)p + *(int*)q; //void 포인터는 연산 시 캐스팅
      else if(op == 2)
          *(float*)s = *(float*)p + *(float*)q;
  }
  ```

- malloc 함수의 return 값 역시 void 형 포인터이다.`void* malloc(size_t size);`

  > malloc() 함수는 할당한 메모리가 어떠한 타입으로 사용될지 모른다. 만약 메모리를 할당해주는 함수가 데이터 타입이 지정되어 있다면, C에서 사용하는 데이터 타입에 대응하는 메모리 할당함수들을 모두 만들어 주어야 한다. 그래서 데이터 타입이 지정되지 않은 void 포인터를 반환하는 것이다. 대신 이 포인터를 사용하려면, 꼭 원래의 데이터 타입으로 캐스팅해서 사용해야 한다. 
  >
  > `int *p = (int*)malloc(sizeof(int) * 10);`



#### 함수 포인터

- 임베디드 환경에서는 디바이스의 종류별로 함수를 다르게 호출하는 경우가 많아서 함수 포인터의 사용이 빈번하므로 확실히 이해해야한다.

- 함수 이름도 다른 포인터와 같이 함수가 사용하는 메모리의 시작 주소를 갖는 상수이다.

  ```c
  #include <stdio.h>
  
  void test(char* str);
  
  void main(void){
      void(*p)(char*);
      char a[] = "함수 포인터 테스트";
      p = test;
      p(a);
  }
  
  void test(char* str){
      printf("\n%s\n", str);
      //함수포인터 테스트
  }
  ```

  

- Switch 문을 활용해서 함수 포인터를 배열 자체에 넣어서 선언할 수도 있다.

  ```c
  #include <stdio.h>
  
  int a(int);
  int b(int);
  int c(int);
  int (*p[3])(int) = {a,b,c};
  
  void main(void){
      int i;
      int z;
      scanf("%d", &i);
      
      z = p[i-1](4);
      
      printF("%d\n", z);
   
  }
  
  int a(int k){
      return k*k;
  }
  
  int b(int k){
      return k*k*k;
  }
  
  int c(int k){
      return k*k*k*k;
  }
  ```

  
