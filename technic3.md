### Condition
1. The user can select the grid type randomly: one, two, three, or four grids.
2. If grid type1 is selected, then the html element id is element1, which can be a canvas area for drawing.
3. If grid type2 is selected, then the html elements are element1 element2, each element is passed on to the next process sequentially.
4. If grid type2 is selected, the process of element2 has to wait until the rendering process or element1 is complete.
5. Each split window has multiple images, which is obtained from the server by async communication.
6. Each split window displays only one image and others are cached.
7. When multi grid type is selected, while the previous split window starts to cache after displaying the first image, the next split window starts the rendering process.

![image1](assets/images/split-window1.png)

Each split window has its own element id, like ( element1, element2, element3, element4)

- case 1: One split window (Grid type 1),
	With defined grid size, template and element id, start image rendering.
- case 2: Multi split window. (Grid type 2, Grid type 3, Grid type 4)
	1. With defined grid size, template and element id, start rendering from the first split window.
	 2. The next split window waits until the previous split window completes rendering.

---

![image2](assets/images/split-window2.png)

Using zip operator (rxjs) to wait until the next process is complete.

- case1: One split window.  
1. Above diagram, step 3 is already made, step 4 is made just the time when grid type1 is selected.

- case2: Multi split window.
1. isStartedRendering: status of starting rendering. Should check if element id of previous split window is the same with id of isFinishedRendering$ value. 
2. isFinishedRendering: status of complete rendering and related side job. 
3. above step 1. and step 2. job is completed.
4. When user select grid type, create this observable for waiting above step 3. 
5. When step3 and step4 is completing, it means one of next split window processing is ready to start.

---
![image3](assets/images/split-window3.png)

1. Start with new html element id
2. currentCtViewerElementId$: Observable that gets the signal of start or end rendering.
3. isStarted$
4. isFinished$
5. Check if this is the final process of grid no. If not, continue next split window process. 
6. End of rendering split window 

---

```ts
    renderingSplitWindows() {
        const isFinished$ = this.currentCtViewerElementId$.pipe(     // 1
            switchMap(val => {
                this.selectedElementId = val.selectedElementId;     // 2
                return this.ctService.isFinishedRendering$[this.selectedElementId]
                    .pipe(take(1)); 				// 3
            }),
            takeUntil(this.unsubscribe$)
        );

        const isStarted$ = this.currentCtViewerElementId$.pipe(      // 4
            switchMap(val => {
                this.selectedElementId = val.selectedElementId;
                return this.ctService.isStartedRendering$[this.selectedElementId]
                    .pipe(take(1));
            }),
            takeUntil(this.unsubscribe$)
        );
        // 
        if ( gridNo > 1 ) {
            if (this.selectedElementId === 'element1') {         // 5
                this.tempObservable = defer(() => of(EMPTY).pipe());
            } else if (this.selectedElementId === 'element2') {
                this.tempObservable = zip(isStarted$, isFinished$).pipe(  //['element2','element1']
                    filter(val => val[1] === 'element1'),        // 6
                );
            } else if (this.selectedElementId === 'element3') {
                this.tempObservable = zip(isStarted$, isFinished$).pipe( //['element3','element2']
                    filter(val => val[1] === 'element2'),
                );
            } else if (this.selectedElementId === 'element4') {
                this.tempObservable = zip(isStarted$, isFinished$).pipe( //['element4','element3']
                    filter(val => val[1] === 'element3'),
                );
            }
        } else { // only one split window
            this.tempObservable = defer(() => of(EMPTY).pipe());    // 7
        }
    }

```

```ts
    private initialize() {
        const rendering$: Observable<any> = this.requestSplitWindow$[this.selectedElementId];
                                                                    // 8
        zip(this.tempObservable, rendering$).pipe(   // 9
            take(1),
            tap((val) => {
                const data = {selectedElementId: val[1]};
                /** Forward this to Webviewer-vertical.component.
                 * Where this value is used to call function "getNodules" */
                this.store.dispatch(new SetSelectedElementId(data));
            })
        ).subscribe((val) => {
            this.renderingCtViewerSplitWindow(val[1]);              // 10
        });
  }
```
showSelectedSeriesToViewer
	this.store.dispatch(new SetSelectedCtViewer(data)) --> currentCtViewerElementId$
	
1. currentCtViewerElementId$, when process arrive at the proper position of checking, 
2. then pass current element id, and make observable 
3. and wait previous process ending.
4. Same as above step 1,2,3 except waiting the process reach the start state.
5. If multiple grid type is selected, start to check from first grid.
6. Because this is the second split window, element of previouse split window must be element1. (isFinished$ === element1, isStarted$ === element2), 
7. When Grid type1 is selected (one split window), there is no need to step5 and step 6.
8. Make observable for each element for waiting rendering previous split window, which can be used parameter of zip operator (rxjs),  
9. rendering$ and tempObservable can be the signal of rendering split window by the zip operator. 
10. Start processing ct-viewer after finished processing for previous split window\
    val[1] may be 'element1' if Grid type1 is selected, 'element2' if Grid type2 is selected. 
