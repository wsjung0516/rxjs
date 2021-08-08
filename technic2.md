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
        // One dimensional array [a,b,c]
    ).subscribe((seriesArray) => {
        from(seriesArray).pipe(
            takeUntil(this.unsubscribe$),
            mergeMap((series) => {
                const se = series;  // each series has series id (seid) and study id (suid)
                savedJson[se.suid][se.seid] = {};
                return http.get.getNodules(se.seid); // each series has multi nodule
            }),
            // Two dimensional array [[a,b,c],[aa,bb,cc],[aaa,bbb,ccc]]
        ).subscribe((nodulesList: any[]) => {
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
                //[[savedJson[st][se][noduleData['sopid']][some_data]],[savedJson[st][se][noduleData['sopid']][some_data]]]
            ).subscribe(newJson => {
                if (JSON.stringify(this.oldObj) !== JSON.stringify(fval)) { // remove duplicated data
                    this.oldObj = newJson;
                    const keys = Object.keys(newJson);
                    from(keys).pipe(
                        takeUntil(this.unsubscribe$),
                        concatMap(key => of(key).pipe(takeUntil(this.unsubscribe$),delay(100))),
                        tap(key => {
                            const data = JSON.stringify(newJson[key]);
                            const blob = new Blob([data], {type: 'application/json'});
                            // save files at the download directory
                            saveAs(blob, key);
                        })
                    ).subscribe();
                }
            });
        });
    });
}
```
