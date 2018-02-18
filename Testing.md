HELLO from CALIFORNIA!


```javascript
const wikiState = {
  listOfArticles: [],

  getList: () => {
    Article.getList().then((response) => {
      Object.assign(wikiState.listOfArticles, response)
    })
  }
}
```
