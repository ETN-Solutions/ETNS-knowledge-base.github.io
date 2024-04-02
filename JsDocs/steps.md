---
layout: home
parent: JsDocs
title: Steps to create JsDocs
---

# Steps


How to add documentations comment to code ?

- JSDoc comments should generally be placed immediately before the code being documented. Each comment must start with a   `/** `  sequence in order to be recognized by the JSDoc parser. Comments beginning with `/*`, `/***`, or more than 3 stars will be ignored. This is a feature to allow you to suppress parsing of comment blocks.

- Example ---- <br>
 ```json
 /**
 * Calculates the sum of two numbers.
 * @param {number} num1 - The first number.
 * @param {number} num2 - The second number.
 * @returns {number} The sum of the two numbers.
 */
function add(num1, num2) {
    return num1 + num2;
}
```



Step to run: To run jsdoc open the terminal and write the following command <br>
`npm run jsdoc`

saahilbiswal2000@gmail.com