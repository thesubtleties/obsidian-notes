### **How Cookies Work**
Cookies are small pieces of data that websites store in your browser. They’re used to maintain state (e.g., keeping you logged in) across different requests. Here’s the breakdown:

1. **Cookie Scope:**
   - Cookies are tied to a specific **domain** (e.g., `yourbank.com`) and sometimes a specific **path** (e.g., `/account`).
   - When you visit a website, the browser only sends cookies that match the domain and path of the request.

2. **Same-Origin Policy:**
   - Browsers enforce the **Same-Origin Policy**, which means cookies are only sent to the site that created them.
   - For example, cookies for `yourbank.com` are only sent to `yourbank.com`, not to `evil-site.com`.

3. **Automatic Inclusion:**
   - When you make a request to a site (e.g., `yourbank.com`), your browser automatically includes all cookies that belong to that site.
   - This happens for **every request**, whether it’s a link click, form submission, or even an image load.

---

### **Why Does the Browser Send Cookies Automatically?**
Cookies are designed to maintain state across requests. For example:
- **Session Cookies:** Keep you logged in as you navigate a website.
- **Preferences Cookies:** Remember your settings (e.g., dark mode, language).

This automatic behavior is useful for legitimate purposes but can be exploited in CSRF attacks.

---

### **Why Doesn’t the Browser Send Cookies to Every Site?**
Cookies are **domain-specific**. This means:
- Cookies for `yourbank.com` are only sent to `yourbank.com`.
- Cookies for `evil-site.com` are only sent to `evil-site.com`.

This separation is enforced by the browser to prevent one site from accessing another site’s data. Without this, any site could steal your cookies and impersonate you on other sites.

---

### **How CSRF Exploits This**
In a CSRF attack:
4. You’re logged into `yourbank.com`, so your browser has a session cookie for `yourbank.com`.
5. You visit `evil-site.com`, which contains a form or button that makes a request to `yourbank.com`.
6. When your browser sends the request to `yourbank.com`, it automatically includes the session cookie for `yourbank.com` (because the request is going to `yourbank.com`).
7. The bank sees the request as legitimate because it comes with your valid session cookie.

---

### **Why Doesn’t the Browser Block This?**
The browser doesn’t block this because:
- The request is going to the correct domain (`yourbank.com`), so the browser includes the cookies for that domain.
- The browser has no way of knowing whether the request was initiated by you or by a malicious site.

---

### **How to Prevent This**
To stop CSRF, websites use **anti-CSRF tokens** or other mechanisms like **SameSite cookies**:
8. **Anti-CSRF Tokens:**
   - The server generates a unique token for each form or request.
   - The token is included in the form and validated by the server.
   - Since the malicious site can’t access the token, the request fails.

9. **SameSite Cookies:**
   - Modern browsers support the `SameSite` attribute for cookies.
   - Setting `SameSite=Strict` or `SameSite=Lax` prevents the browser from sending cookies in cross-site requests.
   - Example:
     ```http
     Set-Cookie: sessionId=abc123; SameSite=Strict; Secure
     ```

10. **Custom Headers:**
   - APIs can require custom headers (e.g., `X-CSRF-Token`) for state-changing requests.
   - Since browsers block cross-origin requests from setting custom headers, this prevents CSRF.

---

### **Key Takeaways**
- Cookies are **domain-specific** and only sent to the site that created them.
- The browser automatically includes cookies with every request to the matching domain.
- CSRF exploits this behavior by tricking your browser into making requests to a site where you’re logged in.
- Anti-CSRF tokens, SameSite cookies, and custom headers are effective defenses.

By understanding this, you can see why CSRF is possible and how to prevent it! 