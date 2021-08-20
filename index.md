## Welcome to RxJS Notes

This page is aims to share my knowlege about rxjs technics. 

### Technic1: 
1. From the array, each item is used as a part of url  to get the first result through async communication
2. Result of the first async communication, it is used as part or url to get the second result from remote server
3. With the second result, get the image url.

![](/assets/images/technic1-1.png)

[Detail diagram](/technic1.md)


### Technic2:
1. 3 step nested object, parent has children (nested objects) and children has childrend (nested objects)
2. There are many parents, 
3. Grand children has a image address
4. Finally, image addresses need to be saved as a file, which has all the relation from parent to child,\
   to local stoarage with the name of parent id. 

![](/assets/images/technic2-1.png)

[Detail diagram](/technic2.md)

### Technic3:
1. User can select randomly, one grid, two or three or four.
2. One grid type is selected, then html element id is element1, which can be a canvas area for drawing.
3. Two grids type are selected, then html elements are element1 element2, each elements is inputed to next process sequencially.
4. Two grids type are selected, process of element2 have to wait until element1 is completing rendering process.
5. Each split window has multiple images, which is got from server by async communication.
6. Each split window display only one image and others are cached
7. When multi grid type is selected, just the time the previous split window start to cache after display the first image, next split window start to rendering process.

![](/assets/images/split-window1-1.png)

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





