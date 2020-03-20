# Storage Bindings and Lifetime

> 명령형 프로그래밍 언어의 기본 특성은 대부분 변수에 대한 스토리지 바인딩이 `디자인에 의해 결정`됩니다.

변수가 바인딩 된 메모리 셀은 어떻게 든 사용 가능한 메모리 풀에서 가져와야합니다. 이 과정을 <b>`allocation(할당)`</b>이라 부름니다.

`Deallocation(할당 해제)`는 변수에서 바인딩되지 않은 메모리 셀을 사용 가능한 메모리 풀로 다시 배치하는 프로세스입니다.

변수의 `lifetime`은 변수가 특정 메모리 위치에 바인딩되는 동안의 시간입니다. 그래서 변수의 lifetime의 시작은 특정 메모리 위치에 바운드 될때이고, 끝은 셀로부터 unbound 될 때 입니다.

변수의 스토리지 바인딩을 조사하려면 스칼라(구조화되지 않은) 변수를 수명에 따라 `네 가지 범주`로 분리하는 것이 편리합니다.

[1. named static](#Static-Variables)
[2. stack-dynamic](#Stack-Dynamic-Variables)
[3. explicit heap-dynamic](#Explicit-Heap-Dynamic-Variables)
[4. implicit heap-dynamic](#Implicit-Heap-Dynamic-Variables)

4개의 카테고리의 목적, 장/단점에 대해 설명할 것 입니다.

## Static Variables

> static 변수들은 프로그램 실행 전에 memory cells에 바운딩 되고, 프로그램 종료시까지 같은 memory cells에 바운드 되어 유지됩니다.

```cpp
T subProgram(){
    static T history; //local static variables
}
```

몇가지 특성 중 전역 적으로 액세스 가능한 변수는 프로그램 실행 전체에서 종종 사용되므로 해당 실행 중에 동일한 스토리지에 바인드 해야 합니다.
때론 히스토리에 민감한 서브 프로그램을 사용하기에 편리하며, 이런 서브 프로그램에는 `로컬 정적 변수(local static variables)`를 가지고 있어야 합니다.

```java
/**
 * Counter.java
 *
 *
 * Created: Mon Sep 15 16:47:41 2003
 *
 * @author Stuart C. Shapiro
 */

public class Counter {
public static int count;

public Counter (){}

/* Prints the number of times that it has been called.*/
public static void counting_function() {
    System.out.println("count = " + ++count);
}

/* Demonstrates counting_function by calling it 5 times. */
public static void main (String[] args) {
counting_function();
counting_function();
counting_function();
counting_function();
counting_function();
} // end of main ()

}// Counter

-------------------------------------------------------
<cirrus:Programs:1:142> javac Counter.java

<cirrus:Programs:1:143> java Counter
count = 1
count = 2
count = 3
count = 4
count = 5
```

클래스 변수의 인스턴스는 클래스 카운터의 인스턴스를 구성하지 않고 사용할 수 있습니다. 

* 정적 변수의 장점 중 하나는 `효율성`입니다.

정적 변수의 모든 주소 지정이 직접적(direct)일 수 있습니다. (정적 변수는 기본 레지스터를 통해 처리되므로 스택 할당 변수와 마찬가지로 비용이 많이 듭니다.)
: 다른 종류의 변수는 종종 간접 주소 지정을 필요로하므로 속도가 느립니다. 또한 정적 변수의 할당 및 처리 위치에 대해서는 런타임 오버 헤드가 발생하지 않지만, 그러더라도 이 시간은 무시할 정도 입니다.

* 스토리지에 정적 바인딩의 한 가지 단점은 `유연성이 떨어지는 것`입니다.

특히, 정적 변수만 있는 언어는 재귀 서브 프로그램(recursive subprograms)를 지원할 수 없습니다.

* 다른 단점은 변수들 사이의 `스토리지 공유를 할 수 없습니다.`

예를 들어, 프로그램에 두 개의 서브 프로그램이 있고 둘 다 큰 배열을 필요로한다고 가정하고, 추가로 두 개의 서브 프로그램이 동시에 활성화되지 않는다고 가정해 보자.
만약 정적 배열이라면, 배열에 대해 동일한 스토리지를 공유 할 수 없습니다.

C 및 C++ 에선 프로그래머가 함수에 변수 정의에 정적 지정자를 포함시켜 변수를 정적으로 정의 할 수 있습니다.

참고할 점은, 정적 수정자(static modifier)가 C ++, Java 및 C#의 클래스 정의에서 변수 선언에 나타날 때 변수가 인스턴스 변수가 아닌 `클래스 변수`임을 나타냅니다. `클래스 변수는 클래스가 처음 인스턴스화 되기 전에 정적으로 생성`됩니다.

## Stack-Dynamic Variables

> 스택 동적 변수는 선언문이 구체화(be elaborated) 될 때 스토리지 바인딩이 생성된다. 하지만 타입이 정적으로 바인딩 된다. `stack에 할당 되며` `런타임에 동적으로 할당`된다.

이러한 `선언의 구체화(Elaboration)`는 선언과 연관된 스토리지 할당 및 바인딩 프로세스를 말하며, 실행은 선언이 첨부 된 코드에 도달 할 때 발생합니다.

`따라서 런타임 중에 elaboration가 발생합니다.`

* `Compile time` is reading your source code, converting it to a binary representation, and generating errors for incorrect syntax.
* `Elaboration time` includes flatting out the hierarchy, resolving hierarchical references, and propagating parameter overrides.
* `Run-time` is executing the resulting elaborated code starting at simulation time 0 and advancing to the next slot.

예로, Java 메소드의 시작 부분에 나타나는 변수 선언은 메소드가 호출 될 때 구체화(elaborated) 되고 해당 선언에 의해 정의 된 변수는 메소드의 실행이 완료되면 할당 해제됩니다.

(In Java, C++, and C#) Stack-Dynamic Variables은 보통 `함수, 메소드, 함수의 로컬 변수`로 사용됩니다.

서브 루틴 A가 서브 루틴 B를 호출 한 다음 B가 종료되고 A가 C를 호출하는 경우,
C의 매개 변수 및 로컬 변수에 사용되는 스택 메모리는 B에 의해 사용 된 일부 또는 모든 메모리 셀이되며 더 많을 수 있습니다.

```c
/*
 * C Program testing stack reuse
 *
 */

#include <stdio.h>

void a(){
  int i = 743;
  printf("In a, i = %d\n", i);
}

void b() {
  int j;
  printf("In b, j = %d\n", j);
}

int main() {
  a();
  b();  //a 함수가 사용한 스택 메모리를 사용하고 있다.
  return 0;
}

-----------------------------------

<wasat:Programs:2:127> gcc -Wall stackReuse.c -o stackReuse.out
<wasat:Programs:2:128> ./stackReuse.out
In a, i = 743
In b, j = 743

```

서브 루틴의 로컬 변수 주소는 서브 루틴이 사용하는 스택 영역의 시작과 관련하여 항상 동일한 주소입니다. 이것이 런타임까지 로컬 변수의 실제 주소가 무엇인지 알지 못하더라도 컴파일러가 서브 루틴에 대한 코드를 컴파일 할 수있는 방법입니다.

* 장점
재귀 함수 호출시 동적으로 메모리를 복사되면서 각각의 버전을 갖는데 이때 stack-dynamic variables이 사용하기 편리합니다.

* 단점
    정적 변수에 비해 할당 및 할당 취소의 런타임 오버 헤드이며 간접 주소 지정이 필요하기 때문에 액세스 속도가 느려질 수 있으며 서브 프로그램이 히스토리에 민감하지 않다는 사실입니다.

    서브 프로그램의 시작 부분에 선언 된 모든 스택 동적 변수가 별도의 조작없이 함께 할당되고 할당 해제되기 때문에, 스택 동적 변수를 할당 및 할당 해제하는 데 필요한 시간은 중요하지 않습니다.

## Explicit Heap-Dynamic Variables

> 프로그래머가 작성한 명시적 런타임 명령에 의해 할당 및 할당 해제되는 `이름이없는 (추상적 인) 메모리 셀`입니다. 이러한 변수는 힙에 할당, 해제되며 포인터나 참조된 변수를 통해 참조 할 수 있습니다.

힙은 예상치 못한 사용으로 인해 조직이 크게 구성되지 않은 스토리지 셀의 모음입니다.

```cpp
int *intnode; // Create a pointer
intnode = new int; // Create the heap-dynamic variable .. .
delete intnode; // Deallocate the heap-dynamic variable
                    // to which intnode points
```

명시적 힙 동적 변수는 컴파일 타임에 유형에 바인딩되므로 해당 바인딩은 정적입니다.
그러나 이러한 변수는 작성시, 즉 런타임 중에 스토리지에 바인드됩니다.

추가적으로, 명시적 힙 동적 변수를 작성하기 위한 서브 프로그램 또는 연산자 외에도 일부 언어에는 명시 적으로 파기하기 위한 서브 프로그램 또는 연산자가 포함되어 있습니다.

```java
import java.util.*;

public class HeapDemo {

public static HashSet singleton(Object obj) {
HashSet set;
set = new HashSet();
set.add(obj);
return set;
}

public static void main (String[] args) {
HashSet myset;
myset = singleton("element");
System.out.println(myset);
myset = singleton("another");
System.out.println(myset);
}

}// HeapDemo

-------------------------------------------------------
<cirrus:Programs:2:101> javac HeapDemo.java

<cirrus:Programs:2:102> java HeapDemo
[element]
[another]
```

HashSet의 메모리는 singleton 메서드로 할당되지만 singleton의 종료 후에도 살아남 아야하므로 스택에 할당 할 수 없습니다.
힙 메모리가 더 이상 필요하지 않을 때 (명명 된 변수에서 더 이상 도달 할 수 없을 때) 힙 메모리를 반환해야합니다.

그렇지 않으면 충분히 오래 실행되는 프로그램이 힙을 사용하여 비정상적으로 종료 될 수 있습니다.

힙 메모리에 점점 도달 할 수 없지만 재 할당에 사용할 수없는 프로세스를 `메모리 누수`라고합니다.

C 및 C ++에서 힙 메모리는 각각 연산자 free (p) 또는 delete p 하여 명시 적으로 할당 취소해야합니다.

* 장점
명시적 힙 동적 변수는 종종 실행 중에 커지거나 축소해야하는 `링크드 리스트 및 트리`와 같은 `동적 구조를 구성하는 데 사용`됩니다. 이러한 구조는 포인터 또는 참조 및 명시적 힙 동적 변수를 사용하여 편리하게 구축 할 수 있습니다.

* 단점
포인터 및 참조 변수를 올바르게 사용하기 어렵고 `변수에 대한 참조 비용`과 `필요한 스토리지 관리 구현의 복잡성`입니다. 이것은 본질적으로 `힙 관리의 문제`이며 비용이 많이 들고 복잡합니다.

## Implicit Heap-Dynamic Variables

> 암시 적 힙 동적 변수는 `값이 할당 된 경우`에만 힙 스토리지에 바인드됩니다.
> `실제로 모든 속성은 할당 될 때마다 바인딩됩니다.`

```js
//js code
highs = [74, 84, 86, 90, 71];
```

highs라는 변수가 프로그램에서 `이전에 사용되었는지 또는 사용되었는지에 관계없이` 이제는 5 개의 숫자 값으로 구성된 배열입니다.

## 참고

* 코드 참고: [Variables and Binding](https://cse.buffalo.edu/~shapiro/Courses/CSE305/Notes/notes6.html)