

![image1](assets/images/split-window1.png)
For each split window has it's own element id like as ( element1, element2, element3, element4)
following snippet replace it as selectedElementId

- case 1: One split window,
	When get the grid size, start to image rendering.
- case 2: Multi split window. (2 or 3 or 4)
	1. When get the grid size, start to rendering from the first split window.
	 2. The next split window wait until the previous split window completing rendering.

---

![image2](assets/images/split-window2.png)

Using zip operator (rxjs) to wait the next process complete.
1. isStartedRendering: status of after initial setting for starting rendering.
2. isFinishedRendering: status of complete rendering and related side job. 
3. above step 1. and step 2. job is completed.
4. When user select grid type, create this observable for wating above step 3. 
5. After step3 and step4 is completing, one of split window processing is completed.

```ts
    private renderingSplitWindows() {
        /**
         * When it comes to rendering of split-windows,
         * each window need to wait until the previous window finished rendering.
         * -----------------
         * 1. The end of redering process of the first image, emit event of "isStartedRendering$" for each split window.
         * 2. As soon as take the event of "isStartedRendering$" start processing some functions
         * 3. After end of processing some functionsmaking series-list, nodule-list, emit event of "isFinishedRendering$"
         *    for each split window.
         * */
        const elementId$ = this.currentCtViewerElementId$.pipe(     // 1
            filter(val => val.selectedElementId !== undefined),     // 2
            switchMap(val => {
                this.selectedElementId = val.selectedElementId;     // 3
                return this.ctService.isFinishedRendering$[this.selectedElementId]
                    .pipe(take(1)); // 4
            }),
            takeUntil(this.unsubscribe$)
        );

        const ctViewer$ = this.currentCtViewerElementId$.pipe(      // 4
            filter(val => val.selectedElementId !== undefined),     
            switchMap(val => {
                this.selectedElementId = val.selectedElementId;
                return this.ctService.isStartedRendering$[this.selectedElementId]
                    .pipe(take(1));
            }),
            takeUntil(this.unsubscribe$)
        );
        // 
        if ( gridNo > 1 ) {
            if (this.selectedElementId === '#dicomImage') {         // 5
                this.tempObservable = defer(() => of(EMPTY).pipe());
            } else if (this.selectedElementId === '#dicomImage01') {
                this.tempObservable = zip(ctViewer$, elementId$).pipe(  
                    filter(val => val[1] === '#dicomImage'),        // 6
                );
            } else if (this.selectedElementId === '#dicomImage02') {
                this.tempObservable = zip(ctViewer$, elementId$).pipe(
                    filter(val => val[1] === '#dicomImage01'),
                );
            } else if (this.selectedElementId === '#dicomImage03') {
                this.tempObservable = zip(ctViewer$, elementId$).pipe(
                    filter(val => val[1] === '#dicomImage02'),
                );
            }
        } else { // only one split window
            this.tempObservable = defer(() => of(EMPTY).pipe());    // 7
        }
    }

```

```ts
    private initializeNgInit() {
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
            /** Start processing ct-viewer after finished processing for previous split window*/
            this.renderingCtViewerSplitWindow(val[1]);              // 10
        });
  }
```
1. 
