Not sure if it's too late for me to post this now. I just saw a previous proposal:
https://es.discourse.group/t/triple-backtick-template-literal-with-indentation-support/337

The proposal only targeted to solve the indentation problem of multiline strings, but it will use up the triple backtick (```) syntax, and making my proposal below likely not able to be implemented without breaking existing codes.

Here is my proposal.

# Learn from (or combine with) the idea of C# 11's raw string literals, which solves almost all problems in one go:
- It solves the indentation problem in that thread
- Can contain any arbitrary text without escape sequences in the content
- Can support interpolation and defining custom interpolation delimiters so that the "normal delimiters" can be in the content without being escaped as well

# If we want to apply the similar concept to JavaScript, I would design the indentation (and raw string literal) feature this way:

1\. The string sequence should start with at least (can be more than) 3 backtick characters,

````````````````js
const str = ```
I am a string
```
````````````````

2\. Obviously, characters such as double quote, single quote, and backslash won't need to be escaped

````````````````js
const str = ```
I would like to request "2 leave days" that start from '11/26/2022' and end on '11/27/2022'.
\_/
```
````````````````
The ", ', and \ characters will be stored in str as-is, as visibly shown in source code, no need \", \', \\.

3\. 1 or 2 backtick characters no need to be escaped either

````````````````js
const str = ```
I would like to request `2 leave days`` that ......
```
````````````````

4\. Here comes the problem: what if we want to embed 3 backtick characters? In such case, the raw string literal needs to start and end with 4 backtick characters:

````````````````js
const markdownExample = ````
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```
````
````````````````
(the 3 backtick ``` characters will be represented as-is, without the need of escaping)

If we want to represent 4 backticks in the content without escaping, we need to delimit the raw string by 5 backtick characters, and so on.
In this design, we can put arbitrary number of backticks in the content as long as we delimit the raw string by more backtick characters.
Such syntax should not be clumsy, as such kind of strings are rare (we want to cover all corner cases though).

5\. Indentation - Any whitespace to the left of the closing delimiter (```, or ````, or `````, ...) will be removed from the string literal

````````````````js
    const markdownExample = ````
       |```json
       |{
       |  "firstName": "John",
       |  "lastName": "Smith",
       |  "age": 25
       |}
       |```
       |
        ````
````````````````
Note: The | characters are not really in the string, they're for illustrating how the string is indented and that the whitespaces at | and on the left of | are not captured in the string.

6\. Interpolation basics - You know it

````````````````js
```My name is ${myName}```
````````````````

7\. What if I want to include ${myName} as part of the content, not interpolation?

The answer borrowed from C# 11 raw string literal is, to include more $ characters at the beginning delimiter:

````````````````js
$$```My name is ${myName}```    // No interpolation, ${myName} is now stored in the string as-is
$$```My name is $${myName}```    // Interpolation will happen
````````````````

````````````````js
const myName = "Ann"
console.log($$```JavaScript tutorial: If you write "my name is ${myName}", it will show "my name is $${myName}"```)
````````````````

will print

JavaScript tutorial: If you write "my name is ${myName}", it will show "my name is Ann"

More examples
````````````````js
$$$```My name is $${myName}```    // No interpolation, ${myName} is now stored in the string as-is
$$$```My name is $$${myName}```    // Interpolation will happen
````````````````

# PS:

1. I am not sure if Discourse is the right place to draft a new proposal.
2. For other corner case, just learn from C# 11's raw string literal, which in turn learnt from years of other language's previous experiences.
