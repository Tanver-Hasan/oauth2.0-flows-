

---
Email verification flow via OTP 
---

```mermaid
sequenceDiagram
    autonumber
    participant User
    participant Application
    participant Auth0
    participant PostLoginAction as Post-Login Action
    participant Auth0Forms as Auth0 Form
    participant SendGrid as SendGrid
    
    User ->> Application: Attempt to login / Register
    Application ->> Auth0: Redirect to Authorization Server (/authorize)
    note right of Application: Execute Authorization Code flow + PKCE
    Auth0 ->> Auth0: Authenticate the user
    Auth0 ->> PostLoginAction: Trigger Post-Login Action
    PostLoginAction ->> PostLoginAction: Check if email is verified
    alt Email not verified
        PostLoginAction ->> Auth0Forms: Trigger Auth0 Form to verify email
        Auth0Forms ->> SendGrid: Send OTP code to user's email
        Auth0Forms ->> User: Display form to enter OTP
        SendGrid ->> User: Receive OTP code via email
        User ->> Auth0Forms: Enter OTP code in the form
        Auth0Forms ->> PostLoginAction: Verify OTP code
        PostLoginAction ->> Auth0: Update user email as verified 
    end
    Auth0 ->> Application: Resume transaction and redirect user
    note left of Auth0 : Reuturns authz code and <br> Auth0 exchanges the code for token
    Application ->> User: User is logged in with verified email
```