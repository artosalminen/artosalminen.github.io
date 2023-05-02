---
title: "How to scrape all links from a website"
date: 2023-05-02
series: ["www"]
weight: 1
tags: ["Chrome", "Dev Tools"]
author: "Arto Salminen"
draft: false
---

Simply copy-paste the following one-liner to Chrome Dev Tools. To limit the results, set the reg exp placeholder to 
contain some sensible value, or replace the function call with return value `true`.

```javascript
Array.prototype.slice.call( document.getElementsByTagName('a'), 0 )
  .map(i => i.href)
  .filter(i => i.match('reg exp placeholder'))
  .forEach(i => console.log(i))
```

Here is an explanation for what it does:

1. Selects all the <a> elements on the document.
2. Converts the HTMLCollection object returned by getElementsByTagName into an Array by using the Array.prototype.slice method.
3. The slice() method returns a shallow copy of a portion of an array into a new array object selected from begin to end (end not included) where begin and end represent the index of items in that array. Here, it selects all the items starting from index 0 to the end of the array by passing 0 as the first argument to slice() method.
4. Maps the href attribute of each of the selected <a> elements into an array using the Array.prototype.map method. The resulting array will contain a list of all the href attributes of the <a> elements.
5. Filters the array obtained in the previous step by selecting only those elements that match a regular expression pattern. The pattern is a placeholder and should be replaced with an actual regular expression to match the desired elements. This is achieved using the Array.prototype.filter method.
6. Loops through the resulting filtered array using the forEach() method and logs each element to the console using console.log() method. This will output the list of href attributes of the <a> elements that match the given regular expression pattern to the console.