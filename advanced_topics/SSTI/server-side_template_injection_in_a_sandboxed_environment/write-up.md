# SSTI 1

---

### **Server-side template injection in a sandboxed environment - Expert**

---

#### Description:

#### This lab uses the Freemarker template engine. It is vulnerable to server-side template injection due to its poorly implemented sandbox. To solve the lab, break out of the sandbox to read the file `my_password.txt` from Carlos's home directory. Then submit the contents of the file.

#### You can log in to your own account using the following credentials: `content-manager:C0nt3ntM4n4g3r`

---

#### PART I: Lab Enumeration & Template Engine Identification

- Accessing the lab, the home page displays several products and `My account` button.
    
    ![image.png](images/image.png)
    
- Then I log in with the given credentials.
    
    ![image.png](images/image%201.png)
    
- By checking details of a product, I have an `Edit template` button which is only visible because my role is `Content Manager` I guess.
    
    ![image.png](images/image%202.png)
    
- Clicking the button allows me to edit the template, so I remove most of the static text to make it easier to work with. I only keep this line because it uses template syntax, and it seems that I can control the `product` object.
    
    ![image.png](images/image%203.png)
    
- Based on the lab description, the template engine being used is `FreeMarker` . However, I want to approach this from a black-box perspective, so I try to trigger an error by calling a non-existent method, which might reveal the template engine, like this.
    
    ![image.png](images/image%204.png)
    

---

#### PART II: Looking For Documented Payloads

- Each template engine has different exploitation techniques, and remembering all of them is impossible. There’s a page called `HackTricks` that contains a massive number of payload, not just for SSTI.
- I go there, look for `FreeMarker`, and find several payloads.
    
    ![image.png](images/image%205.png)
    
- They even have a sandbox bypass payload, although it only works on versions below 2.3.30.
    
    ![image.png](images/image%206.png)
    
- I do some research on `FreeMarker SSTI` and learn that `FreeMarker` does not have a built-in function to execute commands directly, but it does provide a mechanism for calling methods on Java objects when the corresponding objects or classes are accessible.
- There are two commonly used classes for obtaining RCE:
    - `freemarker.template.utility.Execute` : built-in class, designed to run shell command (primarily used in dev/debug, but still present in the lib).
    - These are three separate payloads that achieve the same goal: creating a `freemarker.template.utility.Execute` object.
        
        ```
        <#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id")}
        [#assign ex = 'freemarker.template.utility.Execute'?new()]${ ex('id')}
        ${"freemarker.template.utility.Execute"?new()("id")}
        ```
        
    - According to the [[official documentation](https://freemarker.apache.org/docs/api/freemarker/template/utility/Execute.html)]. This class can execute external commands.
        
        ![image.png](images/image%207.png)
        
    - For more details, we can look at the [[source code](https://apache.googlesource.com/freemarker/%2B/9a19894d448e7bbec8a3cd00efc7c4ea0909e04b/src/main/java/freemarker/template/utility/Execute.java)] of the `Execute` class . It has an `exec()` method that actually calls `Runtime.getRuntime().exec()` to execute the given command.
        
        ```java
        //...
        package freemarker.template.utility;
        import java.io.IOException;
        import java.io.InputStream;
        import java.io.InputStreamReader;
        import java.io.Reader;
        import java.util.List;
        import freemarker.template.TemplateModelException;
        //...
        public class Execute implements freemarker.template.TemplateMethodModel {
            ...
            @Override
            public Object exec (List arguments) throws TemplateModelException {
                String aExecute;
                ...
                try {
                    Process exec = Runtime.getRuntime().exec( aExecute );
                    ...
                    }
                ...
            }
        }
        ```
        
    - Because of that, this lab prevents me from creating an `Execute` object using `?new()` .
        
        ![image.png](images/image%208.png)
        
    - `freemarker.template.utility.ObjectConstructor` allows the instantiation of arbitrary Java classes, including `ProcessBuilder`. (You can research this class for more information.)
    - This class is blocked as well.
        
        ![image.png](images/image%209.png)
        
- The reason these two classes work is that  [[FreeMarker's built-in](https://freemarker.apache.org/docs/ref_builtins_expert.html#ref_builtin_new)] `?new()` allows me to instantiate classes that implement `TemplateModel`. Since both classes are already included in the `FreeMarker JAR`, no additional imports are required.
    
    ![image.png](images/image%2010.png)
    
- However, access to classes and constructors can be restricted by `FreeMarker`'s security mechanisms such as `TemplateClassResolver` or `MemberAccessPolicy` (version ≥ 2.3.30).
- [[Reference](https://freemarker.apache.org/docs/app_faq.html#faq_template_uploading_security)]
    
    ![image.png](images/image%2011.png)
    
- Therefore, we can no longer obtain RCE simply by creating an `Execute` object using the              `?new()` built-in. So, let’s try using the sandbox bypass payload from `Hacktricks` .
- Using the exact same payload returns the following error: `The following has evaluated to null or missing: ===> article` . This happens because the object I have access to in this lab is `product`, while `article` does not seem to exist.
    
    ![image.png](images/image%2012.png)
    
- I need to replace `article` with `product` , and the application return the result of the `id` command, indicating that I can now achieve RCE.
    
    ![image.png](images/image%2013.png)
    
- The rest of the process is simply finding the path to the password file and reading it.
    
    ![image.png](images/image%2014.png)
    
    ![image.png](images/image%2015.png)
    
    ![image.png](images/image%2016.png)
    
- And submit the password to solve the lab.
    
    ![image.png](images/image%2017.png)
    

---

#### PART III:  Explanation

- Before I start explaining, I will assume that you are already familiar with the basics of Java reflection. Javadoc screenshots are provided for reference rather than as a complete introduction to Java.
- Here’s the payload that has successfully run command:
    
    ```
    <#assign classloader=product.class.protectionDomain.classLoader>
    <#assign owc=classloader.loadClass("freemarker.template.ObjectWrapper")>
    <#assign dwf=owc.getField("DEFAULT_WRAPPER").get(null)>
    <#assign ec=classloader.loadClass("freemarker.template.utility.Execute")>
    ${dwf.newInstance(ec,null)("id")}
    ```
    
- And you can see that we still need a `freemarker.template.utility.Execute` object to achieve RCE. But we can’t create it directly using `?new()` . Instead, we need to take a longer path through other objects and find a way to create an `Execute` object without triggering the `TemplateClassResolver` restriction.
- Tp do that:
    - First, we need a `java.lang.ClassLoader`  object, which I will refer to simply as a `ClassLoader`. This object can load classes by name, including the `Execute` class we need.
        
        ![image.png](images/image%2018.png)
        
    - This line shows the traversal through several objects to obtain a `ClassLoader` object and assign it to the `classloader` variable.
        
        ```
        <#assign classloader=product.class.protectionDomain.classLoader>
        ```
        
    - The result shows that the `classloader` variable now contains a `ClassLoader` object.
        
        ![image.png](images/image%2019.png)
        
    - The `Class` object also has a `getClassLoader()` method, but calling it directly returns `null` in this lab, so we cannot use it. Therefore, we must go through the `ProtectionDomain` object instead.
        
        ![image.png](images/image%2020.png)
        
    - The full chain is shown below:
        
        ![image.png](images/image%2021.png)
        
    - Now that we have a `ClassLoader` object, we can use its `loadClass(String name)` method to load the `Execute` class. Then assign the result to the `ec` variable.
        
        ```
        <#assign ec=classloader.loadClass("freemarker.template.utility.Execute")>
        ```
        
    - At this point, `ec` is only a `Class` object representing the `Execute` class, not an actual `Execute` object.
        
        ![image.png](images/image%2022.png)
        
    - Because of that, we need to find a way to create it. This is why we use `ClassLoader` again to load the `ObjectWrapper` class. And assign it to the `owc` variable.
        
        ```
        <#assign owc=classloader.loadClass("freemarker.template.ObjectWrapper")>
        ```
        
    - Just like `ec`, `owc` is also a `Class` object representing `ObjectWrapper` class.
        
        ![image.png](images/image%2023.png)
        
    - Next, we use another `Class` method, `getField(String name)`, to retrieve the `DEFAULT_WRAPPER` field from the `ObjectWrapper` interface and assign its value to the `dwf` variable. But why do we need it ?
        
        ```
        <#assign dwf=owc.getField("DEFAULT_WRAPPER").get(null)>
        ```
        
    - `ObjectWrapper` is a `FreeMarker` interface used to wrap Java objects as `TemplateModel` objects so they can be used in templates.
        
        ![image.png](images/image%2024.png)
        
    - This interface contains a field called `DEFAULT_WRAPPER`.
        
        ![image.png](images/image%2025.png)
        
    - `DefaultObjectWrapperBuilder.build()` creates and returns a `DefaultObjectWrapper` object.
        
        ![image.png](images/image%2026.png)
        
    - Looking at the `DefaultObjectWrapper` documentation, we can see that it extends `BeansWrapper` . Therefore, having a `DefaultObjectWrapper` instance gives us access to the methods it inherits from `BeansWrapper`.
        
        ![image.png](images/image%2027.png)
        
    - `BeansWrapper` provides a `newInstance(Class clazz, List arguments)` method that can create an instance of a specified class. Remember that the `ec` variable still contains a `Class` object representing the `Execute` class ?
        
        ![image.png](images/image%2028.png)
        
    - Therefore, we pass `ec` to `dwf.newInstance(...)` to create an actual `Execute` object.
        
        ![image.png](images/image%2029.png)
        
    - The resulting `Execute` object can then be invoked with the command we want to execute.
        
        ```
        ${dwf.newInstance(ec,null)("id")}
        ```
        
        ![image.png](images/image%2030.png)
        
- Here’s the full chain:
    
    ![image.png](images/image%2031.png)
    
- And a “from the start” diagram:
    
    ```
    product
       │
       └── getClass()
              │
              ▼
           Class
              │
       getProtectionDomain()
              │
              ▼
       ProtectionDomain
              │
        getClassLoader()
              │
              ▼
         ClassLoader
            /      \
     loadClass()  loadClass()
        /             \
       ▼               ▼
    Execute.class   ObjectWrapper.class
                        │
                  getField()
                        │
                        ▼
                     Field
                        │
                     get()
                        │
                        ▼
                 DEFAULT_WRAPPER
                        │
                        ▼
                  BeansWrapper
                        │
                  newInstance()
                        │
                        ▼
                   Execute
                        │
                     ("id")
                        ▼
                       RCE
    ```
    

---

#### PART IV: Alternative Payload - Arbitrary File Read

- There is also an alternative payload in both `HackTricks` and the lab solution.
- This payload doesn't rely on instantiating any class. Instead, it chains getters and I/O methods on objects that already exist, so it never touches the `?new()` built-in at all. This makes it a simpler, more reliable route when the goal is just to read a file:
    
    ```
    ${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}
    ```
    
- `product.getClass().getProtectionDomain().getCodeSource().getLocation()`returns the `java.net.URL` of the JAR which the current class was loaded from. This is standard class-loading metadata, not an exploit-specific trick.
    
    ![image.png](images/image%2032.png)
    
- `.toURI().resolve('/home/carlos/my_password.txt')` is the actual exploit primitive. `URI.resolve()` is normally used to resolve a relative path against a base URI, but passing an absolute path causes the absolute path to replace the original path. This effectively turns a URI pointing to `freemarker.jar` into one pointing directly to an arbitrary file on disk. This is standard URI-resolution behavior, not a `FreeMarker` bug, but it's what makes the traversal possible here.
    
    ![image.png](images/image%2033.png)
    
- `.toURL().openStream().readAllBytes()` just opens and reads the file the resolved URI now points to, using plain `java.net.URL` I/O. Nothing reflection or instantiation-related, which is why this path is never caught by `TemplateClassResolver`.
- `?join(" ")` is a FreeMarker built-in that converts the byte array into a displayable string, since templates can't print raw bytes directly.
    
    ![image.png](images/image%2034.png)
    
- We then need to pass these bytes through an online converter to obtain the actual password.
    
    ![image.png](images/image%2035.png)
    
- This approach is less practical when the target file path is unknown, as it may require guessing or brute-forcing candidate paths using a wordlist.

---

#### PART V: Root Cause & Remediation

- Root Cause:
    - The core issue is that, in versions below 2.3.30, FreeMarker's sandbox controls security at the wrong layer: it blacklists specific class names at the `?new()` syntax level (TemplateClassResolver), but does NOT restrict which Java methods/fields can be accessed on objects already exposed to the template (via BeansWrapper).
    - This creates a security gap: any object passed into the template context, such as `product`, inherits methods from `java.lang.Object`, including `getClass()`. From there, parts of the Java reflection and class-loading APIs become reachable, including `ClassLoader`, `Field`, and `ProtectionDomain`. None of these are covered by the class-name blacklist, because none of them involve calling `?new()` directly.
    - The final piece is that FreeMarker's `BeansWrapper` exposes a `newInstance(Class, List)` method that can instantiate arbitrary classes. Since this is just an ordinary method call on a legitimate, pre-existing object (retrieved via the public static field `ObjectWrapper.DEFAULT_WRAPPER`), it completely bypasses the class-name check that only exists at the `?new()` entry point.
    - In short, the sandbox assumes that the only dangerous operation is creating a new object via `?new()`. However, reflection allows the same result to be achieved through a different code path that is not covered by the blacklist.
- Remediation:
    - Upgrade FreeMarker to a current supported version, including the security improvements introduced in 2.3.30. Versions starting with 2.3.30 introduced `MemberAccessPolicy`, which provides member-level access control in addition to the existing class-resolution restrictions (`DefaultMemberAccessPolicy` by default), This allows access to dangerous members, such as `getClass()`, to be restricted at the member-access level rather than only at the `?new()` level.
    - Do not expose raw application domain objects (like `product`) to templates through an unrestricted `BeansWrapper`, as this can expose their public Java methods to template code. Instead, use a restricted exposure level, such as: `beansWrapper.setExposureLevel(BeansWrapper.EXPOSE_SAFE)` or `EXPOSE_NOTHING`, 
    which can prevent template code from accessing `getClass()` and other sensitive methods, depending on the configured exposure policy.
    - Apply the principle of least privilege at the OS/JVM level as an additional layer of defense: 
    the `ProtectionDomain` information observed during exploitation shows that the process has `RuntimePermission("exitVM")` and file-system read access, the application should run under a user/service account with no access to sensitive files like `/home/carlos/my_password.txt`, so even a successful sandbox bypass yields no sensitive data.
    - Restrict who can edit templates in the first place. The root cause of this lab's attack surface being reachable at all is that the "Content Manager" role can submit raw FreeMarker syntax directly. Template editing should either be removed from untrusted roles or restricted to a predefined set of safe variables and placeholders rather than allowing free-form template code.
    - Treat template injection as a form of code injection: Any feature that allows a user, even a low-privileged authenticated user, to submit content that is parsed and executed as a template should undergo the same security review as a feature capable of executing arbitrary code.
