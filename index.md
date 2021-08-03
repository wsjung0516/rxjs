## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/wsjung0516/rxjs-technics/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Technic 1

#### Requirement:

1. Getting the file infomation from SubItem source.
2. Data has the array of Item, but need only the first SubItem.
3. Item has the array of File;

![sample](/assets/images/technic1.png)

```markdown

```

```ts
Conditions
1. Need to connect to remote server that has the data.
2. Consider async communication to get the data through http connection
3. Three step nested structure, first and second step has each children;

class Data {
    itemList: Item[];
}
class Item {
    sub_item: SubItem[];
}
class SubItem {
    item: File;
}

getMain( data: Data) {
    const dataList = data.itemList;     // 1. 
    return from(dataList).pipe(
        takeUntil(this.unsubscribe$),
        mergeMap( item => {             // 2. 
            const url1 = `http://${defined_host_ip}:3333/sub_items${item}`
            return this.http.get(url1).pipe(
                map( val => val[0]),    // 3. 
                toArray(),              // 4. 
                switchMap( n_sub_items => {             // 5. 
                    return from(n_sub_items).pipe(      // 6. 
                        takeUntil(this.unsubscribe$),
                        mergeMap( item => {             // 7. 
                            const url2 =`http://${defined_hist_ip2}:3334/preview${item}`;  
                            return this.http.get(url2, {responseType: 'blob'}).pipe( // 8. 
                                tap( resp => {
                                    someFunction(resp); // 9. 
                                })
                            )
                        })
                    )
                })
            )
        }),
    ).subscribe()
}
```
1. Data has sub item list
2. each item need to get sub_items from server without considering response sequence.
3. This return value has multiple sub_items and only need the first item
4. Whenver mergeMap(2) working, the result data (3) is accumulated with array format.
5. After step(4), new arrary data is created, and this data baton passed to the next step.  
6. From this step, start again the same step as 1.
7. This step is the same as step
8. Get the item's data.
9. Do the final step

## New technics

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/wsjung0516/rxjs-technics/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
