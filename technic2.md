## Technic2 page


```ts
downloadJsonData() {
    let savedJson = {};
    return from(this.selectedStudies).pipe(
        takeUntil(this.unsubscribe$),
        mergeMap((studyArray: Study) => {
            const st = studyArray;
            savedJson[st.suid] = {};
            return http.get.getSeriesOfStudy(st.suid);
        }),
        toArray(), // One dimensional array [a,b,c]
    ).subscribe((seriesArray) => {
        from(seriesArray).pipe(
            takeUntil(this.unsubscribe$),
            mergeMap((series: Series[]) => {
                const se = series;  // each series has series id (seid) and study id (suid)
                savedJson[se.suid][se.seid] = {};
                return http.get.getNodules(se.seid); // each series has multi nodule
            }),
            toArray() // Two dimensional array [[a,b,c],[aa,bb,cc],[aaa,bbb,ccc]]
        ).subscribe((nodulesList: any[]) => {
            from(nodulesList).pipe(
                takeUntil(this.unsubscribe$),
                map(nodules => {
                    return from(nodules).pipe(
                        takeUntil(this.unsubscribe$),
                        map(noduleData => {
                            const st = noduleData['suid'];
                            const se = noduleData['seid'];

                            if (!savedJson[st][se][noduleData['sopid']]) {
                                savedJson[st][se][noduleData['sopid']] = [];
                            }
                            // 
                            ...
                            savedJson[st][se][noduleData['sopid']].push(some_data);
                            ...
                            return savedJson;
                        }),
                        toArray(), // [[savedJson[st][se][noduleData['sopid']][some_data]],[savedJson[st][se][noduleData['sopid']][some_data]]]
                    ).subscribe(newJson => {
                        if (JSON.stringify(this.oldObj) !== JSON.stringify(fval)) { // remove duplicated data
                            this.oldObj = newJson;
                            const keys = Object.keys(newJson[0]);
                            from(keys).pipe(
                                takeUntil(this.unsubscribe$),
                                concatMap(key => of(key).pipe(takeUntil(this.unsubscribe$),delay(100))),
                                tap(key => {
                                    const data = JSON.stringify(newJson[0][key]);
                                    const blob = new Blob([data], {type: 'application/json'});
                                    // save files at the download directory
                                    saveAs(blob, key);
                                })
                            ).subscribe();
                        }
                    });
                }),
            ).subscribe();
        });
    });
}
```
