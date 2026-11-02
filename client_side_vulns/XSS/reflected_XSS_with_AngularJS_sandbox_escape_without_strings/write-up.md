# XSS 3


## **Reflected XSS with AngularJS sandbox escape without strings - Expert**


### Description:

This lab uses AngularJS in an unusual way where the `$eval` function is not available and you will be unable to use any strings in AngularJS.

To solve the lab, perform a cross-site scripting attack that escapes the sandbox and executes the `alert` function without using the `$eval` function.

---

### PART I: Lab Enumeration

- This application is a blog that provides a `Search` feature.
    
    ![image.png](images/image.png)
    
- After searching for a random string and observing the behavior, I see nothing. Everything looks clean.
    
    ![image.png](images/image%201.png)
    
- Since I already know this application uses AngularJS, I open the page source and scroll down a bit. That's where the main logic is revealed.
    
    ![image.png](images/image%202.png)
    

---

### PART II: Logic Explanation & Injection Point Identification

- Let’s break it down:
    
    ```jsx
    // Create a query property and assign an empty object to it
    $scope.query = {};
    
    // Create a key variable and assign the 'search' parameter to it
    var key = 'search';
    
    // Create $scope.query = {search: 'asdasdasd'}
    // Equivalent to writing $scope.query.search = 'asdasdasd'
    $scope.query[key] = 'asdasdasd'; // 'asdasdasd' is our input
    ```
    
- To understand this line:
    
    ```jsx
    $scope.value = $parse(key)($scope.query);
    ```
    
- Let's talk about `$parse` first. It is an AngularJS service that parses an expression string and returns a function that can evaluate it. The expression passed to `$parse()` can be evaluated directly without using interpolation syntax such as `{{ }}`.
- Usage: `$parse(string)` returns a function. Calling that function with an object (the context) evaluates the original expression against that object.
- Example, the status before that line:
    
    ```jsx
    key = 'search'                              // A normal string
    $scope.query = { search: 'asdasdasd' }      // A normal object
    ```
    
- Step by step:
    - First step: `$parse(key)`
        
        ```jsx
        // Compile to a function
        $parse('search') // -> A function, let's call it compiledFn for now
        ```
        
    - Think of `compiledFn` as being equivalent to what would be written by hand:
        
        ```jsx
        function compiledFn(obj) {
          return obj.search;
        }
        ```
        
    - Second step: call `compiledFn($scope.query)`
        
        ```jsx
        compiledFn({ search: 'asdasdasd' })
        ```
        
    - Following the same logic as `compiledFn`, it accesses the `.search` property and returns `'asdasdasd'`.
- So this line becomes:
    
    ```jsx
    $scope.value = $parse(key)($scope.query);
    // $scope.value = 'asdasdasd'
    ```
    
- With a fixed key of `'search'`, this entire construct serves only to retrieve `$scope.query.search`, which might seem redundant compared to accessing `$scope.query.search` directly. However, this redundancy is a clue: there would be little reason to use a separate variable here unless the key was intended to be dynamic.
- In short, this is where the value of `key` will be evaluated as an AngularJS expression, and it is completely under my control (a.k.a. the injection point).
    
    ```
    ...?search...
    ```
    
- However, the server only renders this AngularJS block when the request contains a parameter named `search`, so renaming it removes the condition that makes the page include this script.
- Instead, I add another parameter. The server includes it as another key-value assignment in the generated script, so the second assignment changes the value of `key` to `test`. As a result, `test` is displayed, and I can now control the expression passed to `$parse()` through this parameter name.
    
    ![image.png](images/image%203.png)
    
- Then I test it using `1%2B1` (`1+1`) and the result is `2` .
    
    ![image.png](images/image%204.png)
    
- The `$scope.query` object only affects the result when the expression references one of its properties. For instance, if the expression is `search` (a bare identifier), Angular looks up the `search` property in the provided context object, `$scope.query`.
- However, an expression like `1+1` is self-contained. It makes no reference to any property names, so the `$scope.query` context is completely ignored and has no impact on the result. Regardless of what object is passed to `$parse('1+1')(...)`, the result will always be `2`.
- This lab prevents me from using string literals by HTML-encoding single and double quotes. Therefore, the payload that worked in the AngularJS Practitioner lab won't work here.
    
    ![image.png](images/image%205.png)
    
- And that leads to the payload in the solution.

---

### PART III: Payload Explanation

#### III.0: The payload

- This lab is based on Gareth Heyes' research, detailed in the following articles:
    - [[XSS without HTML: Client-Side Template Injection with AngularJS](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs)]
    - [[DOM based AngularJS sandbox escape](https://portswigger.net/research/dom-based-angularjs-sandbox-escapes)]
- The solution uses this payload (AngularJS 1.4.4 sandbox):
    
    ```
    toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
    ```
    
- This long payload serves only two purposes.

#### III.1: Overwriting `charAt()`

- Starting with this line:
    
    ```jsx
    toString().constructor.prototype.charAt=[].join
    ```
    
    - The purpose of this line is to overwrite the `charAt` method because Angular's sandbox uses it as part of its internal expression validation mechanism. When Angular parses an expression, it invokes `charAt()` on strings to check for dangerous patterns.
    - The details of this internal security check, along with other aspects of the exploit, are explained in this [[breakdown](https://breachforce.net/angularjs-sandbox-escape)].
    - In short, replacing `charAt()` with `join()` changes how Angular's validation code processes strings. Since `join()` does not behave like `charAt()` and interprets its argument differently, the validation logic no longer works as intended, allowing the payload to bypass checks that would otherwise block it.
    - The mechanism is as follows
        - Since string literals are blocked, `toString()` is used to obtain a string value without explicitly writing one. Accessing its `constructor` property then provides a reference to the `String` constructor.
            
            ```jsx
            toString().constructor
            ```
            
        - Next, by accessing the `prototype` property of the `String` constructor, I can access the methods defined on `String.prototype`.
            
            ```jsx
            // Equivalent to String.prototype
            toString().constructor.prototype 
            ```
            
        - Since `charAt()` is defined on `String.prototype`, it can now be overwritten with `Array.prototype.join`.
            
            ```jsx
            toString().constructor.prototype.charAt=[].join
            ```
            

#### III.2: Recreating a string

- The remaining part is:
    
    ```jsx
    ;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
    ```
    
    - Angular allows multiple expressions to be separated by `;`, so it is used to append the second expression.
    - Also, in AngularJS, the pipe (`|`) represents a filter. More details [[here](https://docs.angularjs.org/guide/filter)].
    - In this lab, the filter being used is `orderBy` . In this lab, the filter being used is `orderBy`. It is normally used to sort an array based on a predicate, which can be provided as an AngularJS expression string.
    - The important part is that AngularJS parses the expression string into a getter and evaluates it when processing the sorting predicate.
    - So, I use the same technique to obtain a reference to the `String` constructor:
        
        ```jsx
        toString().constructor
        ```
        
    - I then call its `fromCharCode()` method . More detaild [[here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode)]:
        
        ```jsx
        toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
        ```
        
    - This returns the string `x=alert(1)`. But why construct an assignment expression instead of simply using `alert(1)`? The explanation is [[here](https://breachforce.net/angularjs-sandbox-escape#step-8-why-do-we-need-xalert1-why-not-simply-alert1)] .

---

### PART IV: Exploitation

- Simply pasting the payload immediately solves the lab. But the point is to understand what it actually does.
    
    ![image.png](images/image%206.png)
    

---

### PART V: Root Cause & Remediation

#### **Root Cause:**

- AngularJS's expression sandbox relies on methods such as `String.prototype.charAt()` to inspect expression syntax character by character. However, JavaScript allows built-in prototype methods to be redefined at runtime. Once `charAt()` is overwritten, the sandbox's validation logic can be disrupted because the application and the validation code share the same JavaScript runtime.
- AngularJS validates an expression before evaluating it. Therefore, the exploit separates the prototype modification and the dangerous expression into two stages. The first expression executes and overwrites `charAt()`. The second expression is then processed after the modification, so its validation pass is affected by the altered method.
- The application passes a user-controlled URL parameter name — not just its value — into `$parse()`. The returned function is then called with `$scope.query`, causing the parameter name to be evaluated as an AngularJS expression. This exposes an `eval()`equivalent primitive through a part of the request that developers may not normally treat as an executable input.
- Blocking quote characters does not prevent this attack, since AngularJS expressions can construct strings without string literals using methods such as `toString()` and `String.fromCharCode()`. This makes character-level filtering ineffective against the underlying expression-evaluation capability.

#### **Remediation:**

- Never pass user-controllable input — including parameter names — into `$parse()`, `$eval()`, or any API that evaluates strings as AngularJS expressions. If dynamic keys are required, validate them against a strict allowlist.
- Do not rely on the AngularJS expression sandbox as a security boundary. It was removed in AngularJS 1.6 after multiple sandbox escape vulnerabilities demonstrated that it was not a reliable security mechanism.
- Migrate away from AngularJS 1.x to a modern framework that does not expose runtime JavaScript execution through user-controlled template expressions.
- Apply a restrictive Content Security Policy (CSP) that omits `'unsafe-eval'` as an additional layer of defense. However, CSP should not replace removing user-controlled input from expression-evaluation sinks.
