* D3 _loves **lists**_! When you do a `.data(blah).enter()`, `blah` needs to be list. You spent quite a bit of time wondering why things didn't work because your `blah`s were maps/objects.
    - Speaking of lists, use [D3's array functions](https://github.com/d3/d3-array)
* [This is the margin convention](https://bl.ocks.org/mbostock/3019563). Follow it. Saves a ton of headaches.
* [This is a great resource for scaling functions](https://www.d3indepth.com/scales/).
* "Save" the element you're adding to add to it later.
* Be minful of SO answers; stuff that's OK with v3 might not be for v4.

```javascript
const background = svg.add.....
background.add(object1)
background.add(object2)
background.add(object3)
```
