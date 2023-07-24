# Here is my revised proposal of raw string literal.

# Learn from (or combine with) the idea of C# 11's raw string literals, which solves almost all problems in one go:
- It solves the indentation problem in that thread
- Can contain any arbitrary text without escape sequences in the content
- Can support interpolation and defining custom interpolation delimiters so that the "normal delimiters" can be in the content without being escaped as well

If we introduce new syntax, we want to put as many features as possible to make the new syntax worthwhile.

# If we want to apply the similar concept to JavaScript, I would design the indentation (and raw string literal) feature this way:

## 1\. The string sequence should start with an @ and at least (can be more than) 2 backtick characters, and close with an equal number of backtick characters without @.

````````````````js
const str = @``
I am a string
``
````````````````

## 2\. These patterns don't need to be escaped

Obviously, characters such as @, double quote, single quote, and backslash won't need to be escaped

````````````````js
const mysqlQuery = @``
    SELECT * FROM `Strange table name` where `strange column name` = 'abcde';
    ``
````````````````

````````````````js
const str = @``
I would like to request "2 leave days" that start from '11/26/2022' and end on '11/27/2022'.
\_/
If any enquiry, please send email to ken@example.com
``
````````````````
The @, ", ', and \ characters will be stored in str as-is, as visibly shown in source code, no need \", \', \\.

1 backtick character doesn't need to be escaped either

````````````````js
const str = @``
I would like to request `2 leave days` that ......
``
````````````````
And the
```
@`
```
sequence doesn't need to be escaped either, since it is not equal to the closing delimiter.

## 3\. Here comes the problem: what if we want to embed 2 or more backtick characters without escaping? In such case, the raw string literal needs to start and end with more backtick characters:

Embedding 2 backtick characters would require opening with an @ and 3 backticks and closing with 3 backticks:
````````````````js
const javaScriptTutorial = @```
   const emptyString = ``  // Yay, backtick quotes can be an empty string too
   ```
````````````````
(the 2 backtick ``` characters will be represented as-is, without the need of escaping)

Embedding 3 backtick characters (a common example is embedding markdown in JavaScript) would require opening with an @ and 4 backticks and closing with 4 backticks:
````````````````js
const markdownExample = @````
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
In this design, we can put arbitrary number of backticks in the content as long as we delimit the raw string by opening and closing with more backtick characters.
Such syntax should not be clumsy, as such kind of strings are rare (we want to cover all corner cases though).

## 4\. Indentation - Any whitespace to the left of the closing delimiter (```, or ````, or `````, ...) will be removed from the string literal

````````````````js
    const markdownExample = @````
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

It is equivalent to
````````````````js
    const markdownExample = @````
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

````
````````````````

Any characters on the left of the closing delimiter will trigger compilation error:
````````````````js
    const markdownExample = @````
        ```json
        {
          "firstName": "John",
          "lastName": "Smith",
          "age": 25
        }
        ```
       a   // The character a is illegal here and should give compilation error rather than ignoring it.
    b   ````  // The character b is also illegal here, the closing delimiter must be on its own line. Should give compilation error rather than ignoring it.
````````````````

## 5\. Indentation - indentation whitespace must be consistent

If the closing delimiter has 8 spaces to the left, then all lines in the content must start with 8 spaces, not tab characters. They can have tab or space characters after the initial 8 spaces though.

If the closing delimiter has 2 tabs to the left, then all lines in the content must start with 2 tabs, not space characters. They can have space or tab characters after the initial 2 tabs though.

## 6\. Interpolation basics - You know it

````````````````js
@``My name is ${myName}``
````````````````

## 7\. What if I want to include ${myName} as part of the content, not interpolation?

The answer borrowed from C# 11 raw string literal is, to include more @ characters at the beginning delimiter.

The number of @ characters at the opening delimiter, will control how many $ characters are needed to start an interpolation:
````````````````js
@@``My name is ${myName}``    // No interpolation, ${myName} is now stored in the string as-is
@@``My name is $${myName}``    // Interpolation will happen
````````````````

````````````````js
const myName = "Ann"
console.log(@@``JavaScript tutorial: If you write console.log(`my name is ${myName}`), it will print "my name is $${myName}" in the results (not including "")``)
````````````````

will print
```
JavaScript tutorial: If you write console.log(`my name is ${myName}`), it will print "my name is Ann" in the results (not including "")
```

More examples
````````````````js
@@@```My name is $${myName}```    // No interpolation, ${myName} is now stored in the string as-is
@@@```My name is $$${myName}```    // Interpolation will happen
````````````````

## 8\. Single-line raw string literal

````````````````js
    const str1 = @``abc``;  // Same as "abc"
    const str2 = @``a`c``; // Same as "a`c"
    const str3 = @```a``c```; // Same as "a``c"
    const str4 = @`````; // Not allowed
    const str5 = @```a``c"d\n```; // Same as "a``c\"d\\n", printed as a``c"d\n
    const str6 = @``I am ${name}``; // Interpolation allowed
    const str7 = @@``I am ${name}``; // No interpolation here
    const str8 = @@``I am $${name}``; // With interpolation here
````````````````

## 9\. More rules on multi-line raw string literal

- Both opening and closing quote characters must be on different lines.
- Whitespace following the opening quote on the same line is ignored.
- Any non-whitespace characters (except comments) following the opening quote on the same line are illegal, and will be treated as unterminated single-line raw string literal.
- Whitespace only lines below the opening quote are included in the string literal.


# PS:

1. I am not sure if Discourse is the right place to draft a new proposal.
2. For other corner case, just learn from C# 11's raw string literal, which in turn learnt from years of other language's previous experiences.
