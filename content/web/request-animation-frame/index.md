---
emoji: 🔮
title: requestAnimationFrame 활용한 애니메이션 구현
date: '2022-03-09 00:00:00'
author: YoungHwanJoe
tags: 최적화
categories: web
---

웹 애니메이션을 구현하는 방식은 크게 `setInterval`을 사용하는 방식과 `requestAnimationFrame`을 사용하는 방식이 있습니다.두 방식의 구현 방법과 차이점을 알아보도록 하겠습니다.

## setInterval을 사용한 웹 애니메이션 구현

웹에서 애니메이션을 구현하는 방법에서 전통적인 방식은 `setInterval`함수를 사용하는 것입니다.
예를 들면 다음과 같습니다.

```javascript
const animate = (cb, delay, duration) => {
  let count = 0;
  const intervalId = setInterval(() => {
    if (delay * count >= duration) return clearInterval(intervalId);
    cb();
    count++;
  }, delay);
};

const makeCb = (target, moveX) => {
  let start;
  let renderCount = 0;
  let $elapsed = document.getElementById('elapsed');
  let $total = document.getElementById('total');
  let $render = document.getElementById('render');
  return function () {
    if (start === undefined) {
      start = window.performance.now();
    }
    const elapsed = window.performance.now() - start;
    $elapsed.textContent = elapsed;
    renderCount++;
    $render.textContent = renderCount;
    const transformStyle = target.style.transform;
    const translateX = transformStyle.replace(/[^\d.]/g, '');
    target.style.transform = `translateX(${+translateX + moveX}px)`;
  };
};

const $box = document.getElementById('box');
animate(makeCb($box, 2), 10, 1000);
```

위 코드를 분석하면 다음과 같습니다.

1. 먼저 `animate`함수를 정의합니다. `animate` 함수는 콜백함수와 콜백함수의 호출주기, 그리고 애니메이션을 구동할 총 시간을 인자로 받습니다.
2. `animate`함수에 전달할 콜백함수를 반환하는 `makeCb`함수를 정의합니다. 이는 커링함수를 사용하여 콜백함수의 내용이 변하더라도 각각 다른 콜백함수를 생성해 `animate`에 전달할 수 있게 했습니다. `makeCb`함수는 움직이는 대상이 되는 HTML 엘리먼트와 렌더링시 움직일 x좌표상의 거리(px)를 인자로 받습니다.
3. `animate` 함수에 (1) `$box`엘리먼트를 x축으로 2px씩 움직이게하는 콜백함수와 (2) 렌더링 간격 10 밀리세컨드, (3) 애니메이션 시간 1000 밀리세컨드를 전달하여 호출합니다.
   위 `animate` 함수를 실행시키면, 1초간 10밀리 세컨드 간격으로 `$box` 엘리먼트를 x축으로 2px씩 움직입니다. 이때 렌더링은 총 100(=1000/10)번 일어납니다.
4. 이해를 위해 경과 시간(`elapsed`)과 렌더링 횟수(`render count`)를 함께 표시했습니다.

구현 결과는 아래 데모에서 확인할 수 있습니다.

https://codepen.io/younghwanjoe/pen/WNpWdZK

## requestAnimationFrame이란?

이번에는 `requestAnimationFrame`(이하 `rAF`)를 사용한 애니메이션 구현에 대해 설명하겠습니다. 네이밍에서 유추할 수 있듯이 애니메이션 구현을 위한 함수입니다. 함수 구조는 간단합니다. 인자는 하나를 받는데, 다음 렌더링에서 호출할 콜백함수입니다. 그리고 이 콜백함수에는 하나의 인자가 전달됩니다. `DOMHighResTimeStamp`라고 하는 타임스탬프입니다.

여기서 `DOMHighResTimeStamp`는 `timeorigin`을 기준으로 얼마나 시간이 지났는지 나타내는 타입입니다. `timeorigin`은 브라우저에서 스크립트가 시작됐을때를 말합니다. 따라서 rAF의 콜백함수에 전달되는 `DOMHighResTimeStamp`는 스크립트의 시작시점을 기준으로 콜백함수 호출 시점 사이의 시간차이를 나타냅니다. `window.performance.now()` 메서드 호출시에도 `DOMHighResTimeStamp`를 반환합니다. 참고로 브라우저가 아닌 Node환경에서는 `rAF`나 `window.performance`는 지원하지 않습니다. 다만 `Node` 환경의 '`perf_hooks`' 내장 라이브러리를 통해 브라우저의 `Web Performance APIs`의 `performance`와 같은 기능을하는 함수를 사용할 수는 있습니다.
이제 간단하게 `rAF`를 소개했고 위에서 `setInterval`을 통한 애니메이션 구현을 rAF를 사용해서 비슷하게 구현해봅니다. `rAF`는 `setInterval`과 같이 콜백 함수의 호출 간격을 따로 인자로 받지 않으므로 애니메이션 지속시간만 같게 하여 구현해보겠습니다.

```javascript
const animate = (cb) => {
  requestAnimationFrame(cb);
};

const makeCb = (target, moveX, duration) => {
  let start;
  let renderCount = 0;
  let $elapsed = document.getElementById('elapsed');
  let $render = document.getElementById('render');

  return function cb(timestamp) {
    if (start === undefined) {
      start = timestamp;
    }
    const elapsed = timestamp - start;
    $elapsed.textContent = elapsed;

    const transformStyle = target.style.transform;
    const translateX = transformStyle.replace(/[^\d.]/g, '');
    target.style.transform = `translateX(${+translateX + moveX}px)`;
    if (elapsed < duration) {
      renderCount++;
      $render.textContent = renderCount;
      requestAnimationFrame(cb);
    }
  };
};

const $box = document.getElementById('box');
animate(makeCb($box, 2, 1000));
```

위 코드를 분석해보면 다음과 같습니다.

1. `setInterval`과 달리 `rAF`는 콜백함수의 호출 시간간격이나 지속시간을 따로 인자로 받지 않습니다. 따라서 `DOMHighResTimeStamp`타입의 `timestamp`를 통해 `elapsed`라는 시간차이를 계산하여 사용합니
2. 처음 `rAF`의 콜백 함수가 시작되는 시점의 `timestamp`를 `start`에 저장하고 이후 콜백 함수 호출시마다 `start`와의 시간 차이를 계산한 값인 `elapsed` 사용하여 경과시간을 측정합니다.
3. `duration`으로 주어진 1000밀리세컨드가 `elapsed`보다 작을 경우에는 화면을 변화시켜주고 `elapsed`가 그보다 커지는 시점에 rAF 콜백함수 호출을 멈춥니다.
4. `rAF`가 호출하는 콜백함수에는 반드시 `rAF` 호출구문을 포함해야합니다. `rAF`는 재귀적으로 작동합니다.
5. `setInterval` 예제와 마찬가지로 이해를 위해 경과 시간(`elapsed`)과 렌더링 횟수(`render count`)를 함께 표시했습니다.

구현 결과는 아래 데모에서 확인할 수 있습니다.

https://codepen.io/younghwanjoe/pen/XWMQeJd

## setInterval과 requestAnimationRequest 비교

비교를 위해 두 함수를 사용한 애니메이션을 한 화면에서 동시에 실행시켜봤습니다.

https://codepen.io/younghwanjoe/pen/qBrwPOm

`setInterval`을 사용한 애니메이션이 적용된 빨간박스와 `rAF`를 사용한 애니메이션이 적용된 초록박스가 있습니다. 두 박스는 동시에 출발해서 애니메이션이 시작한지 1초가 되는 시점에 (아주 미세한 차이를 무시한다면) 동시에 멈춥니다. 그러나 여러분이 보는 화면에서 두개의 박스는 서로 멈추는 지점(x축 기준)은 **아마** 서로 다를 것입니다."**아마**"라는 표현을 사용한 이유에 대해서는 잠시 후에 설명하겠습니다.
위 예제들의 데모 상단에 표시되는 `setInterval`과 `rAF` 예제 데모들의 `render count`들도 다를 것입니다. `rAF` 예제에서는 렌더링을 해주는 콜백 함수의 호출 간격을 따로 설정하지 않았습니다. 두 예제의 렌더링 횟수가 다르므로 두 예제의 박스들이 움직인 거리도 다를겁니다.

여기서 잠깐 아래의 개념들을 살펴볼 필요가 있습니다.

## 프레임(Frame)과 프레임 레이트(Frame rate) 그리고 헤르츠(Hz)

아래는 `frame rate`에 대한 위키백과의 설명을 일부 각색한 것입니다.

> 프레임 레이트(`frame rate`)란 디스플레이 장치가 화면 하나의 데이터를 표시하는 속도를 말한다.
> 이와 같은 뜻으로 쓰이는 초당 `frame` 수(`frames per second`)는 1초 동안 보여주는 화면의 수를 가리킨다.
> 애니메이션은 보통 눈의 잔상을 이용해서 표시하기 때문에 1초에 30번 또는 25번 이상이 필요하나, 부드러운 표시를 위해서 1초에 60~120번의 `frame`을 표시하기도 한다.

우리가 애니메이션을 보는 것은 짧은 시간 간격에 이어지는 장면을 보는 것과 같습니다. 이 각각의 장면이 바로 `frame`입니다. 그리고 `frame rate`란 특정 시간 내에서 보여지는 `frame` 갯수를 보여주는 속도를 말합니다. 특정시간에 보여지는 `frame`의 갯수라고 봐도 무방합니다. 보통 기준은 1초당 보여지는 `frame` 갯수로 사용하므로 1초당 보여지는 `frame` 갯수를 `FPS(Frame Per Second)`로 표현하며 우리가 제일 흔히 접하게 되는 단위입니다.

비슷한 의미의 `Hz`라는 단위도 있습니다. 이는 1초 동안의 빈도를 나타내는 단위입니다. `FPS`는 `frame`이 1초동안 몇 번일어나는지를 나타내는 단위이고, Hz는 무언가가 1초동안 몇 번 일어나는지를 나타내는 단위입니다. 사실 우리에게는 같은 의미입니다. 굳이 `FPS`를 설명하고 `Hz`까지 설명하는 이유는 `rAF`의 콜백 함수의 호출 빈도수에 둘 다 연관되어 있기 때문입니다.

애니메이션을 구현하는 것은 `frame rate`를 기준으로 `frame`을 연속적으로 생성하는 것이라고 볼 수 있습니다. 위에서 구현한 두 개의 예제코드에서는 모두 `frame rate`를 직접 정의하지는 않았습니다. 다만 `setInterval`을 사용한 예제에서는 1초(1000밀리세컨드) 동안 10밀리세컨드마다 `frame`을 생성했으므로 100 FPS라고 정의했다고 할 수 있습니다.

아까 `rAF` 예제를 구현할때에는 은근슬쩍 넘어갔지만 `rAF`에서는 전달된 콜백 함수의 호출 간격를 어떻게 정할 수 있는지에 대해 설명하지 않았습니다. `setInterval`을 사용해봤다면 당연히 의문점이 들었을 것입니다. 이는 `FPS`와 `Hz`에 대한 이해가 필요하기 때문에 그렇습니다. `FPS`와 `Hz`에 대해 설명했으니 이제 `rAF`의 콜백함수의 호출 간격은 어떻게 정하는지 알아보겠습니다.

## requestAnimationFrame의 콜백함수의 호출 횟수

바로 이 부분이 애니메이션을 구현하는데에 있어 **setInterval과 rAF의 결정적인 차이점**입니다.
`rAF`의 콜백함수 호출에 대한 MDN 문서의 설명을 보겠습니다.

> The number of callbacks is usually 60 times per second, but will generally match the display refresh rate in most web browsers as per W3C recommendation.

`rAF`의 콜백 함수는 대개 1초에 60번, 즉 `60FPS`를 기준으로 호출됩니다. 그러나 W3C 권장을 따르는 대부분의 웹브라우저에서는 디스플레이의 `refresh rate`, 즉 디스플레이의 `Hz`를 따릅니다. 이것이 무슨 말일까요?

일반적인 가정에서 사용하는 모니터는 상당수가 `60Hz`를 지원합니다. 아까도 설명했듯이 `60Hz`는 곧 `60FPS`라고 할 수 있습니다. 따라서 `60Hz`를 지원하는 모니터를 사용하는 브라우저에서 `rAF`는 `60FPS`를 기준으로 콜백 함수를 호출합니다. 그래서 위 MDN 문서에서도 대개 `rAF`의 콜백함수가 1초에 60번 호출된다고 한 것입니다.

그렇다면 `100Hz`를 지원하는 모니터에서는 `rAF`의 콜백함수는 1초동안 몇번 호출될까요? 당연히 100번입니다. `100FPS`를 기준으로 호출되는 것과 같으니까요.

이 부분이 `setInterval`과 `rAF`의 애니메이션 구현의 가장 큰 차이점입니다. `setInterval`에서는 콜백 함수 호출 간격을 시간으로 조절함으로써 `FPS`를 설정하지만, `rAF`에서는 모니터의 주사율을 따르게 됩니다.

따라서 `setInterval`로 구현한 애니메이션은 모니터 주사율이 달라져도 `FPS`가 동일하지만, `rAF`로 구현한 애니메이션은 모니터 주사율에 따라 `FPS`가 달라집니다.

위에서 "여러분이 보는 화면에서 두개의 박스는 서로 멈추는 지점(x축 기준)은 **아마** 서로 다를 것입니다"라고 한 이유가 바로 이것입니다. 여러분은 아마 `60Hz`나 `144Hz` 등의 주사율을 가진 모니터로 화면을 봤을 것입니다(`100Hz`로 모니터를 사용하는 경우는 흔치 않을 겁니다). 이 경우에 두 개의 박스가 나타나는 예제 화면에서 1초동안의 두 박스 이동 거리는 서로 달랐을 것입니다. 두 애니메이션에서의 FPS가 서로 달랐을테니까요. 만약 `100Hz` 주사율의 모니터를 사용하거나, 주사율 변경이 가능한 모니터에서 `100Hz`로 주사율을 변경하고 두 박스의 이동 화면을 보게 된다면, 두 박스는 1초 동안 동일한 거리만큼 움직일 것입니다.

그렇다면 `rAF`에서 모니터 주사율과 무관하게 특정 `FPS`를 구현하려면 어떻게 할까요? `setInterval`처럼 시간 간격을 주면 될 것 같습니다. 하지만 `setInterval`을 사용할때 보다 조금 복잡한 구현이 필요합니다. 여기서 **쓰로틀(throttle)**을 사용합니다.

## 쓰로틀(throttle)을 사용하여 rAF에서 FPS 구현하기

**쓰로틀(throttle)**은 `디바운스(debounce)`와 함께 연속된 브라우저 이벤트 처리 기법으로 많이 언급됩니다. 이 둘은 연속된 이벤트에 대해 이벤트 리스너가 등록되어 있을때 이벤트 리스너의 불필요한 콜백호출을 줄이거나 폐기함으로써 리소스 사용을 줄이는 기법입니다.

여기에서는 FPS구현을 위한 쓰로틀을 주로 다룰 것이므로 `디바운스`에 대한 자세한 설명은 생략하고 쓰로틀 기법에 대한 간단한 설명만 하겠습니다. **쓰로틀은 연속된 이벤트를 그룹화해서 일정 시간간격동안 한번만 이벤트 처리**를 하는 것입니다. 이 개념을 FPS 구현에 적용해 본다면 1초(일정 시간간격)동안 **FPS의 간격**만큼 렌더링(이벤트 처리)를 해주는 것으로 이해할 수 있습니다.

여기서 **FPS 간격**이란 용어를 사용했습니다. 이는 정식적인 기술용어는 아니고 이해를 돕기 위해 사용한 용어입니다. 위에서 살펴본대로 `FPS`는 1초동안 보여지는 `frame`의 갯수입니다. 그리고 1초는 1000 밀리세컨드입니다. 이때 FPS를 100으로 설정하려고 한다면 1000/100 밀리세컨드마다 드로잉을
해주는 것과 같습니다.

여기서 한가지 주의할 점은, 모니터 주사율보다 높은 `FPS`를 구현할 수는 없다는 점입니다. `60Hz`주사율으로 설정된 모니터에서 `60FPS`이상을 구현할 수는 없습니다.

> 즉, 쓰로틀을 통해 구현하는 `FPS`는 사용하는 모니터의 주사율보다 클 수는 없습니다.

`rAF`의 콜백 함수의 호출과 드로잉이 반드시 함께 일어나지는 않습니다. `쓰로틀`을 통한 `FPS` 구현은 `rAF`의 콜백함수 횟수와는 연관이 없습니다. 아까도 살펴봤듯이, `rAF`의 콜백함수는 주사율을 따라 결정되는 것이고 이것을 우리가 제어하려는 것이 아닙니다. 우리가 `FPS`를 구현하는 방식은 `rAF`의 콜백 함수 내의 드로잉 로직, 즉 브라우저를 그리는 코드가 실행되는 횟수를 조절하는 것입니다. 때문에 드로잉 횟수를 조절해도 1초간의 최대 드로잉 횟수는 1초간의 `rAF`의 콜백 함수가 호출되는 횟수, 즉 모니터의 주사율보다 클 수 없습니다.

그럼 `rAF`를 사용한 예제에서 `FPS`를 직접 설정하도록 구현해보겠습니다. 대개 `60Hz`의 모니터를 많이 사용하므로 이보다 낮은 30 FPS을 설정해보도록 하죠.

```javascript
const animate = (cb) => {
  requestAnimationFrame(cb);
};

const makeCb = (target, moveX, duration, fps) => {
  let fpsInterval = 1000 / fps;
  let renderCount = 0;
  let start;
  let then;
  let $totalElapsed = document.getElementById('total_elapsed');
  let $render = document.getElementById('render');

  return function cb(timestamp) {
    if (start === undefined && then === undefined) {
      start = window.performance.now();
      then = window.performance.now();
    }
    const totalElapsed = window.performance.now() - start;
    if (totalElapsed > duration) {
      return;
    }
    $totalElapsed.textContent = totalElapsed;
    const elapsed = timestamp - then;

    if (elapsed >= fpsInterval) {
      // draw
      then = timestamp - (elapsed % fpsInterval);
      renderCount++;
      $render.textContent = renderCount;
      const transformStyle = target.style.transform;
      const translateX = transformStyle.replace(/[^\d.]/g, '');
      target.style.transform = `translateX(${+translateX + moveX}px)`;
    }
    requestAnimationFrame(cb);
  };
};

const $box = document.getElementById('box');
animate(makeCb($box, 2, 1000, 30));
```

위 코드를 이전 rAF예제와 비교해서 살펴보겠습니다.

1. `makeCb` 함수에서 구현하고자하는 `fps` 값을 추가적으로 파라미터로 받습니다.
2. `rAF`의 첫 콜백함수 시작 시점과의 시간차를 `totalElapsed`에 저장하여 갱신합니다. `totalElapsed`는 `duration`과 비교하여 `rAF`종료를 제어합니다.
3. `rAF`의 연속된 두 콜백 함수의 호출 시점의 시간차를 `elapsed`로 저장하여 갱신합니다. `elapsed`는 애니메이션 드로잉에 쓰로틀을 적용하기 위해 사용합니다.
4. `elapsed`는 현재 호출된 `rAF`의 콜백함수의 `timestamp`와 이전에 저장된 `then`이라는 시간값의 차이입니다. `elapsed`가 `fpsInterval`보다 크거나 같아지는 시점에서 애니메이션 드로잉을 해줍니다. 그렇지 않은 경우에는 드로잉을 하지않고 `rAF`를 재호출합니다.
   여기서 `then` 계산식에 대해서 좀 더 자세히 설명하겠습니다. `then`은 마지막 렌더링 때의 시간값을 저장합니다. `elasped`는 마지막 드로잉 때와의 시간간격입니다. 직전 드로잉 시점의 시간값과 현재 호출된 `rAF`의 콜백함수의 시간 간격을 쓰로틀 기준값과 비교해줍니다. 따라서 `then`에는 `timestamp`에 "`elapsed`를 `fpsInterval`로 나눈 나머지"를 빼준 값을 저장하고 다음 콜백함수에서 사용합니다.
   그런데 `then`에 그냥 드로잉 시점에 `rAF` 콜백함수의 `timestamp`를 저장하고 사용하면 되지않을까란 생각할 수 있습니다. 그러나 이 구현방식은 정확하지 않습니다.
   왜냐하면 단순히 `timestamp`를 `then`에 저장하면 elapsed에서 fpsInterval를 뺀 값이 축적되어 부정확한 계산을 하게됩니다. 따라서 `timestamp -( elasped - fpsInterval)`을 `then`에 저장해줘야합니다. 이때 `elapsed`가 계속 커지고 `fpsInterval`은 고정값이기 때문에 단순히 `elasped - fpsInterval`이 아니라 `elapsed % fpsInterval`로 계산해서 `then` 저장해야 다음번 콜백함수 호출시에 정확한 `elapse`d 계산이 가능합니다.

이 내용을 확인하기 쉽게 Node 환경에서 간단한 코드를 짜보겠습니다. Node 환경에서는 `performance`를 사용하기 위해 '`perf_hooks`'를 사용합니다. 또한 Node환경은 `requestAnimationFrame`을 지원하지 않지만 for문으로도 충분히 의도한 결과를 확인할 수 있습니다.

```javascript
const { performance } = require('perf_hooks');

const fpsInterval = 1000 / 30;
let then1 = performance.now();
let then2 = then1;

for (let i = 0; i < 100; i++) {
  const timestamp = performance.now();
  const elapsed1 = timestamp - then1;
  if (elapsed1 >= fpsInterval) {
    then1 = timestamp;
  }
  const elapsed2 = timestamp - then2;
  if (elapsed2 >= fpsInterval) {
    then2 = timestamp - (elapsed2 % fpsInterval);
  }
  console.log(elapsed1, elapsed2);
}
```

예제의 구현 결과는 아래 데모에서 확인할 수 있습니다.

https://codepen.io/younghwanjoe/pen/OJpGOZX

예제 화면에서 확인되듯이, 쓰로틀을 사용한 `FPS`를 구현한 애니메이션 화면에서는 그렇지 않은 `rAF` 애니메이션과 달리 모니터 주사율이 어떻든 같은 `FPS`를 나타냅니다. 여러분의 모니터가 `60Hz`나 `144Hz` 혹은 다른 주사율로 설정되어 있든지 똑같은 30FPS를 나타낼 것입니다.
다만 `render count`를 확인해보면 30이 아닌 29일 것입니다. 이는 시간값의 차이를 통해 쓰로틀을 사용하는 구현방식에서는 미묘한 오차가 발생할 수 있기 때문에 딱 떨어지는 fps 구현은 아닙니다. 아마 `rAF` 콜백함수 호출시 무조건 `render count`를 더해주면 방식으로 코드를 수정하면 `render count`를 30으로 맞출 수 있겠지만, 굳이 그렇기하지는 않았습니다.

지금까지 `requestAnimationFrame`의 개념과 `requsetAnimationFrame`을 사용하여 애니메이션을 구현하는 방법과 차이점에 대해서 설명했습니다.

```toc

```
