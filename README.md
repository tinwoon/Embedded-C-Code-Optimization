# Embedded-C-Code-Optimization
#### 임베디드 시스템 개발 환경

- 네이티브 개발환경

  > 개발하는 시스템의 프로세서와 프로그램이 실행될 시스템의 츠로세서가 같은 개발환경

- 크로스 컴파일 개발환경

  > - 임베디드 소프트웨어 개발환경은 개발하는 시스템의 프로세서와 실행될 시스템의 프로세서가 같지 않은 환경
  >
  > - 생성되는 개발 환경을 다른 컴파일러를 사용해야 한다는 뜻이 된다.
  > - 즉 임베디드 소프트웨어를 개발하려면 다른 프로세서용 기계코드를 생성해주는 컴파일러가 필요하며 이를 크로스 컴파일러라 한다. (arm-linux-gcc, ADC 등)
  > - 크로스 컴파일러, 크로스 어셈블러, 링커/로케이터 등으로 구성된 소프트웨어 개발도구를 "툴체인"이라 한다.

- 입출력 장치 제어 방법

  - 입출력 장치 제어 방법에는 두가지 방법이 있다.

  > - 메모리 맵I/O
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
  > - 

