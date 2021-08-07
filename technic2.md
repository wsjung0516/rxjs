## Technic2 page


```ts
    onDownloadAnnotationsDataStudies() {
        let fullJSONDataByStudyUIDs = {};
        return from(this.selectedStudies).pipe(
            takeUntil(this.unsubscribe$),
            mergeMap((selectedStudy: Study) => {
                const st = selectedStudy;
                fullJSONDataByStudyUIDs[selectedStudy.study_instance_uid] = {};
                return this.restAPIService.getSeriesOfStudy(st.study_instance_uid);
            }),
            toArray(),
        ).subscribe((seriesList) => {
            from(seriesList).pipe(
                takeUntil(this.unsubscribe$),
                mergeMap((series: Series[]) => {
                    const se = series[0];
                    fullJSONDataByStudyUIDs[se.study_instance_uid][se.series_instance_uid] = {};
                    return this.restAPIService.getNodules(se.series_instance_uid, this.userId);
                }),
                toArray()
            ).subscribe((noduleList: any[]) => {
                from(noduleList).pipe(
                    takeUntil(this.unsubscribe$),
                    map(nodules => {
                        return from(nodules).pipe(
                            takeUntil(this.unsubscribe$),
                            map(noduleData => {
                                const st = noduleData['study_instance_uid'];
                                const se = noduleData['series_instance_uid'];
                                
                                if (!fullJSONDataByStudyUIDs[st][se][noduleData['sop_instance_uid']]) {
                                    fullJSONDataByStudyUIDs[st][se][noduleData['sop_instance_uid']] = [];
                                }
                                // OcViewerProxy.makeCornerstoneToolsDataForNodule(noduleData);
                                OcViewerProxy.generateCstDataForNodule(noduleData);
                                const defaultData = {
                                    probe: {x: noduleData['nodule_coordinate_x'], y: noduleData['nodule_coordinate_y']},
                                    length: noduleData['long_diameter'],
                                    nodule_uuid: noduleData['nodule_uuid'],
                                    series_instance_index: noduleData['series_instance_index']
                                };
                                
                                const jsonData = JSON.parse(noduleData['nodule_segments_info_json']);
                                
                                // the freehandroi -> bidirectional order must be kept
                                if (jsonData.freehandroi_cst_data) {
                                    let annotationData = {...defaultData};
                                    annotationData['type'] = 'polygon';
                                    annotationData['polygon'] = jsonData.freehandroi_cst_data;
                                    annotationData['annotation_uuid'] = jsonData.freehandroi_cst_data.uuid;
                                    fullJSONDataByStudyUIDs[st][se][noduleData['sop_instance_uid']].push(annotationData);
                                }
                                if (jsonData.bidirectional_solid_cst_data) {
                                    let annotationData = {...defaultData};
                                    annotationData['type'] = 'axes';
                                    annotationData['axes'] = jsonData.bidirectional_solid_cst_data;
                                    annotationData['annotation_uuid'] = jsonData.bidirectional_solid_cst_data.uuid;
                                    fullJSONDataByStudyUIDs[st][se][noduleData['sop_instance_uid']].push(annotationData);
                                }
                                return fullJSONDataByStudyUIDs;
                            }),
                            distinctUntilChanged(),
                            toArray(),
                        ).subscribe(fval => {
                            if (JSON.stringify(this.oldObj) !== JSON.stringify(fval)) { // remove duplicated data
                                this.oldObj = fval;
                                const keys = Object.keys(fval[0]);
                                // console.log('keys',keys);
                                from(keys).pipe(
                                    takeUntil(this.unsubscribe$),
                                    concatMap(key => of(key).pipe(takeUntil(this.unsubscribe$),delay(100))),
                                    tap(key => {
                                        const data = JSON.stringify(fval[0][key]);
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
