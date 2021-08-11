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
    return from(dataList).pipe(                 // 1.
        takeUntil(this.unsubscribe$),
        mergeMap((data: Data[]) => {            // 2.
            const da = data;
            savedJson[da.id] = {};              // 3
            return http.get.getSeriesOfStudy(da.id);
        }),
        // [{..}, {..}, {..}]                   // 4.
    ).subscribe((itemList) => {
        from(itemList).pipe(
            takeUntil(this.unsubscribe$),
            mergeMap((item) => {
                const it = item;
                savedJson[da.id][it.id] = {};    // 5.
                return http.get.getNodules(it.id); 
            }),
            // [{..}, {..}, {..}, {..}]
        ).subscribe((subItemList: any[]) => {
            return from(subItemList).pipe(
                takeUntil(this.unsubscribe$),
                map(subItem => {
                    const da_id = subItem['da_id']; // 6.
                    const it_id = subItem['it_id'];
                    const su_id = subItem['su_id'];

                    if (!savedJson[da_id][it_id][su_id]) {
                        savedJson[da_id][it_id][su_id] = [];        // 7.
                    }
                    // 
                    ...
                    savedJson[da_id][it_id][su_id].push(some_data); // 8.
                    ...
                    return savedJson;
                }),
                /* // 9.
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
                const keys = Object.keys(newJson); // 10.
                from(keys).pipe(
                    takeUntil(this.unsubscribe$),
                    concatMap(key => of(key).pipe(takeUntil(this.unsubscribe$),delay(100))), // 11.
                    tap(key => {
                        const data = JSON.stringify(newJson[key]);
                        const blob = new Blob([data], {type: 'application/json'});
                        // save files at the download directory
                        saveAs(blob, key); // 12.
                    })
                ).subscribe();
            });
        });
    });
}
```
1. 복수개의 data를 Array로 입력하여 처리한다.
2. 각 data처리는 Async 통신의 출력 순서와 상관 없으므로 한개씩 처리한다.
3. 최종값을 저장할 Object에 첫번째 key 값은 만든다.
4. 처리결과는 Array가된다.
5. 최종값을 저장할 Object에 두번째 key값을 만든다.
6. 각 subItem은 parent의 키 값을 가지고 있다. 
7. 최종값을 저장할 Object에 세번째 key값을 만든다.
8. 최종값을 저장할 Object에 함수 수행의 결과 값을 저장한다. 
9. 함수 수행의 결과 값이 저장된 Object의 구조이다 (console.log 로 표시한 값)
10. 최종결과 값을 파일로 저장해야되는데, 복수개의 data를 처리하는 경우 복수개의 file을 local storage에 
    저장할 때 비동기로 이루어지면, 
11. 화일을 local 에 저장하는 경우 저장이 완료되었는지 결과 값을 받을 수 없으므로, 다음 순서를 임의의 시간 뒤에, 순차적으로 기록하도록 한다.
12. 최종 값을 file로 저장한다.
    
