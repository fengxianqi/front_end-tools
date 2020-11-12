## uuid.js
```javascript
/**
 * Generate random number between 0 - 16.
 *
 * @returns {number}
 */
function random () {
  if (window.crypto && window.crypto.getRandomValues) {
    return window.crypto.getRandomValues(new window.Uint8Array(1))[0] % 16
  } else {
    return Math.random() * 16
  }
}

// black magic here, **DO NOT** try to understand it
// see also:
//  * https://gist.github.com/jed/982883
//  * https://stackoverflow.com/questions/105034/create-guid-uuid-in-javascript
export default function b (a) {
  return a // if the placeholder was passed, return
    ? ( // a random number from 0 to 15
      a ^ // unless b is 8,
      random() >>
      a / 4 // 8 to 11
    ).toString(16) // in hexadecimal
    : ( // or otherwise a concatenated string:
      [1e7] + // 10000000 +
      -1e3 + // -1000 +
      -4e3 + // -4000 +
      -8e3 + // -80000000 +
      -1e11 // -100000000000,
    ).replace( // replacing
      /[018]/g, // zeroes, ones, and eights with
      b // random hex digits
    )
}
```
