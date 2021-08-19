## Welcome to William's Notes

This page is aims to share my knowlege about rxjs techics. 

### Technic1: 
1. From the array, each item is used as a part of url  to get the second result through async communication
2. Result of the first http communication, it is used as part or url to get the second result from remote server
3. With the second result, get the image url.

![](/assest/images/technic1-1.png)

[Detail diagram](/technic1.md)


### Technic2:
다수의 3차원 object를, 1차원 object(parent)를 기준으로 nested object(children), nested object (grand childrend)안의 각 object의 키 값으로 해서 3차원 배열을 만든 다음에 이 배열에 값을 저장하여, 최종적으로 그것을 물리적 저장 장치에 파일로 저장한다. 각 차원의 object는 1:n의 구조로 다음 그림과 같다.

data#1, data#2 … data#n으로 다수가 존재하고 이 data의 갯수만큼 파일이 저장된다.
각 data 에는 다수의 item이 존재하고, 각 item은 다수의 sub item이 존재한다.
각 sub item은 각 image 정보를 가지고 있다.
[Detail diagram](/technic2.md)

![](/assest/images/technic2-1.png)

### Technic3:
User can select randomly, one grid, two or three or four.
One grid, then html element id is element1, which can be a canvas area for drawing.
Two grids, then html elements are element1 element2, each elements is inputed to next process sequencially.
If two grids slected, process of element2 have to wait until element1 is completing process.
Each sprit window has multiple images, which is got from server by async communication.
Each sprit window display one page of ct-image and others are cached or in the middle of caching process.
Just the time the previous split window start to cache, next split window start to rendering.

![](/assest/images/split-window1-1.png)

[Detail Diagram](/technic3.md)

### High quality RxJS articles

1. [RxJS Cached](https://blog.thoughtram.io/angular/2018/03/05/advanced-caching-with-rxjs.html )
2. [Creating Custom Operators](https://netbasal.com/creating-custom-operators-in-rxjs-32f052d69457)
3. [Partition, shareReplay, concatMapTo](https://netbasal.com/use-rxjs-to-modify-app-behavior-based-on-page-visibility-ce499c522be4)
4. [Reactive Sticky Header in Angular](https://netbasal.com/reactive-sticky-header-in-angular-12dbffb3f1d3)
5. [Reactive Search Feature with Angular](https://medium.com/lapis/searching-through-a-list-reactively-in-angular-c61c9d1832df)
6. [Observableinput](https://medium.com/javascript-everyday/rxjs-observableinput-dbc9c7035adc)
7. [15 Technics](https://sentinelone-tech.medium.com/15-rxjs-awesome-tips-from-15-sentinels-84ad132b13fd)
8. [3 Ways To Handle Errors in RxJS](https://medium.com/javascript-in-plain-english/3-ways-to-handle-errors-in-rxjs-97a04f2ecdc)
9. [Multicast](https://netbasal.com/understanding-rxjs-multicast-operators-77b3f60af0a2)
10. [Defer](https://netbasal.com/getting-to-know-the-defer-observable-in-rxjs-a16f092d8c09)





