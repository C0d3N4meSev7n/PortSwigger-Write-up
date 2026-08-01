# Authentication-2

---

### Broken brute-force protection, multiple credentials per request (Expert)

---

#### Description:

#### This lab is vulnerable due to a logic flaw in its brute-force protection. To solve the lab, brute-force Carlos's password, then access his account page.

#### Victim's username: carlos
Candidate passwords (password list)

---

- Accessing the lab, I immediately go to `My account` page.
    
    ![image.png](images/image.png)
    
- Then try logging in with username carlos to see if there’s any lockout mechanism.
    
    ![image.png](images/image%201.png)
    
- After 4 failed attempts, the application blocks me for 1 min (still being able to brute-force with small wordlist, just takes more time, but it won’t be a good choice)
    
    ![image.png](images/image%202.png)
    
- Then I try changing the value of the session cookie to see if the application is using per-session lockout, and adding the header `X-Forwarded-For` to check for per-ip lockout either, but the result shows that I still have to wait after doing both of them, meaning the application is using per-account lockout.
    
    ![image.png](images/image%203.png)
    
- I notice something looks strange with this POST request, usually the content-type of these login forms should be `application/x-www-form-urlencoded` , because the server expected only a single string for these type of input. But this application uses JSON instead, making it becomes vulnerable and completely erase the purpose of using lockout mechanism.
    
    ![image.png](images/image%204.png)
    
- Since JSON supports multiple data types, I wonder whether the backend validates the password is actually a string , if it simply iterates over whatever value is provided, it can lead to the entire password list to be processed within a single request…like this, just need to capture the logged in session cookie and replace it in the browser to solve the lab.
    
    ![image.png](images/image%205.png)
    
    ![image.png](images/image%206.png)
    
- Root cause:
    - Missing type validation on the `password` field: The backend
    expects a single string but never rejects a JSON array, so it
    accepts and processes whatever shape the client sends.
    - Rate limiting is enforced per HTTP request, not per actual credential
    check: When the array is looped through server-side to test each
    candidate password, all of that happens inside a single request, so
    the per-account lockout counter only registers one attempt no matter
    how many passwords were actually tried.
    - Combined effect: a per-account lockout that looks solid under normal
    (single-credential) testing becomes fully bypassable once the input
    shape is changed, allowing an entire password list to be tried in
    one request instead of one attempt at a time.
