## Technic2 page
다수의 3차원 object를, 1차원 object(parent)를 기준으로 nested object(children), nested object (grand childrend)안의 각 object의 키 값으로 해서 3차원 배열을 만든 다음에 이 배열에 값을 저장하여, 최종적으로 그것을 물리적 저장 장치에 파일로 저장한다.
각 차원의 object는 1:n의 구조로 다음 그림과 같다.  

1. data#1, data#2 ... data#n으로 다수가 존재하고 이 data의 갯수만큼 파일이 저장된다.
2. 각 data 에는 다수의 item이 존재하고, 각 item은 다수의 sub item이 존재한다.
3. 각 sub item은 각 image 정보를 가지고 있다. 

![technic](/assets/images/technic2.png)

```ts
downloadJsonData() {
    let savedJson = {};
    return from(dataList).pipe(             // 1.
        takeUntil(this.unsubscribe$),
        mergeMap((data: Data[]) => {        // 2.
            const da = data;
            savedJson[da.id] = {};
            return http.get.getSeriesOfStudy(da.id);
        }),
        // [{..}, {..}, {..}]               // 3.
    ).subscribe((itemList) => {
        from(itemList).pipe(
            takeUntil(this.unsubscribe$),
            mergeMap((item) => {
                const it = item;
                savedJson[da.id][it.id] = {};
                return http.get.getNodules(it.id); // each series has multi nodule
            }),
            // [{..}, {..}, {..}, {..}]
        ).subscribe((subItemList: any[]) => {
            return from(subItemList).pipe(
                takeUntil(this.unsubscribe$),
                map(subItem => {
                    const da_id = subItem['da_id']; // 4.
                    const it_id = subItem['it_id'];
                    const su_id = subItem['su_id'];

                    if (!savedJson[da_id][it_id][su_id]) {
                        savedJson[da_id][it_id][su_id] = [];    // 5.
                    }
                    // 
                    ...
                    savedJson[da_id][it_id][su_id].push(some_data); // 6.
                    ...
                    return savedJson;
                }),
                /* // 7.
               {da_id:
                   {it_id: 
                       {su_id: Array(2)
                         0: {...}
                         1: {...}
                        }
                    }
                   {it_id: 
                       {su_id: Array(3)
                         0: {...}
                         1: {...}
                         2: {...}
                        }
                    }
                }
               {da_id:
                   {it_id: 
                       {su_id: Array(1)
                         0: {...}
                        }
                    }
                   {it_id: 
                       {su_id: Array(4)
                         0: {...}
                         1: {...}
                         2: {...}
                         3: {...}
                        }
                    }
                }
            */
            ).subscribe(newJson => {
                const keys = Object.keys(newJson); // 8.
                from(keys).pipe(
                    takeUntil(this.unsubscribe$),
                    concatMap(key => of(key).pipe(takeUntil(this.unsubscribe$),delay(100))), // 9.
                    tap(key => {
                        const data = JSON.stringify(newJson[key]);
                        const blob = new Blob([data], {type: 'application/json'});
                        // save files at the download directory
                        saveAs(blob, key); // 10.
                    })
                ).subscribe();
            });
        });
    });
}
```
