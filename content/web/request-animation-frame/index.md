---
emoji: ๐ฎ
title: requestAnimationFrame ํ์ฉํ ์ ๋๋ฉ์ด์ ๊ตฌํ
date: '2022-03-09 00:00:00'
author: YoungHwanJoe
tags: ์ต์ ํ
categories: web
---

์น ์ ๋๋ฉ์ด์์ ๊ตฌํํ๋ ๋ฐฉ์์ ํฌ๊ฒ `setInterval`์ ์ฌ์ฉํ๋ ๋ฐฉ์๊ณผ `requestAnimationFrame`์ ์ฌ์ฉํ๋ ๋ฐฉ์์ด ์์ต๋๋ค.๋ ๋ฐฉ์์ ๊ตฌํ ๋ฐฉ๋ฒ๊ณผ ์ฐจ์ด์ ์ ์์๋ณด๋๋ก ํ๊ฒ ์ต๋๋ค.

## setInterval์ ์ฌ์ฉํ ์น ์ ๋๋ฉ์ด์ ๊ตฌํ

์น์์ ์ ๋๋ฉ์ด์์ ๊ตฌํํ๋ ๋ฐฉ๋ฒ์์ ์ ํต์ ์ธ ๋ฐฉ์์ `setInterval`ํจ์๋ฅผ ์ฌ์ฉํ๋ ๊ฒ์๋๋ค.
์๋ฅผ ๋ค๋ฉด ๋ค์๊ณผ ๊ฐ์ต๋๋ค.

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

์ ์ฝ๋๋ฅผ ๋ถ์ํ๋ฉด ๋ค์๊ณผ ๊ฐ์ต๋๋ค.

1. ๋จผ์  `animate`ํจ์๋ฅผ ์ ์ํฉ๋๋ค. `animate` ํจ์๋ ์ฝ๋ฐฑํจ์์ ์ฝ๋ฐฑํจ์์ ํธ์ถ์ฃผ๊ธฐ, ๊ทธ๋ฆฌ๊ณ  ์ ๋๋ฉ์ด์์ ๊ตฌ๋ํ  ์ด ์๊ฐ์ ์ธ์๋ก ๋ฐ์ต๋๋ค.
2. `animate`ํจ์์ ์ ๋ฌํ  ์ฝ๋ฐฑํจ์๋ฅผ ๋ฐํํ๋ `makeCb`ํจ์๋ฅผ ์ ์ํฉ๋๋ค. ์ด๋ ์ปค๋งํจ์๋ฅผ ์ฌ์ฉํ์ฌ ์ฝ๋ฐฑํจ์์ ๋ด์ฉ์ด ๋ณํ๋๋ผ๋ ๊ฐ๊ฐ ๋ค๋ฅธ ์ฝ๋ฐฑํจ์๋ฅผ ์์ฑํด `animate`์ ์ ๋ฌํ  ์ ์๊ฒ ํ์ต๋๋ค. `makeCb`ํจ์๋ ์์ง์ด๋ ๋์์ด ๋๋ HTML ์๋ฆฌ๋จผํธ์ ๋ ๋๋ง์ ์์ง์ผ x์ขํ์์ ๊ฑฐ๋ฆฌ(px)๋ฅผ ์ธ์๋ก ๋ฐ์ต๋๋ค.
3. `animate` ํจ์์ (1) `$box`์๋ฆฌ๋จผํธ๋ฅผ x์ถ์ผ๋ก 2px์ฉ ์์ง์ด๊ฒํ๋ ์ฝ๋ฐฑํจ์์ (2) ๋ ๋๋ง ๊ฐ๊ฒฉ 10 ๋ฐ๋ฆฌ์ธ์ปจ๋, (3) ์ ๋๋ฉ์ด์ ์๊ฐ 1000 ๋ฐ๋ฆฌ์ธ์ปจ๋๋ฅผ ์ ๋ฌํ์ฌ ํธ์ถํฉ๋๋ค.
   ์ `animate` ํจ์๋ฅผ ์คํ์ํค๋ฉด, 1์ด๊ฐ 10๋ฐ๋ฆฌ ์ธ์ปจ๋ ๊ฐ๊ฒฉ์ผ๋ก `$box` ์๋ฆฌ๋จผํธ๋ฅผ x์ถ์ผ๋ก 2px์ฉ ์์ง์๋๋ค. ์ด๋ ๋ ๋๋ง์ ์ด 100(=1000/10)๋ฒ ์ผ์ด๋ฉ๋๋ค.
4. ์ดํด๋ฅผ ์ํด ๊ฒฝ๊ณผ ์๊ฐ(`elapsed`)๊ณผ ๋ ๋๋ง ํ์(`render count`)๋ฅผ ํจ๊ป ํ์ํ์ต๋๋ค.

๊ตฌํ ๊ฒฐ๊ณผ๋ ์๋ ๋ฐ๋ชจ์์ ํ์ธํ  ์ ์์ต๋๋ค.

https://codepen.io/younghwanjoe/pen/WNpWdZK

## requestAnimationFrame์ด๋?

์ด๋ฒ์๋ `requestAnimationFrame`(์ดํ `rAF`)๋ฅผ ์ฌ์ฉํ ์ ๋๋ฉ์ด์ ๊ตฌํ์ ๋ํด ์ค๋ชํ๊ฒ ์ต๋๋ค. ๋ค์ด๋ฐ์์ ์ ์ถํ  ์ ์๋ฏ์ด ์ ๋๋ฉ์ด์ ๊ตฌํ์ ์ํ ํจ์์๋๋ค. ํจ์ ๊ตฌ์กฐ๋ ๊ฐ๋จํฉ๋๋ค. ์ธ์๋ ํ๋๋ฅผ ๋ฐ๋๋ฐ, ๋ค์ ๋ ๋๋ง์์ ํธ์ถํ  ์ฝ๋ฐฑํจ์์๋๋ค. ๊ทธ๋ฆฌ๊ณ  ์ด ์ฝ๋ฐฑํจ์์๋ ํ๋์ ์ธ์๊ฐ ์ ๋ฌ๋ฉ๋๋ค. `DOMHighResTimeStamp`๋ผ๊ณ  ํ๋ ํ์์คํฌํ์๋๋ค.

์ฌ๊ธฐ์ `DOMHighResTimeStamp`๋ `timeorigin`์ ๊ธฐ์ค์ผ๋ก ์ผ๋ง๋ ์๊ฐ์ด ์ง๋ฌ๋์ง ๋ํ๋ด๋ ํ์์๋๋ค. `timeorigin`์ ๋ธ๋ผ์ฐ์ ์์ ์คํฌ๋ฆฝํธ๊ฐ ์์๋์๋๋ฅผ ๋งํฉ๋๋ค. ๋ฐ๋ผ์ rAF์ ์ฝ๋ฐฑํจ์์ ์ ๋ฌ๋๋ `DOMHighResTimeStamp`๋ ์คํฌ๋ฆฝํธ์ ์์์์ ์ ๊ธฐ์ค์ผ๋ก ์ฝ๋ฐฑํจ์ ํธ์ถ ์์  ์ฌ์ด์ ์๊ฐ์ฐจ์ด๋ฅผ ๋ํ๋๋๋ค. `window.performance.now()` ๋ฉ์๋ ํธ์ถ์์๋ `DOMHighResTimeStamp`๋ฅผ ๋ฐํํฉ๋๋ค. ์ฐธ๊ณ ๋ก ๋ธ๋ผ์ฐ์ ๊ฐ ์๋ Nodeํ๊ฒฝ์์๋ `rAF`๋ `window.performance`๋ ์ง์ํ์ง ์์ต๋๋ค. ๋ค๋ง `Node` ํ๊ฒฝ์ '`perf_hooks`' ๋ด์ฅ ๋ผ์ด๋ธ๋ฌ๋ฆฌ๋ฅผ ํตํด ๋ธ๋ผ์ฐ์ ์ `Web Performance APIs`์ `performance`์ ๊ฐ์ ๊ธฐ๋ฅ์ํ๋ ํจ์๋ฅผ ์ฌ์ฉํ  ์๋ ์์ต๋๋ค.
์ด์  ๊ฐ๋จํ๊ฒ `rAF`๋ฅผ ์๊ฐํ๊ณ  ์์์ `setInterval`์ ํตํ ์ ๋๋ฉ์ด์ ๊ตฌํ์ rAF๋ฅผ ์ฌ์ฉํด์ ๋น์ทํ๊ฒ ๊ตฌํํด๋ด๋๋ค. `rAF`๋ `setInterval`๊ณผ ๊ฐ์ด ์ฝ๋ฐฑ ํจ์์ ํธ์ถ ๊ฐ๊ฒฉ์ ๋ฐ๋ก ์ธ์๋ก ๋ฐ์ง ์์ผ๋ฏ๋ก ์ ๋๋ฉ์ด์ ์ง์์๊ฐ๋ง ๊ฐ๊ฒ ํ์ฌ ๊ตฌํํด๋ณด๊ฒ ์ต๋๋ค.

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

์ ์ฝ๋๋ฅผ ๋ถ์ํด๋ณด๋ฉด ๋ค์๊ณผ ๊ฐ์ต๋๋ค.

1. `setInterval`๊ณผ ๋ฌ๋ฆฌ `rAF`๋ ์ฝ๋ฐฑํจ์์ ํธ์ถ ์๊ฐ๊ฐ๊ฒฉ์ด๋ ์ง์์๊ฐ์ ๋ฐ๋ก ์ธ์๋ก ๋ฐ์ง ์์ต๋๋ค. ๋ฐ๋ผ์ `DOMHighResTimeStamp`ํ์์ `timestamp`๋ฅผ ํตํด `elapsed`๋ผ๋ ์๊ฐ์ฐจ์ด๋ฅผ ๊ณ์ฐํ์ฌ ์ฌ์ฉํฉ๋
2. ์ฒ์ `rAF`์ ์ฝ๋ฐฑ ํจ์๊ฐ ์์๋๋ ์์ ์ `timestamp`๋ฅผ `start`์ ์ ์ฅํ๊ณ  ์ดํ ์ฝ๋ฐฑ ํจ์ ํธ์ถ์๋ง๋ค `start`์์ ์๊ฐ ์ฐจ์ด๋ฅผ ๊ณ์ฐํ ๊ฐ์ธ `elapsed` ์ฌ์ฉํ์ฌ ๊ฒฝ๊ณผ์๊ฐ์ ์ธก์ ํฉ๋๋ค.
3. `duration`์ผ๋ก ์ฃผ์ด์ง 1000๋ฐ๋ฆฌ์ธ์ปจ๋๊ฐ `elapsed`๋ณด๋ค ์์ ๊ฒฝ์ฐ์๋ ํ๋ฉด์ ๋ณํ์์ผ์ฃผ๊ณ  `elapsed`๊ฐ ๊ทธ๋ณด๋ค ์ปค์ง๋ ์์ ์ rAF ์ฝ๋ฐฑํจ์ ํธ์ถ์ ๋ฉ์ถฅ๋๋ค.
4. `rAF`๊ฐ ํธ์ถํ๋ ์ฝ๋ฐฑํจ์์๋ ๋ฐ๋์ `rAF` ํธ์ถ๊ตฌ๋ฌธ์ ํฌํจํด์ผํฉ๋๋ค. `rAF`๋ ์ฌ๊ท์ ์ผ๋ก ์๋ํฉ๋๋ค.
5. `setInterval` ์์ ์ ๋ง์ฐฌ๊ฐ์ง๋ก ์ดํด๋ฅผ ์ํด ๊ฒฝ๊ณผ ์๊ฐ(`elapsed`)๊ณผ ๋ ๋๋ง ํ์(`render count`)๋ฅผ ํจ๊ป ํ์ํ์ต๋๋ค.

๊ตฌํ ๊ฒฐ๊ณผ๋ ์๋ ๋ฐ๋ชจ์์ ํ์ธํ  ์ ์์ต๋๋ค.

https://codepen.io/younghwanjoe/pen/XWMQeJd

## setInterval๊ณผ requestAnimationRequest ๋น๊ต

๋น๊ต๋ฅผ ์ํด ๋ ํจ์๋ฅผ ์ฌ์ฉํ ์ ๋๋ฉ์ด์์ ํ ํ๋ฉด์์ ๋์์ ์คํ์์ผ๋ดค์ต๋๋ค.

https://codepen.io/younghwanjoe/pen/qBrwPOm

`setInterval`์ ์ฌ์ฉํ ์ ๋๋ฉ์ด์์ด ์ ์ฉ๋ ๋นจ๊ฐ๋ฐ์ค์ `rAF`๋ฅผ ์ฌ์ฉํ ์ ๋๋ฉ์ด์์ด ์ ์ฉ๋ ์ด๋ก๋ฐ์ค๊ฐ ์์ต๋๋ค. ๋ ๋ฐ์ค๋ ๋์์ ์ถ๋ฐํด์ ์ ๋๋ฉ์ด์์ด ์์ํ์ง 1์ด๊ฐ ๋๋ ์์ ์ (์์ฃผ ๋ฏธ์ธํ ์ฐจ์ด๋ฅผ ๋ฌด์ํ๋ค๋ฉด) ๋์์ ๋ฉ์ถฅ๋๋ค. ๊ทธ๋ฌ๋ ์ฌ๋ฌ๋ถ์ด ๋ณด๋ ํ๋ฉด์์ ๋๊ฐ์ ๋ฐ์ค๋ ์๋ก ๋ฉ์ถ๋ ์ง์ (x์ถ ๊ธฐ์ค)์ **์๋ง** ์๋ก ๋ค๋ฅผ ๊ฒ์๋๋ค."**์๋ง**"๋ผ๋ ํํ์ ์ฌ์ฉํ ์ด์ ์ ๋ํด์๋ ์ ์ ํ์ ์ค๋ชํ๊ฒ ์ต๋๋ค.
์ ์์ ๋ค์ ๋ฐ๋ชจ ์๋จ์ ํ์๋๋ `setInterval`๊ณผ `rAF` ์์  ๋ฐ๋ชจ๋ค์ `render count`๋ค๋ ๋ค๋ฅผ ๊ฒ์๋๋ค. `rAF` ์์ ์์๋ ๋ ๋๋ง์ ํด์ฃผ๋ ์ฝ๋ฐฑ ํจ์์ ํธ์ถ ๊ฐ๊ฒฉ์ ๋ฐ๋ก ์ค์ ํ์ง ์์์ต๋๋ค. ๋ ์์ ์ ๋ ๋๋ง ํ์๊ฐ ๋ค๋ฅด๋ฏ๋ก ๋ ์์ ์ ๋ฐ์ค๋ค์ด ์์ง์ธ ๊ฑฐ๋ฆฌ๋ ๋ค๋ฅผ๊ฒ๋๋ค.

์ฌ๊ธฐ์ ์ ๊น ์๋์ ๊ฐ๋๋ค์ ์ดํด๋ณผ ํ์๊ฐ ์์ต๋๋ค.

## ํ๋ ์(Frame)๊ณผ ํ๋ ์ ๋ ์ดํธ(Frame rate) ๊ทธ๋ฆฌ๊ณ  ํค๋ฅด์ธ (Hz)

์๋๋ `frame rate`์ ๋ํ ์ํค๋ฐฑ๊ณผ์ ์ค๋ช์ ์ผ๋ถ ๊ฐ์ํ ๊ฒ์๋๋ค.

> ํ๋ ์ ๋ ์ดํธ(`frame rate`)๋ ๋์คํ๋ ์ด ์ฅ์น๊ฐ ํ๋ฉด ํ๋์ ๋ฐ์ดํฐ๋ฅผ ํ์ํ๋ ์๋๋ฅผ ๋งํ๋ค.
> ์ด์ ๊ฐ์ ๋ป์ผ๋ก ์ฐ์ด๋ ์ด๋น `frame` ์(`frames per second`)๋ 1์ด ๋์ ๋ณด์ฌ์ฃผ๋ ํ๋ฉด์ ์๋ฅผ ๊ฐ๋ฆฌํจ๋ค.
> ์ ๋๋ฉ์ด์์ ๋ณดํต ๋์ ์์์ ์ด์ฉํด์ ํ์ํ๊ธฐ ๋๋ฌธ์ 1์ด์ 30๋ฒ ๋๋ 25๋ฒ ์ด์์ด ํ์ํ๋, ๋ถ๋๋ฌ์ด ํ์๋ฅผ ์ํด์ 1์ด์ 60~120๋ฒ์ `frame`์ ํ์ํ๊ธฐ๋ ํ๋ค.

์ฐ๋ฆฌ๊ฐ ์ ๋๋ฉ์ด์์ ๋ณด๋ ๊ฒ์ ์งง์ ์๊ฐ ๊ฐ๊ฒฉ์ ์ด์ด์ง๋ ์ฅ๋ฉด์ ๋ณด๋ ๊ฒ๊ณผ ๊ฐ์ต๋๋ค. ์ด ๊ฐ๊ฐ์ ์ฅ๋ฉด์ด ๋ฐ๋ก `frame`์๋๋ค. ๊ทธ๋ฆฌ๊ณ  `frame rate`๋ ํน์  ์๊ฐ ๋ด์์ ๋ณด์ฌ์ง๋ `frame` ๊ฐฏ์๋ฅผ ๋ณด์ฌ์ฃผ๋ ์๋๋ฅผ ๋งํฉ๋๋ค. ํน์ ์๊ฐ์ ๋ณด์ฌ์ง๋ `frame`์ ๊ฐฏ์๋ผ๊ณ  ๋ด๋ ๋ฌด๋ฐฉํฉ๋๋ค. ๋ณดํต ๊ธฐ์ค์ 1์ด๋น ๋ณด์ฌ์ง๋ `frame` ๊ฐฏ์๋ก ์ฌ์ฉํ๋ฏ๋ก 1์ด๋น ๋ณด์ฌ์ง๋ `frame` ๊ฐฏ์๋ฅผ `FPS(Frame Per Second)`๋ก ํํํ๋ฉฐ ์ฐ๋ฆฌ๊ฐ ์ ์ผ ํํ ์ ํ๊ฒ ๋๋ ๋จ์์๋๋ค.

๋น์ทํ ์๋ฏธ์ `Hz`๋ผ๋ ๋จ์๋ ์์ต๋๋ค. ์ด๋ 1์ด ๋์์ ๋น๋๋ฅผ ๋ํ๋ด๋ ๋จ์์๋๋ค. `FPS`๋ `frame`์ด 1์ด๋์ ๋ช ๋ฒ์ผ์ด๋๋์ง๋ฅผ ๋ํ๋ด๋ ๋จ์์ด๊ณ , Hz๋ ๋ฌด์ธ๊ฐ๊ฐ 1์ด๋์ ๋ช ๋ฒ ์ผ์ด๋๋์ง๋ฅผ ๋ํ๋ด๋ ๋จ์์๋๋ค. ์ฌ์ค ์ฐ๋ฆฌ์๊ฒ๋ ๊ฐ์ ์๋ฏธ์๋๋ค. ๊ตณ์ด `FPS`๋ฅผ ์ค๋ชํ๊ณ  `Hz`๊น์ง ์ค๋ชํ๋ ์ด์ ๋ `rAF`์ ์ฝ๋ฐฑ ํจ์์ ํธ์ถ ๋น๋์์ ๋ ๋ค ์ฐ๊ด๋์ด ์๊ธฐ ๋๋ฌธ์๋๋ค.

์ ๋๋ฉ์ด์์ ๊ตฌํํ๋ ๊ฒ์ `frame rate`๋ฅผ ๊ธฐ์ค์ผ๋ก `frame`์ ์ฐ์์ ์ผ๋ก ์์ฑํ๋ ๊ฒ์ด๋ผ๊ณ  ๋ณผ ์ ์์ต๋๋ค. ์์์ ๊ตฌํํ ๋ ๊ฐ์ ์์ ์ฝ๋์์๋ ๋ชจ๋ `frame rate`๋ฅผ ์ง์  ์ ์ํ์ง๋ ์์์ต๋๋ค. ๋ค๋ง `setInterval`์ ์ฌ์ฉํ ์์ ์์๋ 1์ด(1000๋ฐ๋ฆฌ์ธ์ปจ๋) ๋์ 10๋ฐ๋ฆฌ์ธ์ปจ๋๋ง๋ค `frame`์ ์์ฑํ์ผ๋ฏ๋ก 100 FPS๋ผ๊ณ  ์ ์ํ๋ค๊ณ  ํ  ์ ์์ต๋๋ค.

์๊น `rAF` ์์ ๋ฅผ ๊ตฌํํ ๋์๋ ์๊ทผ์ฌ์ฉ ๋์ด๊ฐ์ง๋ง `rAF`์์๋ ์ ๋ฌ๋ ์ฝ๋ฐฑ ํจ์์ ํธ์ถ ๊ฐ๊ฒฉ๋ฅผ ์ด๋ป๊ฒ ์ ํ  ์ ์๋์ง์ ๋ํด ์ค๋ชํ์ง ์์์ต๋๋ค. `setInterval`์ ์ฌ์ฉํด๋ดค๋ค๋ฉด ๋น์ฐํ ์๋ฌธ์ ์ด ๋ค์์ ๊ฒ์๋๋ค. ์ด๋ `FPS`์ `Hz`์ ๋ํ ์ดํด๊ฐ ํ์ํ๊ธฐ ๋๋ฌธ์ ๊ทธ๋ ์ต๋๋ค. `FPS`์ `Hz`์ ๋ํด ์ค๋ชํ์ผ๋ ์ด์  `rAF`์ ์ฝ๋ฐฑํจ์์ ํธ์ถ ๊ฐ๊ฒฉ์ ์ด๋ป๊ฒ ์ ํ๋์ง ์์๋ณด๊ฒ ์ต๋๋ค.

## requestAnimationFrame์ ์ฝ๋ฐฑํจ์์ ํธ์ถ ํ์

๋ฐ๋ก ์ด ๋ถ๋ถ์ด ์ ๋๋ฉ์ด์์ ๊ตฌํํ๋๋ฐ์ ์์ด **setInterval๊ณผ rAF์ ๊ฒฐ์ ์ ์ธ ์ฐจ์ด์ **์๋๋ค.
`rAF`์ ์ฝ๋ฐฑํจ์ ํธ์ถ์ ๋ํ MDN ๋ฌธ์์ ์ค๋ช์ ๋ณด๊ฒ ์ต๋๋ค.

> The number of callbacks is usually 60 times per second, but will generally match the display refresh rate in most web browsers as per W3C recommendation.

`rAF`์ ์ฝ๋ฐฑ ํจ์๋ ๋๊ฐ 1์ด์ 60๋ฒ, ์ฆ `60FPS`๋ฅผ ๊ธฐ์ค์ผ๋ก ํธ์ถ๋ฉ๋๋ค. ๊ทธ๋ฌ๋ W3C ๊ถ์ฅ์ ๋ฐ๋ฅด๋ ๋๋ถ๋ถ์ ์น๋ธ๋ผ์ฐ์ ์์๋ ๋์คํ๋ ์ด์ `refresh rate`, ์ฆ ๋์คํ๋ ์ด์ `Hz`๋ฅผ ๋ฐ๋ฆ๋๋ค. ์ด๊ฒ์ด ๋ฌด์จ ๋ง์ผ๊น์?

์ผ๋ฐ์ ์ธ ๊ฐ์ ์์ ์ฌ์ฉํ๋ ๋ชจ๋ํฐ๋ ์๋น์๊ฐ `60Hz`๋ฅผ ์ง์ํฉ๋๋ค. ์๊น๋ ์ค๋ชํ๋ฏ์ด `60Hz`๋ ๊ณง `60FPS`๋ผ๊ณ  ํ  ์ ์์ต๋๋ค. ๋ฐ๋ผ์ `60Hz`๋ฅผ ์ง์ํ๋ ๋ชจ๋ํฐ๋ฅผ ์ฌ์ฉํ๋ ๋ธ๋ผ์ฐ์ ์์ `rAF`๋ `60FPS`๋ฅผ ๊ธฐ์ค์ผ๋ก ์ฝ๋ฐฑ ํจ์๋ฅผ ํธ์ถํฉ๋๋ค. ๊ทธ๋์ ์ MDN ๋ฌธ์์์๋ ๋๊ฐ `rAF`์ ์ฝ๋ฐฑํจ์๊ฐ 1์ด์ 60๋ฒ ํธ์ถ๋๋ค๊ณ  ํ ๊ฒ์๋๋ค.

๊ทธ๋ ๋ค๋ฉด `100Hz`๋ฅผ ์ง์ํ๋ ๋ชจ๋ํฐ์์๋ `rAF`์ ์ฝ๋ฐฑํจ์๋ 1์ด๋์ ๋ช๋ฒ ํธ์ถ๋ ๊น์? ๋น์ฐํ 100๋ฒ์๋๋ค. `100FPS`๋ฅผ ๊ธฐ์ค์ผ๋ก ํธ์ถ๋๋ ๊ฒ๊ณผ ๊ฐ์ผ๋๊น์.

์ด ๋ถ๋ถ์ด `setInterval`๊ณผ `rAF`์ ์ ๋๋ฉ์ด์ ๊ตฌํ์ ๊ฐ์ฅ ํฐ ์ฐจ์ด์ ์๋๋ค. `setInterval`์์๋ ์ฝ๋ฐฑ ํจ์ ํธ์ถ ๊ฐ๊ฒฉ์ ์๊ฐ์ผ๋ก ์กฐ์ ํจ์ผ๋ก์จ `FPS`๋ฅผ ์ค์ ํ์ง๋ง, `rAF`์์๋ ๋ชจ๋ํฐ์ ์ฃผ์ฌ์จ์ ๋ฐ๋ฅด๊ฒ ๋ฉ๋๋ค.

๋ฐ๋ผ์ `setInterval`๋ก ๊ตฌํํ ์ ๋๋ฉ์ด์์ ๋ชจ๋ํฐ ์ฃผ์ฌ์จ์ด ๋ฌ๋ผ์ ธ๋ `FPS`๊ฐ ๋์ผํ์ง๋ง, `rAF`๋ก ๊ตฌํํ ์ ๋๋ฉ์ด์์ ๋ชจ๋ํฐ ์ฃผ์ฌ์จ์ ๋ฐ๋ผ `FPS`๊ฐ ๋ฌ๋ผ์ง๋๋ค.

์์์ "์ฌ๋ฌ๋ถ์ด ๋ณด๋ ํ๋ฉด์์ ๋๊ฐ์ ๋ฐ์ค๋ ์๋ก ๋ฉ์ถ๋ ์ง์ (x์ถ ๊ธฐ์ค)์ **์๋ง** ์๋ก ๋ค๋ฅผ ๊ฒ์๋๋ค"๋ผ๊ณ  ํ ์ด์ ๊ฐ ๋ฐ๋ก ์ด๊ฒ์๋๋ค. ์ฌ๋ฌ๋ถ์ ์๋ง `60Hz`๋ `144Hz` ๋ฑ์ ์ฃผ์ฌ์จ์ ๊ฐ์ง ๋ชจ๋ํฐ๋ก ํ๋ฉด์ ๋ดค์ ๊ฒ์๋๋ค(`100Hz`๋ก ๋ชจ๋ํฐ๋ฅผ ์ฌ์ฉํ๋ ๊ฒฝ์ฐ๋ ํ์น ์์ ๊ฒ๋๋ค). ์ด ๊ฒฝ์ฐ์ ๋ ๊ฐ์ ๋ฐ์ค๊ฐ ๋ํ๋๋ ์์  ํ๋ฉด์์ 1์ด๋์์ ๋ ๋ฐ์ค ์ด๋ ๊ฑฐ๋ฆฌ๋ ์๋ก ๋ฌ๋์ ๊ฒ์๋๋ค. ๋ ์ ๋๋ฉ์ด์์์์ FPS๊ฐ ์๋ก ๋ฌ๋์ํ๋๊น์. ๋ง์ฝ `100Hz` ์ฃผ์ฌ์จ์ ๋ชจ๋ํฐ๋ฅผ ์ฌ์ฉํ๊ฑฐ๋, ์ฃผ์ฌ์จ ๋ณ๊ฒฝ์ด ๊ฐ๋ฅํ ๋ชจ๋ํฐ์์ `100Hz`๋ก ์ฃผ์ฌ์จ์ ๋ณ๊ฒฝํ๊ณ  ๋ ๋ฐ์ค์ ์ด๋ ํ๋ฉด์ ๋ณด๊ฒ ๋๋ค๋ฉด, ๋ ๋ฐ์ค๋ 1์ด ๋์ ๋์ผํ ๊ฑฐ๋ฆฌ๋งํผ ์์ง์ผ ๊ฒ์๋๋ค.

๊ทธ๋ ๋ค๋ฉด `rAF`์์ ๋ชจ๋ํฐ ์ฃผ์ฌ์จ๊ณผ ๋ฌด๊ดํ๊ฒ ํน์  `FPS`๋ฅผ ๊ตฌํํ๋ ค๋ฉด ์ด๋ป๊ฒ ํ ๊น์? `setInterval`์ฒ๋ผ ์๊ฐ ๊ฐ๊ฒฉ์ ์ฃผ๋ฉด ๋  ๊ฒ ๊ฐ์ต๋๋ค. ํ์ง๋ง `setInterval`์ ์ฌ์ฉํ ๋ ๋ณด๋ค ์กฐ๊ธ ๋ณต์กํ ๊ตฌํ์ด ํ์ํฉ๋๋ค. ์ฌ๊ธฐ์ **์ฐ๋กํ(throttle)**์ ์ฌ์ฉํฉ๋๋ค.

## ์ฐ๋กํ(throttle)์ ์ฌ์ฉํ์ฌ rAF์์ FPS ๊ตฌํํ๊ธฐ

**์ฐ๋กํ(throttle)**์ `๋๋ฐ์ด์ค(debounce)`์ ํจ๊ป ์ฐ์๋ ๋ธ๋ผ์ฐ์  ์ด๋ฒคํธ ์ฒ๋ฆฌ ๊ธฐ๋ฒ์ผ๋ก ๋ง์ด ์ธ๊ธ๋ฉ๋๋ค. ์ด ๋์ ์ฐ์๋ ์ด๋ฒคํธ์ ๋ํด ์ด๋ฒคํธ ๋ฆฌ์ค๋๊ฐ ๋ฑ๋ก๋์ด ์์๋ ์ด๋ฒคํธ ๋ฆฌ์ค๋์ ๋ถํ์ํ ์ฝ๋ฐฑํธ์ถ์ ์ค์ด๊ฑฐ๋ ํ๊ธฐํจ์ผ๋ก์จ ๋ฆฌ์์ค ์ฌ์ฉ์ ์ค์ด๋ ๊ธฐ๋ฒ์๋๋ค.

์ฌ๊ธฐ์์๋ FPS๊ตฌํ์ ์ํ ์ฐ๋กํ์ ์ฃผ๋ก ๋ค๋ฃฐ ๊ฒ์ด๋ฏ๋ก `๋๋ฐ์ด์ค`์ ๋ํ ์์ธํ ์ค๋ช์ ์๋ตํ๊ณ  ์ฐ๋กํ ๊ธฐ๋ฒ์ ๋ํ ๊ฐ๋จํ ์ค๋ช๋ง ํ๊ฒ ์ต๋๋ค. **์ฐ๋กํ์ ์ฐ์๋ ์ด๋ฒคํธ๋ฅผ ๊ทธ๋ฃนํํด์ ์ผ์  ์๊ฐ๊ฐ๊ฒฉ๋์ ํ๋ฒ๋ง ์ด๋ฒคํธ ์ฒ๋ฆฌ**๋ฅผ ํ๋ ๊ฒ์๋๋ค. ์ด ๊ฐ๋์ FPS ๊ตฌํ์ ์ ์ฉํด ๋ณธ๋ค๋ฉด 1์ด(์ผ์  ์๊ฐ๊ฐ๊ฒฉ)๋์ **FPS์ ๊ฐ๊ฒฉ**๋งํผ ๋ ๋๋ง(์ด๋ฒคํธ ์ฒ๋ฆฌ)๋ฅผ ํด์ฃผ๋ ๊ฒ์ผ๋ก ์ดํดํ  ์ ์์ต๋๋ค.

์ฌ๊ธฐ์ **FPS ๊ฐ๊ฒฉ**์ด๋ ์ฉ์ด๋ฅผ ์ฌ์ฉํ์ต๋๋ค. ์ด๋ ์ ์์ ์ธ ๊ธฐ์ ์ฉ์ด๋ ์๋๊ณ  ์ดํด๋ฅผ ๋๊ธฐ ์ํด ์ฌ์ฉํ ์ฉ์ด์๋๋ค. ์์์ ์ดํด๋ณธ๋๋ก `FPS`๋ 1์ด๋์ ๋ณด์ฌ์ง๋ `frame`์ ๊ฐฏ์์๋๋ค. ๊ทธ๋ฆฌ๊ณ  1์ด๋ 1000 ๋ฐ๋ฆฌ์ธ์ปจ๋์๋๋ค. ์ด๋ FPS๋ฅผ 100์ผ๋ก ์ค์ ํ๋ ค๊ณ  ํ๋ค๋ฉด 1000/100 ๋ฐ๋ฆฌ์ธ์ปจ๋๋ง๋ค ๋๋ก์์
ํด์ฃผ๋ ๊ฒ๊ณผ ๊ฐ์ต๋๋ค.

์ฌ๊ธฐ์ ํ๊ฐ์ง ์ฃผ์ํ  ์ ์, ๋ชจ๋ํฐ ์ฃผ์ฌ์จ๋ณด๋ค ๋์ `FPS`๋ฅผ ๊ตฌํํ  ์๋ ์๋ค๋ ์ ์๋๋ค. `60Hz`์ฃผ์ฌ์จ์ผ๋ก ์ค์ ๋ ๋ชจ๋ํฐ์์ `60FPS`์ด์์ ๊ตฌํํ  ์๋ ์์ต๋๋ค.

> ์ฆ, ์ฐ๋กํ์ ํตํด ๊ตฌํํ๋ `FPS`๋ ์ฌ์ฉํ๋ ๋ชจ๋ํฐ์ ์ฃผ์ฌ์จ๋ณด๋ค ํด ์๋ ์์ต๋๋ค.

`rAF`์ ์ฝ๋ฐฑ ํจ์์ ํธ์ถ๊ณผ ๋๋ก์์ด ๋ฐ๋์ ํจ๊ป ์ผ์ด๋์ง๋ ์์ต๋๋ค. `์ฐ๋กํ`์ ํตํ `FPS` ๊ตฌํ์ `rAF`์ ์ฝ๋ฐฑํจ์ ํ์์๋ ์ฐ๊ด์ด ์์ต๋๋ค. ์๊น๋ ์ดํด๋ดค๋ฏ์ด, `rAF`์ ์ฝ๋ฐฑํจ์๋ ์ฃผ์ฌ์จ์ ๋ฐ๋ผ ๊ฒฐ์ ๋๋ ๊ฒ์ด๊ณ  ์ด๊ฒ์ ์ฐ๋ฆฌ๊ฐ ์ ์ดํ๋ ค๋ ๊ฒ์ด ์๋๋๋ค. ์ฐ๋ฆฌ๊ฐ `FPS`๋ฅผ ๊ตฌํํ๋ ๋ฐฉ์์ `rAF`์ ์ฝ๋ฐฑ ํจ์ ๋ด์ ๋๋ก์ ๋ก์ง, ์ฆ ๋ธ๋ผ์ฐ์ ๋ฅผ ๊ทธ๋ฆฌ๋ ์ฝ๋๊ฐ ์คํ๋๋ ํ์๋ฅผ ์กฐ์ ํ๋ ๊ฒ์๋๋ค. ๋๋ฌธ์ ๋๋ก์ ํ์๋ฅผ ์กฐ์ ํด๋ 1์ด๊ฐ์ ์ต๋ ๋๋ก์ ํ์๋ 1์ด๊ฐ์ `rAF`์ ์ฝ๋ฐฑ ํจ์๊ฐ ํธ์ถ๋๋ ํ์, ์ฆ ๋ชจ๋ํฐ์ ์ฃผ์ฌ์จ๋ณด๋ค ํด ์ ์์ต๋๋ค.

๊ทธ๋ผ `rAF`๋ฅผ ์ฌ์ฉํ ์์ ์์ `FPS`๋ฅผ ์ง์  ์ค์ ํ๋๋ก ๊ตฌํํด๋ณด๊ฒ ์ต๋๋ค. ๋๊ฐ `60Hz`์ ๋ชจ๋ํฐ๋ฅผ ๋ง์ด ์ฌ์ฉํ๋ฏ๋ก ์ด๋ณด๋ค ๋ฎ์ 30 FPS์ ์ค์ ํด๋ณด๋๋ก ํ์ฃ .

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

์ ์ฝ๋๋ฅผ ์ด์  rAF์์ ์ ๋น๊ตํด์ ์ดํด๋ณด๊ฒ ์ต๋๋ค.

1. `makeCb` ํจ์์์ ๊ตฌํํ๊ณ ์ํ๋ `fps` ๊ฐ์ ์ถ๊ฐ์ ์ผ๋ก ํ๋ผ๋ฏธํฐ๋ก ๋ฐ์ต๋๋ค.
2. `rAF`์ ์ฒซ ์ฝ๋ฐฑํจ์ ์์ ์์ ๊ณผ์ ์๊ฐ์ฐจ๋ฅผ `totalElapsed`์ ์ ์ฅํ์ฌ ๊ฐฑ์ ํฉ๋๋ค. `totalElapsed`๋ `duration`๊ณผ ๋น๊ตํ์ฌ `rAF`์ข๋ฃ๋ฅผ ์ ์ดํฉ๋๋ค.
3. `rAF`์ ์ฐ์๋ ๋ ์ฝ๋ฐฑ ํจ์์ ํธ์ถ ์์ ์ ์๊ฐ์ฐจ๋ฅผ `elapsed`๋ก ์ ์ฅํ์ฌ ๊ฐฑ์ ํฉ๋๋ค. `elapsed`๋ ์ ๋๋ฉ์ด์ ๋๋ก์์ ์ฐ๋กํ์ ์ ์ฉํ๊ธฐ ์ํด ์ฌ์ฉํฉ๋๋ค.
4. `elapsed`๋ ํ์ฌ ํธ์ถ๋ `rAF`์ ์ฝ๋ฐฑํจ์์ `timestamp`์ ์ด์ ์ ์ ์ฅ๋ `then`์ด๋ผ๋ ์๊ฐ๊ฐ์ ์ฐจ์ด์๋๋ค. `elapsed`๊ฐ `fpsInterval`๋ณด๋ค ํฌ๊ฑฐ๋ ๊ฐ์์ง๋ ์์ ์์ ์ ๋๋ฉ์ด์ ๋๋ก์์ ํด์ค๋๋ค. ๊ทธ๋ ์ง ์์ ๊ฒฝ์ฐ์๋ ๋๋ก์์ ํ์ง์๊ณ  `rAF`๋ฅผ ์ฌํธ์ถํฉ๋๋ค.
   ์ฌ๊ธฐ์ `then` ๊ณ์ฐ์์ ๋ํด์ ์ข ๋ ์์ธํ ์ค๋ชํ๊ฒ ์ต๋๋ค. `then`์ ๋ง์ง๋ง ๋ ๋๋ง ๋์ ์๊ฐ๊ฐ์ ์ ์ฅํฉ๋๋ค. `elasped`๋ ๋ง์ง๋ง ๋๋ก์ ๋์์ ์๊ฐ๊ฐ๊ฒฉ์๋๋ค. ์ง์  ๋๋ก์ ์์ ์ ์๊ฐ๊ฐ๊ณผ ํ์ฌ ํธ์ถ๋ `rAF`์ ์ฝ๋ฐฑํจ์์ ์๊ฐ ๊ฐ๊ฒฉ์ ์ฐ๋กํ ๊ธฐ์ค๊ฐ๊ณผ ๋น๊ตํด์ค๋๋ค. ๋ฐ๋ผ์ `then`์๋ `timestamp`์ "`elapsed`๋ฅผ `fpsInterval`๋ก ๋๋ ๋๋จธ์ง"๋ฅผ ๋นผ์ค ๊ฐ์ ์ ์ฅํ๊ณ  ๋ค์ ์ฝ๋ฐฑํจ์์์ ์ฌ์ฉํฉ๋๋ค.
   ๊ทธ๋ฐ๋ฐ `then`์ ๊ทธ๋ฅ ๋๋ก์ ์์ ์ `rAF` ์ฝ๋ฐฑํจ์์ `timestamp`๋ฅผ ์ ์ฅํ๊ณ  ์ฌ์ฉํ๋ฉด ๋์ง์์๊น๋ ์๊ฐํ  ์ ์์ต๋๋ค. ๊ทธ๋ฌ๋ ์ด ๊ตฌํ๋ฐฉ์์ ์ ํํ์ง ์์ต๋๋ค.
   ์๋ํ๋ฉด ๋จ์ํ `timestamp`๋ฅผ `then`์ ์ ์ฅํ๋ฉด elapsed์์ fpsInterval๋ฅผ ๋บ ๊ฐ์ด ์ถ์ ๋์ด ๋ถ์ ํํ ๊ณ์ฐ์ ํ๊ฒ๋ฉ๋๋ค. ๋ฐ๋ผ์ `timestamp -( elasped - fpsInterval)`์ `then`์ ์ ์ฅํด์ค์ผํฉ๋๋ค. ์ด๋ `elapsed`๊ฐ ๊ณ์ ์ปค์ง๊ณ  `fpsInterval`์ ๊ณ ์ ๊ฐ์ด๊ธฐ ๋๋ฌธ์ ๋จ์ํ `elasped - fpsInterval`์ด ์๋๋ผ `elapsed % fpsInterval`๋ก ๊ณ์ฐํด์ `then` ์ ์ฅํด์ผ ๋ค์๋ฒ ์ฝ๋ฐฑํจ์ ํธ์ถ์์ ์ ํํ `elapse`d ๊ณ์ฐ์ด ๊ฐ๋ฅํฉ๋๋ค.

์ด ๋ด์ฉ์ ํ์ธํ๊ธฐ ์ฝ๊ฒ Node ํ๊ฒฝ์์ ๊ฐ๋จํ ์ฝ๋๋ฅผ ์ง๋ณด๊ฒ ์ต๋๋ค. Node ํ๊ฒฝ์์๋ `performance`๋ฅผ ์ฌ์ฉํ๊ธฐ ์ํด '`perf_hooks`'๋ฅผ ์ฌ์ฉํฉ๋๋ค. ๋ํ Nodeํ๊ฒฝ์ `requestAnimationFrame`์ ์ง์ํ์ง ์์ง๋ง for๋ฌธ์ผ๋ก๋ ์ถฉ๋ถํ ์๋ํ ๊ฒฐ๊ณผ๋ฅผ ํ์ธํ  ์ ์์ต๋๋ค.

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

์์ ์ ๊ตฌํ ๊ฒฐ๊ณผ๋ ์๋ ๋ฐ๋ชจ์์ ํ์ธํ  ์ ์์ต๋๋ค.

https://codepen.io/younghwanjoe/pen/OJpGOZX

์์  ํ๋ฉด์์ ํ์ธ๋๋ฏ์ด, ์ฐ๋กํ์ ์ฌ์ฉํ `FPS`๋ฅผ ๊ตฌํํ ์ ๋๋ฉ์ด์ ํ๋ฉด์์๋ ๊ทธ๋ ์ง ์์ `rAF` ์ ๋๋ฉ์ด์๊ณผ ๋ฌ๋ฆฌ ๋ชจ๋ํฐ ์ฃผ์ฌ์จ์ด ์ด๋ป๋  ๊ฐ์ `FPS`๋ฅผ ๋ํ๋๋๋ค. ์ฌ๋ฌ๋ถ์ ๋ชจ๋ํฐ๊ฐ `60Hz`๋ `144Hz` ํน์ ๋ค๋ฅธ ์ฃผ์ฌ์จ๋ก ์ค์ ๋์ด ์๋ ์ง ๋๊ฐ์ 30FPS๋ฅผ ๋ํ๋ผ ๊ฒ์๋๋ค.
๋ค๋ง `render count`๋ฅผ ํ์ธํด๋ณด๋ฉด 30์ด ์๋ 29์ผ ๊ฒ์๋๋ค. ์ด๋ ์๊ฐ๊ฐ์ ์ฐจ์ด๋ฅผ ํตํด ์ฐ๋กํ์ ์ฌ์ฉํ๋ ๊ตฌํ๋ฐฉ์์์๋ ๋ฏธ๋ฌํ ์ค์ฐจ๊ฐ ๋ฐ์ํ  ์ ์๊ธฐ ๋๋ฌธ์ ๋ฑ ๋จ์ด์ง๋ fps ๊ตฌํ์ ์๋๋๋ค. ์๋ง `rAF` ์ฝ๋ฐฑํจ์ ํธ์ถ์ ๋ฌด์กฐ๊ฑด `render count`๋ฅผ ๋ํด์ฃผ๋ฉด ๋ฐฉ์์ผ๋ก ์ฝ๋๋ฅผ ์์ ํ๋ฉด `render count`๋ฅผ 30์ผ๋ก ๋ง์ถ ์ ์๊ฒ ์ง๋ง, ๊ตณ์ด ๊ทธ๋ ๊ธฐํ์ง๋ ์์์ต๋๋ค.

์ง๊ธ๊น์ง `requestAnimationFrame`์ ๊ฐ๋๊ณผ `requsetAnimationFrame`์ ์ฌ์ฉํ์ฌ ์ ๋๋ฉ์ด์์ ๊ตฌํํ๋ ๋ฐฉ๋ฒ๊ณผ ์ฐจ์ด์ ์ ๋ํด์ ์ค๋ชํ์ต๋๋ค.

```toc

```
