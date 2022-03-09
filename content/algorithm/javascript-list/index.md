---
emoji: 🔮
title: Javascript로 리스트 ADT 구현
date: '2022-03-09 00:00:00'
author: YoungHwanJoe
tags: javascript
categories: algorithm
---

> 다음은 `자바스크립트 자료구조와 알고리즘 (마이클 맥일런)`의 내용을 정리한 것입니다.

## 1. 리스트 ADT(abstract data type)

리스트 ADT를 설계하려면 리스트의 정의, 프로퍼티, 리스트가 수행하는 동작, 리스트에 의해 수행되는 동작 등을 알아야 한다.

**리스트는 순서가 있는 일련의 데이터 집합**이다. 리스트에 저장된 각 데이터 항목을 요소 element라 부른다. 자바스크립트에서는 모든 데이터형이 리스트의 요소가 될 수 있다. 리스트에 저장할 수 있는 요소 수에는 제한이 없지만 일반적으로 리스트를 사용하는 프로그램의 가용 메모리가 리스트에 저장할 수 있는 최대 요소 수가 된다.

요소가 없는 리스트를 빈empth 리스트라고 한다. 리스트에 저장된 요소의 수를 리스트의 길이length라고 한다. 내부적으로 리스트는 요소 개수를 listSize 변수에 저장한다. 리스트에서는 요소를 리스트의 끝에 append하거나, 기존 요소 뒤 또는 요소의 앞 부분에 insert한 수 있다. remove동작을 이용해 리스트에서 요소를 삭제할 수 있고, clear동작으로 리스트의 모든 요소를 삭제할 수도 있다.

toString() 함수를 이용해 리스트의 모든 요소를 출력하거나 getElement()함수를 이용해 현재 요소의 값만을 출력할 수 있다.

**리스트는 위치를 가리키는 프로퍼티를 포함한다**. 리스트에는 front, end 프로퍼티가 있다. next() 함수로 리스트의 현재 요소에서 prev() 함수로 현재 요소에서 이전 요소로 이동할 수 있따. moveTo(n) 함수를 이용하면 n번째 위치로 한 번에 이동할 수 있다. currPos() 함수는 리스트의 현재 위치를 가리킨다.

함수리스트 ADT는 리스트를 어떻게 저장할지는 정의하지 않는다. 우리는 dataStore라는 배열을 이용해 리스트의 요소를 저장한다.

아래는 전체 리스트 ADT를 보여준다

| listSize(프로퍼티) | 리스트의 요소 수                       |
| ------------------ | -------------------------------------- |
| pos (프로퍼티)     | 현재 위치                              |
| length (프로퍼티)  | 리스트의 요소 수 반환                  |
| clear (함수)       | 리스트의 모든 요소 삭제                |
| toString(함수)     | 리스트를 문자열로 표현해 반환          |
| getElement(함수)   | 현재 위치의 요소를 반환                |
| insert(함수)       | 기존 요소 위로 새 요소를 추가          |
| append(함수)       | 새 요소를 리스트의 끝에 추가           |
| remove(함수)       | 리스트의 요소 삭제                     |
| front(함수)        | 현재 위치를 리스트 첫 번째 요소로 설정 |
| end(함수)          | 현재 위치를 리스트 마지막 요소로 설정  |
| prev (함수)        | 현재 위치를 한 요소 뒤로 이동          |
| next (함수)        | 현재 위치를 한 요소 앞으로 이동        |
| currPos (함수)     | 리스트의 현재 위치 반환                |
| moveTo (함수)      | 현재 위치를 지정된 위치로 이동         |

## 2. List 클래스 구현

앞에서 정의한 리스트 ADT를 이용해 List 클래스를 구현할 수 있다. ADT에는 없었지만 우선 생성자 함수부터 정의한다.

```tsx
function List() {
  this.listSize = 0;
  this.pos = 0;
  this.dataStore = [];
  this.clear = clear;
  this.find = find;
  this.toString = this.toString;
  this.insert = insert;
  this.append = append;
  this.remove = remove;
  this.front = front;
  this.end = end;
  this.prev = prev;
  this.next = next;
  this.length = length;
  this.currPos = currPos;
  this.moveTo = moveTo;
  this.getElement = getElement;
  this.length = length;
  this.contains = contains;
}
```

<aside>
💡 책에선 function 으로 정의했지만, 나는 ES6부터 도입된 class문법을 사용했다.

</aside>

### #2.1 Append: 리스트에 요소 추가

먼저 append() 함수를 구현한다. append() 함수는 리스트의 다음 가용 위치 (=listSize 변수의 값)에 새 요소를 추가하는 함수다.

```tsx
function append(element) {
  this.dataStore[this.listSize++] = element;
}
```

요소를 추가한 다음 listSize를 1만큼 증가시킨다.

### #2.2 Remove: 리스트의 요소 삭제

이번에는 리스트의 요소를 삭제하는 방법을 살펴본다. remove() 함수는 List 클래스의 함수 중 가장 구현하기 어려운 함수에 속한다. 우선 리스트에서 삭제하려는 요소를 찾은 다음, 요소를 삭제하고 나머지 배열 요소를 왼쪽으로 이동시켜 요소가 삭제된 자리를 메꿔야 한다. 다행히 splice () 변형자 함수를 이용하면 이 과정을 쉽게 해결할 수 있다. 우선 삭제할 요소를 찾는 헬퍼helper 한수 find()를 구현해보자.

```tsx
function find(element) {
  for (const i = 0; i < this.dataStore.length; ++i) {
    if (this.dataStore[i] === element) {
      return i;
    }
  }
  return -1;
}
```

### #2.2 Find: 리스트의 요소 검색

fine () 함수는 루프로 dataStore를 반복하면서 원하는 요소를 검색한다. 요소를 발견하면 요소의 위치를 반환한다. 요소를 발견하지 못하면 배열에서 요소를 찾지 못했을 때 반환하는 표준 값인 -1을 반환한다. remove() 함수는 find()함수의 반환값으로 에러를 검사할 수 있다.

remove() 함수는 find() 함수의 반환값을 splice() 함수에 넘겨주어 원하는 요소를 삭제한 다음 dataStore 배열을 연결한다. remove() 함수는 요소를 삭제했으면 true를 반환하고, 요소를 삭제하지 못했으면 false를 반환한다. 다음은 remove() 함수를 구현한 코드다.

```tsx
function remove(element) {
  const fountAt = this.find(element);
  if (fountAt > -1) {
    this.datStore.splice(foundAt, 1);
    --this.listSize;
    return true;
  }
  return false;
}
```

### #2.4 Length: 리스트의 요소 개수

length() 함수는 리스트의 요소 수를 반환한다

```tsx
function length() {
  return this.listSize;
}
```

### #2.5 toString: 리스트의 요소 확인

이번에는 리스트의 요소를 확인하는 함수를 만든다. 다음은 간단한 toString() 함수를 구현한 코드다.

```tsx
function toString() {
  return this.dataStore;
}
```

엄밀히 말해 toString() 함수는 문자열이 아닌 배열 객체를 반환한다. 하지만 배열을 반환하므로 현재 요소 상태를 확인할 수 있다.

일단 지금까지 구현한 List 클래스가 잘 동작하는지 중간점검을 해보자. 다음은 지금까지 구현한 코드를 동작시키는 간단한 테스트 프로그램이다.

```tsx
const names = new List();
names.append('A');
names.append('B');
names.append('C');
console.log(names.toString());
console.log(names.remove('B'));
console.log(names.toString());
```

### #2.6 Insert: 리스트에 요소 삽입

이번에는 insert() 함수를 살펴보자. 이전 예제에서 B를 삭제했다. 이번엔 다시 원래 위치로 Raymond를 돌려놓으려면 어떻게 해야 할까? 요소를 삽입하려면 어디에 요소를 삽입할지 지시해야 한다. insert() 함수에서는 리스트의 기존 요소 뒤에 새로운 요소를 삽입하게 한다. 다음은 insert() 함수를 구현한 코드다

```tsx
insert(element, after) {
     const insertPos  = this.find(after);
     if (insertPos > -1) {
         this.dataStore.splice(insertPos +1, 0 , element);
         ++this.listSize;
         return true;
     }
     return false;
}
```

insert() 함수는 헬퍼 함수 find(after)를 이용해 새 요소의 삽입 위치를 결정한다. 요소를 삽입할 위치를 찾았으면 splice() 함수를 이용해 새 요소를 리스트에 추가한다. 그리고 listSize를 1만큼 증가시키고 요소를 성공적으로 삽입했으므로 true를 반환한다.

### #2.7 리스트의 모든 요소 삭제

이번에는 새로운 요소를 리스트에 입력할 수 있게 리스트의 모든 요소를 삭제하는 함수를 구현한다

```tsx
clear() {
     delete this.dataStore;
     this.dataStore.length = 0;
     this.listSize = this.pose = 0;
}
```

clear() 함수는 delete 명령어로 dataStore 배열을 삭제한 다음 빈 배열을 다시 만든다. 마지막 행에서 listSize와 pos를 0 즉, 새 리스트의 시작 위치로 초기화한다.

### #2.8 Contains: 리스트에 특정값이 있는지 판단

contains() 함수는 어떤 값이 리스트에 포함되어 있는지를 확인할 때 사용하는 함수다. 다음은 contains() 함수를 구현한 코드다.

```tsx
contains(element) {
     for(let i=0; i < this.dataStore.length; ++i) {
         if(this.dataStore[i] === element) {
             return true;
         }
     }
     return false;
}
```

### #2.9 리스트 탐색

다음은 리스트 탐색 기능과 관련 있으며 마지막의 getElement() 함수는 리스트의 현재 요소를 출력한다.

```tsx
front() {
     this.pos = 0;
}
end() {
     this.pos = this.listSize -1;
}
prev() {
     if(this.pos >0) {
         --this.pos
     }
}
next() {
     if(this.pos < this.listSize ) {
         ++this.pos
     }
}
currPos() {
     return this.pos
}
moveTo(position) {
     this.pos = position;
}
getElement() {
     return this.dataStore[this.pos]
}
```

동작을 확인하려면 새로운 리스트를 만든다

```tsx
const names = new List();
names.append('A');
names.append('B');
names.append('C');
names.append('D');
names.append('E');

names.front();
console.log(names.getElement()); // A

names.next();
console.log(names.getElement()); // B

names.next();
names.next();
names.prev();
console.log(names.getElement()); // C
```

## 3. 리스트와 반복

반복자를 이용하면 List클래스의 내부 저장소를 직접 참조하지 않고 리스트를 탐색할 수 있다. List 클래스의 front() 함수, end() 함수, prev() 함수, next() 함수, currPos() 함수를 이용해 반복자를 구현할 수 있다. 반복자는 배열의 인덱스에 비해 다음과 같은 장점이 있다.

- 리스트 요소에 접근할 떄 내부 데이터 저장소가 무엇인지 걱정할 필요가 없다.
- 리스트에 새 요소를 추가했을 떄는 현재 인덱스가 쓸모 없는 값이 되는 반면 반복자를 이용하면 리스트가 바뀌어도 반복자를 갱신할 필요가 없다
- List 클래스에 사용하는 데이터 저장소의 종류가 달라져도 이전과 같은 방식으로 요소에 접근할 수 있다.

다음은 반복자를 이용해 리스트를 탐색하는 코드다

```tsx
const names = new List();
names.append('A');
names.append('B');
names.append('C');
names.append('D');
names.append('E');

for (names.front(); names.currPos() < names.length(); names.next()) {
  console.log(names.getElement());
}
```

위의 for문은 먼저 현재 위치를 리스트 앞으로 설정한다. 그리고 currPos() 함수가 리스트 길이보다 작을 때 루프를 반복한다. 루프를 돌 떄마다 next() 함수를 이용해 현재 위치를 한 요소씩 전진시킨다.

물론 아래처럼 반복자를 이용해 반대 방향으로도 리스트를 탐색할 수 있다.

```tsx
for (names.end(); names.currPos() >= 0; names.prev()) {
  console.log(names.getElement());
}
```

이 예제에서는 리스트의 마지막 요소에서 루프가 시작되고 prev()함수를 이용해 현재 위치가 0보다 크거나 같을때 현재 위치를 뒤로 이동시킨다. 리스트를 탐색할 떄만 반복자를 사용해야 하며 리스트의 항목을 추가하거나 삭제하는 동작과 반복자를 함께 사용하면 안된다.

> 💡 다만 위코드는 pos가 0보다 작아지지 않기때문에 순회가 끝나지 않는다는 단점이있다.

```toc

```
