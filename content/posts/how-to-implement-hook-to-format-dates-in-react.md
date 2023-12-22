---
title: "How to create a custom React hook for locale-specific date formatting"
date: 2023-12-22
series: ["Coding"]
weight: 1
tags: ["Typescript", "Coding", "React"]
author: "Arto Salminen"
draft: false
---

In this post, I will show you how to create a custom React hook that provides locale-specific date formatting functions. This hook can be useful if you want to display dates and times in different languages and formats depending on the user's preferences.

## What is a custom React hook?

A custom React hook is a function that starts with the word `use` and may call other hooks. Custom hooks let you reuse stateful logic between components without duplicating code or introducing complex patterns. You can learn more about custom hooks from [the official React documentation (1)](#here-is-some-further-reading-for-you) or [this tutorial (2)](#here-is-some-further-reading-for-you).

## What is Intl.DateTimeFormat?

Intl.DateTimeFormat is a JavaScript object that enables language-sensitive date and time formatting. It takes a locale (a string that identifies a language and a region) and an options object (that specifies how to format the date and time) as arguments and returns a formatter object that has a `format` method. You can use this method to convert a Date object into a string that follows the locale and options you provided. You can also use other methods such as `formatToParts` and `formatRange` to get more control over the output. You can learn more about Intl.DateTimeFormat from [the MDN web docs (3)](#here-is-some-further-reading-for-you) or [this Stack Overflow answer (4)](#here-is-some-further-reading-for-you).

## How to create the custom hook?

The custom hook we want to create is called `useLocaleDateFormat`. It takes no arguments and returns an object containing the default locale and three formatting functions: `formatDate`, `formatTime`, and `formatDateTime`. These functions take a Date object as an argument and return a string that represents the date, the time, or the date and time in the default locale.

Here is the code of the custom hook:


```typescript
import React from 'react';

interface UseLocaleDateFormat {
  defaultLocale: string;
  formatDate: (value: Date) => string;
  formatTime: (value: Date) => string;
  formatDateTime: (value: Date) => string;
}

/**
 * Custom hook that provides locale-specific date formatting functions.
 * @returns An object containing the default locale and formatting functions for date, time, and date-time.
 */
export function useLocaleDateFormat(): UseLocaleDateFormat {
  let defaultLocale = (typeof navigator !== 'undefined' && navigator.language) || 'en-US';
  try {
    // @ts-ignore
    Intl.DateTimeFormat.supportedLocalesOf([defaultLocale]);
  } catch (_err) {
    defaultLocale = 'en-US';
  }

  const value = React.useMemo(() => {
    return {
      defaultLocale,
      formatDate: (value: Date) => value.toLocaleDateString(defaultLocale),
      formatTime: (value: Date) =>
        value.toLocaleTimeString(defaultLocale, { hour: '2-digit', minute: '2-digit' }),
      formatDateTime: (value: Date) =>
        `${value.toLocaleDateString(defaultLocale)} ${value.toLocaleTimeString(defaultLocale, {
          hour: '2-digit',
          minute: '2-digit',
        })}`,
    };
  }, [defaultLocale]);

  return value;
}

```

Let's break down the code and explain each part.

### Define the interface

The first thing we do is to define an interface that describes the shape of the object that the hook returns. This is optional, but it helps us to write type-safe code and get better editor support. The interface has four properties: `defaultLocale`, `formatDate`, `formatTime`, and `formatDateTime`. The first one is a string that represents the default locale, and the rest are functions that take a Date object and return a string.

### Get the default locale

The next thing we do is to get the default locale from the `navigator.language` property, which returns the language of the browser. This property may not be available in some environments, such as server-side rendering, so we use a fallback value of `'en-US'` in case it is undefined. We also use a try-catch block to check if the locale is supported by Intl.DateTimeFormat using the `supportedLocalesOf` method. If the locale is not supported, we also use the fallback value of `'en-US'`.

### Use React.useMemo

The next thing we do is to use the `React.useMemo` hook to memoize the value of the object that contains the default locale and the formatting functions. This hook takes a function that returns the value and an array of dependencies as arguments and returns the memoized value. The memoized value is only recomputed when one of the dependencies changes, which in this case is the default locale. This way, we avoid creating a new object and new functions on every render, which can improve performance and avoid unnecessary re-renders of the components that use the hook.

### Create the formatting functions

The last thing we do is to create the formatting functions using the `toLocaleDateString`, `toLocaleTimeString`, and template literals. These functions use the default locale and some options to format the date and time according to the user's preferences. For example, the `formatDate` function uses the `toLocaleDateString` method, which returns a string with a language-sensitive representation of the date portion of the Date object. The `formatTime` function uses the `toLocaleTimeString` method, which returns a string with a language-sensitive representation of the time portion of the Date object. The `formatDateTime` function uses both methods and a template literal to combine the date and time strings into one.

## How to use the custom hook?

To use the custom hook, we simply import it and call it inside a functional component. We can then use the returned object to access the default locale and the formatting functions. For example, we can create a component that displays the current date and time in the default locale:

```js
import React from 'react';
import { useLocaleDateFormat } from './useLocaleDateFormat';

export function CurrentDateTime() {
  const { defaultLocale, formatDateTime } = useLocaleDateFormat();
  const [now, setNow] = React.useState(new Date());

  React.useEffect(() => {
    const timer = setInterval(() => {
      setNow(new Date());
    }, 1000);
    return () => {
      clearInterval(timer);
    };
  }, []);

  return (
    <div>
      <p>The current date and time in {defaultLocale} is:</p>
      <p>{formatDateTime(now)}</p>
    </div>
  );
}
```

This component uses the `useLocaleDateFormat` hook to get the default locale and the `formatDateTime` function. It also uses the `useState` and `useEffect` hooks to create a state variable for the current date and time and update it every second using a timer. It then renders a paragraph that shows the current date and time in the default locale.

## Tests

And here are a couple of tests for the `useLocaleDateFormat` hook.

```typescript
import { renderHook } from '@testing-library/react';
import { useLocaleDateFormat } from './useLocaleDateFormat';

describe('useLocaleDateFormat', () => {
  const locale = vitest.spyOn(navigator, 'language', 'get');

  it('should format date, time, and datetime correctly in en-US locale', () => {
    locale.mockReturnValue('en-US');

    const { result } = renderHook(() => useLocaleDateFormat());

    expect(result.current.defaultLocale).toEqual('en-US');

    const date = new Date(2022, 0, 15, 12, 30, 0);

    const formattedDate = result.current.formatDate(date);
    expect(formattedDate).toEqual('1/15/2022');

    const formattedTime = result.current.formatTime(date);
    expect(formattedTime).toEqual('12:30 PM');

    const formattedDateTime = result.current.formatDateTime(date);
    expect(formattedDateTime).toEqual('1/15/2022 12:30 PM');
  });

  it('should format date, time, and datetime correctly in fi-FI locale', () => {
    locale.mockReturnValue('fi-FI');

    const { result } = renderHook(() => useLocaleDateFormat());

    expect(result.current.defaultLocale).toEqual('fi-FI');

    const date = new Date(2022, 0, 15, 12, 30, 0);

    const formattedDate = result.current.formatDate(date);
    expect(formattedDate).toEqual('15.1.2022');

    const formattedTime = result.current.formatTime(date);
    expect(formattedTime).toEqual('12.30');

    const formattedDateTime = result.current.formatDateTime(date);
    expect(formattedDateTime).toEqual('15.1.2022 12.30');
  });
});


```
## Conclusion

I hope you found this post helpful and learned something new. If you have any questions or feedback, please let me know in the comments. Thank you for reading! 😊.

### Here is some further reading for you:

1. <a name="1"></a> Reusing Logic with Custom Hooks – React - GitHub Pages. https://react.dev/learn/reusing-logic-with-custom-hooks.
2. <a name="2"></a>How to create your own custom React Hooks - LogRocket Blog. https://blog.logrocket.com/create-your-own-custom-react-hooks/.
3. <a name="3"></a>Intl.DateTimeFormat - JavaScript | MDN - MDN Web Docs. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat.
4. <a name="4"></a>How to format a JavaScript date object using Intl.DateTimeFormat. https://stackoverflow.com/questions/60672126/how-to-format-a-javascript-date-object-using-intl-datetimeformat.
5. <a name="5"></a>Intl.DateTimeFormat() constructor - JavaScript | MDN. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat.
6. Custom Hooks | Hands on React. https://handsonreact.com/docs/custom-hooks.
7. useHooks – The React Hooks Library. https://usehooks.com/.
8. React Custom Hook tutorial with example - BezKoder. https://www.bezkoder.com/react-custom-hook/.
9. en.wikipedia.org. https://en.wikipedia.org/wiki/React_(software).
