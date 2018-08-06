# Spectre

## INDEX
1. 스펙터와 멜트다운 간략 소개

2. 예측 실행이란?

3. 스펙터 공격 방법

4. 취약 코드 예시

5. 스펙터 방어법


## 1.	스펙터와 멜트다운 간략 소개
<p>CPU는 프로세스 실행 시간을 단축시키기 위해 예측 실행이라는 기술을 사용합니다.</p>
<p>예를 들어 조건문 분기 상황을 만났을 때, CPU는 미리 결과를 예측하고 그 결과값을 캐시 메모리에 로드시켜 놓습니다.</p>
<p>만약 CPU가 예측한 것이 맞다면, 미리 연산해 놓은 결과값을 바로 사용해 실행 시간을 단축시킬 수 있지만, 틀렸다면 처음으로 되돌아가 다시 실행해야 합니다.</p>
그리고 CPU가 예측해 놓은 결과값은 캐시된 채로 남아있게 됩니다.
캐시는 3가지로 나누어 지는데, **L1(Level 1), L2, L3** 이렇게 3가지로 나눠 집니다.<
<p>예측 실행으로 인해 발생한 결과값은 L1 cache에 저장 됩니다.</p>
<p>이 때, 캐시에 저장된 데이터에 접근한 시간을 통해 해커는 timing attack을 시도하고, 어떤 데이터가 캐시에 저장되었는지 알아낸 후 자신의 메모리 경계를 넘어 다른 프로세스의 메모리를 읽을 수 있게 됩니다.</p>
</p>지금까지 Spectre 취약점에 대해 간략하게 설명해 보았습니다.</p>
스펙터에 대해 조금 더 깊게 들어가 보기 전에 스펙터 취약점과 함께 나온 **멜트다운(Meltdown)** 에 대해 간략히 보겠습니다.
<p>멜트다운도 메모리를 유출시킬 수 있는 취약점인데, 스펙터와는 몇 가지 차이점이 있습니다.</p>

* 멜트다운 취약점은 intel CPU에만 적용되는 취약점이지만 스펙터 취약점은 intel, AMD, ARM CPU에 적용되는 취약점 입니다.

* 그리고 멜트다운은 예측 실행을 위해 권한 상승 취약점을 이용하지만 스펙터 취약점은 분기 예측 취약점을 이용합니다.

* 멜트다운은 최종적으로 나오는 결과물이 커널 메모리를 읽어 오는 것이고, 스펙터는 다른 프로세스의 메모리를 가져오는 것입니다.
![Meltdown-Spectre-comparison-table](/Meltdown-Spectre-comparison-table.png)
이렇게 몇 가지 차이점이 있지만, 메모리를 유출시킨다는 점에서 비슷한 두 취약점이라고 볼 수 있습니다.

## 2.	예측 실행이란?
<p>프로세서는 앞으로 실행 흐름이 어떻게 될지 알지 못합니다. 따라서 예측 실행이라는 기술이 나오게 되었습니다. 앞에서 간단하게 설명했듯이, 예측 실행은 다음 실행 경로를 예측해 미리 결과를 캐시해 놓는 것을 말합니다.</p>
<p>프로세서가 현재의 레지스터 상태를 포함하는 체크포인트를 저장한 후, 프로그램이 따라가야 할 경로에 대한 예측해 그 경로에 따라 명령을 예측적으로 실행합니다.</p>
<p>만약 예측이 맞았다면, 저장해 두었던 체크포인트는 필요하지 않게 되고, 예측 실행에서 수행된 명령들은 프로그램 실행 순서 상에서 회수됩니다.</p>
<p>그렇지 않고 만약 프로세서가 잘못된 경로를 따라갔다고 판단하게 되면 프로세서는 체크포인트로 돌아가 거기서부터 다시 연산을 수행함으로써 실행 경로상의 모든 명령을 중지시키고 옳은 경로로 다시 실행을 시작하게 됩니다.</p>
<p>이런 이유로, 예측 실행은 만약 올바른 경로로 실행 되었다면 그 상태 그대로를 유지하지만, 그렇지 않을 때 예외 상황을 발생하게 됩니다.</p>
<p>스펙터 취약점은 이러한 예외 상황을 이용하는 메모리 누수 취약점 입니다.</p>

## 3.	스펙터 공격 방법
<p>여러 프로그램들이 같은 하드웨어에서 동작할 때, 동시 또는 시간 공유를 통해 한 프로그램의 동작으로 인해 발생되는 캐시 상태의 변화는 다른 프로그램에도 영향을 끼칠 수 있습니다.</p>
<p>그리고 이로 인해 한 프로그램에서 다른 프로그램으로의 의도하지 않은 정보 유출 결과가 나올 수도 있습니다.</p>
<p>스펙터 공격은 Side-Channel 공격을 통해 이루어 집니다. Side-Channel 공격은 우리가 일반적으로 주던 input이나 output과는 달리, 외적인 요소들을 통해 공격을 시도하는 기법입니다.</p>
예를 들어, 

* [Timing-attack(소요 시간 분석)](https://en.wikipedia.org/wiki/Timing_attack)

* Power-analysis-attack(전력 모니터링)

* Acoustic cryptanalysis-attack(음성 암호 해독)</br>

<p>등이 있는데, 모두 다 일반적인 방법이 아닌 알고리즘의 구현 상의 물리적인 문제를 가지고 공격을 하는 공격 기법들 입니다.</p>
<p>이 스펙터 공격 방법에서는 Timing-attack 기술이 사용됩니다.</p>
<p>처음 이 취약점을 발견한 사람은 Flush+Reload 공격 기술과 그 변종인 Evict+Reload 공격 기술을 사용해 공격 하였습니다.
이 기술들을 이용해 공격자는 캐시 메모리에서 희생자와 함께 공유하고 있는 Line을 축출함으로써 공격을 시작합니다.</p>
<p>희생자가 얼마 정도 실행한 후, 공격자는 축출해낸 캐시 Line에 해당하는 주소에서 메모리를 읽는 데 걸리는 시간을 측정합니다.</p>
<p>만약 공격자가 모니터 하고 있는 Line에 희생자가 접근했다면, 데이터가 캐시에 있을 것이고 접근 속도가 빠를 것입니다.
하지만 공격자가 모니터 하고있는 Line에 희생자가 접근하지 않았다면, 속도가 느릴 것입니다.</p>
<p>이러한 방법으로, 접근 시간을 측정함으로써 공격자는 자신이 모니터 하고 있는 Line에 희생자가 접근 하였는지에 대한 여부를 확인할 수 있습니다.</p>
<p>Flush+Reload 공격 기술과 Evict+Reload 공격 기술의 차이점은 캐시 Line을 축출해 내는데 사용되는 방법 입니다.</p>
그건 좀 더 공부한 뒤에 자세히…

## 4.	취약 코드 예시
이제 스펙터에 취약한 코드를 통해 스펙터에 대해 더 자세히 알아보겠습니다.
```c
if (x < array1_size)
    y = array2[array1[x] * 256];
```
<p>이 코드가 어떤 곳으로부터 int형 변수 x를 받아오는 함수의 일부분이라고 생각해 보십시오.</p>
<p>이 코드는 크기가 array1_size인 array1과 크기가 64kb인 배열 array2에 접근하였습니다.</p>
<p>내부에선 보안에 필요한 bounds check를 하게 될 것입니다. Bounds check란 변수가 사용되기 전 메모리 경계 내에 변수가 있는지 검사하는 것입니다.</p>
<p>따라서 이 검사를 통해 프로세스가 array1 경계 외부에 있는 민감한 메모리를 읽을 수 없도록 막습니다.</p>
만약 [Bounds-check](https://en.wikipedia.org/wiki/Bounds_checking)를 하지 않는다면 경계를 벗어난 입력값 x가 예외를 발생시켜 메모리의 중요한 부분에 접근하게 될 수도 있습니다.
<p>또한 아래와 같은 과정을 통해 잘못된 분기 예측을 하게 만들 수 있습니다.</p>

1) 경계를 벗어난 값을 변수 x에 입력합니다.

2) 프로세스는 array1_size와 x의 값을 비교합니다. -> 예외 상황 발생하게 되므로 DRAM에서 array1_size가 올 때까지 대기

3) 이때 분기 예측이 조건문을 참으로 가정하고 조건문 내부 명령을 실행합니다.

4) 그리고 array1의 기본 주소+x 한 곳의 값을 메모리로부터 가져옵니다(결과값 = x).

5) array2[x * 256]의 데이터를 메모리에게 요청합니다.

6) 이제 DRAM으로부터 array1_size가 도착하고, 잘못된 가정 이었음을 인지합니다. -> 레지스터 상태를 되돌립니다.

7) 그러나 실제 프로세스 상에서는 array2가 예측적으로 읽은 값이 x에 의존하는 주소 방식이기 때문에 캐시 상태에 영향을 주게 됩니다.

이제 x의 값을 알기 위해서 공격자는 캐시 상태의 변경사항을 찾아내야 합니다. 이 때 Timing-attack이 사용됩니다. 

8) array2[n*256] 배열에 대해서 읽는 속도가 n = x 일 때 빠를 것입니다. (캐시에 저장되어 있기 때문에) 이를 이용해 값을 알아냅니다.

## 5.	스펙터 방어법
멜트다운은 **카이저 패치** 라는 소프트웨어 패치로 어느정도 방어가 되지만 스펙터는 공식적으로 나온 방어법이 존재하지 않습니다.
<p>스펙터는 멜트다운과 달리 이전에 전혀 볼 수 없었던 공격 유형이기에 어떤 공격이 가능한지 또는 어떠한 위협이 있을 수 있는지에 대한 명확한 자료가 없기 때문에 더욱 방어하기 힘든 취약점 입니다. </p>
<p>중요한 것은 두 취약점 모두 CPU 설계 자체의 문제이기 때문에 어떤 소프트웨어적인 패치로도 완벽히 방어할 수는 없습니다.</p>
<p>따라서 현재의 CPU 구조를 바꾸지 않는 이상 이 취약점은 영원할 것으로 보입니다.</p>
