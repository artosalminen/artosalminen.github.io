

---
title: "How to capitalize each word of a string in TypeScript"
date: 2023-09-12
series: ["Coding"]
weight: 1
tags: ["Typescript", "Coding"]
author: "Arto Salminen"
draft: false
---

In the world of web development and application design, small details can make a big difference. One such detail is the proper capitalization of text, especially when dealing with user-generated content or dynamic data. Ensuring that text appears neat and consistent can significantly enhance the user experience. In this blog post, we'll introduce you to a handy utility function called `capitalizeEachWord` that can help you achieve just that.

## What is `capitalizeEachWord`?

The `capitalizeEachWord` function is a TypeScript utility that capitalizes the first letter of each word in a given string while keeping the rest of the letters in lowercase. This function also allows you to specify a word separator, which can be useful when dealing with text containing multiple words separated by a character or whitespace.

Here's the function code for reference:

```javascript
export const capitalizeEachWord = (input: string, wordSeparator: string) =>
  input
    .split(wordSeparator)
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(wordSeparator);
```

## How Does It Work?

The `capitalizeEachWord` function works by following these steps:

1. **Splitting the Input String**: It takes the input string and splits it into an array of words using the specified word separator.

2. **Mapping**: It then applies a mapping function to each word in the array. In this mapping function, it capitalizes the first letter of each word (using `charAt(0).toUpperCase()`) and converts the rest of the letters to lowercase (using `slice(1).toLowerCase()`).

3. **Joining**: Finally, it joins the modified words back together using the same word separator to form a single string with each word capitalized correctly.

## Example Usage

Let's illustrate the usage of `capitalizeEachWord` with some examples:

```javascript
const inputText1 = "hello world";
const inputText2 = "typeScript_is_awesome";
const separator = "_";

const result1 = capitalizeEachWord(inputText1, " ");
const result2 = capitalizeEachWord(inputText2, separator);

console.log(result1); // Output: "Hello World"
console.log(result2); // Output: "TypeScript_Is_Awesome"
```

Benefits of Using `capitalizeEachWord`

1. **Consistency**: By ensuring that each word starts with a capital letter, you can achieve a consistent and professional look for your text, regardless of the input.

2. **Improved Readability**: Capitalizing each word makes text more readable, especially for titles, headings, or user-generated content.

3. **Customization**: The function allows you to define the word separator, making it adaptable to various text formatting scenarios.

4. **Efficiency**: `capitalizeEachWord` streamlines the process of capitalizing words in a string, saving you time and effort in manually formatting text.

## Conclusion

The `capitalizeEachWord` utility function is a simple yet powerful tool that can enhance the appearance and readability of your text in web development projects. Whether you're working on a blog, a content management system, or an e-commerce platform, this function can help you achieve consistent and professional text formatting effortlessly. So, the next time you need to capitalize words in a string, remember to give `capitalizeEachWord` a try, and watch your text come to life!
