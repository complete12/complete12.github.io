---
title: "chapter01. 자바8을 눈여겨봐야 하는 이유"
date: 2020-07-13 19:08:28
categories:
  - java 8 In Action
tags: 
  - java
  - java8InAction
  - study
toc: true
toc_label: "목차"
toc_sticky: true
---

# 1. 자바8을 눈여겨봐야 하는 이유
## 자바8 변화의 배경 : 
* 컴퓨팅 환경의 변화 : 멀티코어와 대용량 데이터집합(빅데이터) 처리
* 시대적 변화 요구 : 기존의 명령형을 탈피 -> **함수형** 스타일 아키텍처

## 자바8을 구성하는 핵심적인 사항 :
* 간결한 코드<br>
  예) 무게에 따라 목록(inventory)에서 사과 리스트를 정렬하는 코드<br>
  * 기존
  ```java
  Collection.sort(inventory, new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
      return a1.getWeight().compareTo(a2.getWeight());
    }
  });
  ```
  * 자바8
  ```java
  inventory.sort(comparing(Apple::getWeight));
  ```
* 멀티코어 프로세스의 간단한 활용
  * 병렬 연산을 지원하는 스트림 API 제공
  * 메서드에 코드를 전달하는 간결기법 (메서드 레퍼런스와 람다)
  * 인터페이스의 디폴트 메서드 : 인터페이스, 라이브러리의 간결성 유지 및 재컴파일을 줄이는 데 활용
  
## 스트림 처리<small>(stream processing)</small>
#### 스트림이란? 
* 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임
* 입력 스트림에서 데이터를 한 개씩 읽어들이며 출력 스트림으로 데이터를 한개씩 기록.(_조립라인처럼 어떤 항목을 연속으로 제공하는 기능_)
* 자바8에서 `java.util.stream` 패키지에 위치.
* 기존에 한 번에 항 항목 처리 -> 고수준으로 추상화해서 일련의 스트림으로 만들어 처리

#### 장점
: 스트림 파이프라인을 이용해서 입력 부분을 여러 CPU 코어에 쉽게 할당 -> '스레드'라는 복잡한 작업을 사용하지 않으면서도 공짜로 병렬성을 얻을 수 있다.
#### 유의할 점
: 스트림 메서드로 전달하는 코드는 동시에 실행하더라도 **안전하게 실행**되어야 한다. <br>
#### 공유된 가변 데이터(shared mutable data)에 접근하지 않아야 한다. 
: 순수(pure) 함수, 부작용 없는(side-effect-free) 함수, 상태 없는(stateless) 함수

#### 컬렉션 API 사용 시 중첩된 제어 흐름 문장을, 스트림 API를 사용하여 해결할 수 있다.
* 컬렉션 : 구현 시 for-each 루프를 이용하여 각 요소를 반복하며 작업 수행 (외부 반복 : external iteration) -> 대량의 데이터 반복적으로 단일 CPU에서 처리
* 스트림 : 라이브러리 내부에서 모든 데이터가 처리되어 루프를 신경 쓸 필요 없어짐 (내부 반복 : internal iteration) -> 대량의 데이터를 서로 다른 CPU 코어에 각 작업을 할당하여 병렬로 작업 수행

_CPU 코어가 8개라면 단일 CPU 컴퓨터에 비해 8배 빨리 작업 처리가 가능하다._

#### 멀티스레딩 모델의 '멀티코어 활용 어려움' 해결 가능
* 멀티스레딩 : 각각의 스레드는 동시에 공유된 데이터에 접근 시, 스레드를 잘 제어하지 못하면 원치 않는 방식으로 데이터가 바뀔 수 있다.
* 스트림 : 라이브러리에서 분할(큰 스트림을 병렬로 처리 가능한 작은 스트림으로 나눔)을 처리한다. <br>
또한 filter같은 라이브러리 메서드로, 전달된 메서드가 상호작용을 하지 않는다면 가변 공유 객체를 통한 병렬성 확보 가능

## 자바 함수
자바8에서는 함수를 새로운 값의 형식으로 취급 가능<br>
일급(first-class)값 과 이급값 :
* 일급값 : 프로그램을 실행하는 동안 값을 바꾸어 전달 가능 (예. int, double, String, HashMap 등)
* 이급값 : 메서드, 클래스 등 프로그램 실행 중 자유롭게 전달할 수 없는 구조체 -> 자바8에서 일급값으로 바꿀 수 있는 기능 추가

### 메서드 레퍼런스
사용할 메서드를 `::`를 이용해서 값으로 사용 가능<br>
예 1.) 디렉터리에서 모든 숨겨진 파일을 필터링하는 기능 구현
* 기존 : File 클래스에서 isHidden 메서드 제공(File 클래스를 인수로 받아 숨김파일 여부를 boolean으로 반환하는 함수)
```java
// FileFilter 객체 내부에 위치한 isHidden의 결과를 File.listFiles 메서드로 전달하는 방법으로 숨겨진 파일 필터링
File[] hidddenFiles = new File(".").listFiles(new FileFilter() { // FileFilter로 isHidden을 감싼 다음 FileFilter를 인스턴트화
  public boolen accept(File pathname) { // FileFilter 인터페이스에서 제공하는 추상메서드, 조건에 맞는 pathname이 포함되었는지 여부 반환
    return pathname.isHidden(); // 실제 필요한 기능
  }
});
```
* 자바8
```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden); // 이미 isHidden 함수는 준비되어 있으므로 lisfFiles에 isHidden 함수를 직접 전달
```

예 2.) 사과가 녹색 사과인지, 150 그램보다 무거운지 판별하는 기능 구현
* 기존 :

```java
// Apple 클래스 내부에 getColor 메소드로 사과 색 반환하는 기능은 이미 구현되었다고 가정
public static List<Apple> filterGreenApples(List<Apple> inventory) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if ("green".equals(apple.getColor())) { // 실제 필요한 기능
      result.add(apple);
    }
  }
  return result;
}

// 위의 코드와 중복되는 코드가 많다.
public static List<Apple> filterHeavyApples(List<Apple> inventory) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (apple.getWeight() > 150) { // 실제 필요한 기능
      result.add(apple);
    }
  }
  return result;
}
```

* 자바8

```java
public static boolean isGreenApple(Apple apple) {
  return "green".equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple) {
  return apple.getWeight() > 150;
}

public interface Predicate<T> {
  boolean test(T t); // 인수로 값을 받아 true/false를 반환하는 test 함수
}

static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
  List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
    if (p.test(apple)) { // 사과가 p가 제시하는 조건에 맞으면 리스트에 추가
      result.add(apple);
    }
  }
  return result;
}

// 사과가 녹색인지 판별
filterApples(inventory, Apple::isGreenApple);
// 사과가 150그램보다 무거운지 판별
filterApples(inventory, Apple::isHeavyApple);
```

### 람다 : 익명 함수
함수도 값으로 취급 가능 (직접 메서드를 정의할 수도 있지만, 이용할 수 없는 편리한 클래스나 메서드가 없을 때 람다 문법을 이용하여 좀 더 간결하게 코드 구현 가능)<br>
예) 앞의 예제를 람다(익명 함수)로 구현

```java
filterApples(inventory, (Apple apple) -> "green".equals(apple.getColor()));
filterApples(inventory, (Apple apple) -> apple.getWeight() > 150;
filterApples(inventory, (Apple apple) -> apple.getWeight() > 150 || "green".equals(apple.getColor()));
```

## 디폴트 메서드
: 더 쉽게 변화할 수 있는 인터페이스를 구현 가능하도록 한다.<br>
하지만 프로그래머가 직접 디폴트 메서드를 구현하는 상황은 흔치 않음 -> 특정 프로그램 구현에 도움을 주는 기능이 아닌, 미래에 프로그램이 쉽게 변화할 수 있는 환경을 제공하는 기능<br>
* 구현 클래스에서 구현하지 않아도 되는 메서드를, 인터페이스가 포함할 수 있는 기능
* 메서드 바디는 클래스에서 구현하는 것이 아니라, 인터페이스의 일부로 포함된다.

예) Collections.sort는 사실 List 인터페이스에 포함된다. 이론적으로는 Collectios.sort(list, comparator)가 아니라 list.sort(comparator)를 수행하는 것이 적절하다.<br>
* 기존 : 인터페이스를 업데이트하려면 해당 인터페이스를 구현하는 모튼 클래스도 업데이트해야 하므로 불가능에 가까움.(List를 구현하는 모든 클래스가 sort를 구현해야 함)
* 자바8 : `default` 키워드를 통해 List에 직접 sort 메소드 구현 가능. 자바8의 List 인터페이스에, Collection.sort를 호출하는 디폴트 메서드 정의가 추가됨

```java
default void sort(Comparator<? super E> c) {
  Collections.sort(this, c);
}
```
* **다이아몬드 상속 문제**를 피할 수 있다.

## 함수형 프로그래밍
* 메서드와 람다를 일급값으로 사용
* 가변 공유 상태가 없는 병렬 실행을 이용해서 효율적이고 안전하게 함수/메서드를 호출 가능
* NullPointException 회피를 위한 Optional<T> 클래스 제공 : 값을 갖거나 갖지 않을 수 있는 컨테이너 객체.(값이 없는 상황을 어떻게 처리할 지 명시적으로 구현하는 메서드 포함)
* 패턴 매칭 기법 : if-then-else 가 아닌 케이스로 정의하는 수학/함수형 프로그래밍 기능 (정규표현식). 클래스 패밀리 방문 시 이용하는 visitor 패턴과 결합하여 활용 가능.

