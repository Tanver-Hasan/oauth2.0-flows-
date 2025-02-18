
### Option 1-  Interactive Reset Flow triggered from Login Screen (Built into the universal login page) 
	
>The user can click the Don't remember your password link on the Login screen and then enter their email address. This fires off a POST request to Auth0 that triggers the password reset process. The user receives a password reset email.

```mermaid
sequenceDiagram
    actor User 
    participant App as Application 
    participant Tenant as Okta CIC (Auth0)
    autonumber

    User->>App:  Login 
    App->>Tenant:  /authorize
    note left of Tenant :  Universal Login Redirect
    Tenant-->>User:  Universal Login Page

    User->>Tenant: Click on Forgot Password button
    Tenant-->>User: Prompt to enter email 
    User->>Tenant:  User Enter email address and submit request 
    Tenant->>User: Auth0 sends an email with magic link 
    note  right of User: Email sent only if email exist in the system

    User->> Tenant:  Click on Password Reset Link from the inbox
    note right of User: User redirected to the Universal Loign Change Password Page
    Tenant->>Tenant:  Verify Password Reset Magic Link
    
    opt If MFA Enforced
    Tenant->>User :  MFA Challenge
    User -->> Tenant:  MFA Response
    end

    Tenant->>User:  Universal Login Page Change Password Prompt
    User->> Tenant : Enter New Password & Confirm
    User->>Tenant:  Submit New Password
    Tenant-->>Tenant:  New Password Set
    Tenant -->> User : Password Reset Confirmation Screen 
    note left of Tenant : Confirmation screen shows button <br> nagivate back to the application Login Page / Home Page


    User->>App:  Return to Login Page
    App->>Tenant:  Redirect to Universal Login Page
    Tenant-->>User : Universal Login Page

    note right of User: * User can now log in with the new password *

```

Considerations: 

* This is feature is built into Auth0 Platfrom. Therefore, does not require any additional secuirty risk in the application layer 
* Does not require user to enter rember old password 

### Option 2: Interactive Password Reset Flow from Profile Page of the application  via Authentication API Change Password Endpoint

> Add an option in the profile page for resetting the password (no inputs are required, as pwd will be set later, and email can be taken from the logged in user). Trigger an interactive password reset flow through the Authentication API, by making a POST call passing the email field. If the call is successful, the user receives a password reset email.
```mermaid
sequenceDiagram
    actor User 
    participant App as Application 
    participant Tenant as Okta CIC (Auth0)
    autonumber
    %% section [A] User Initiates Login
    User->>App:  Login 
    App->>Tenant:  /authorize
    note left of Tenant :  Universal Login Redirect
    Tenant-->>User:  Universal Login Page
    %% end

    %% section [B] User Initiates Password Reset
    User->>App: Click on Forgot Password button on the profile page
    App-->>User: Prompt to enter email 
    User->>App:  User Enter email address and submit request 
    App->>Tenant: Password Reset Request via Authentication API change <br>password endpoint <br> (/dbconnections/change_password)
    Tenant-->>User: Email delivered with Password Reset Magic Link
    note  right of User: Email sent only if email exist in the system
    App-->>User:  Email Dispatch Confirmation
    %% end

    %% section [C] User Performs Password Reset
    User->> Tenant:  Click on Password Reset Link from the inbox
    note right of User: User redirected to the Universal Loign Change Password Page
    Tenant->>Tenant:  Verify Password Reset Magic Link
    
    opt If MFA Enforced
    Tenant->>User :  MFA Challenge
    User -->> Tenant:  MFA Response
    end

    Tenant->>User:  Universal Login Page Change Password Prompt
    User->> Tenant : Enter New Password & Confirm
    User->>Tenant:  Submit New Password
    Tenant-->>Tenant:  New Password Set
    Tenant -->> User : Password Reset Confirmation Screen 
    note left of Tenant : Confirmation screen shows button <br> nagivate back to the application Login Page / Home Page
    %% end

    %% section [D] User Navigates to Universal Login Page
    User->>App:  Return to Login Page
    App->>Tenant:  Redirect to Universal Login Page
    Tenant-->>User : Universal Login Page

    note right of User: * User can now log in with the new password *
    %% end
```

Considerations : 
* Similar to Option 1 but triggers the password reset flow from the application 
* Does not require user to remember the current password 

### Option 3: Self Service Password Reset Flow via Management API (Directly set the new password )
>Add a form in the profile page, which includes the new password field. Directly set the new password via Management API without sending a password reset email.
Optionally, sent an email after the password change via an action Post Change Password Trigger 

```mermaid
sequenceDiagram
    actor User 
    participant App as Application 
    participant Tenant as Okta CIC (Auth0)
    Participant API as Auth0 Management API Proxy
    autonumber
    %%  User Initiates Login
    User->>App:  Login 
    App->>Tenant:  /authorize
    note left of Tenant :  Universal Login Page Redirect
    Tenant-->>User:  Universal Login Page
    User ->> Tenant : User enters the Email + Password 
    Tenant ->> Tenant : User credentials verification complete
    Tenant --> App : Redirect to /callback endpoint
    note right of App : Redirected Back to Application callback URL with token <br> (Access Token, idToken , Refresh token)
    App -->> User: User Logged In

    User->> App:  User goes to application profile <br> Page for password change 
    App-->> User : Applicatin prompts the user the user <br>to enter old password and new password 
    User->>App:  Submit Old & New Password
    App ->> API : Call Passowrd change endpoint 
    note left of API: API access token passed as authorization header
    note right of App: Passed both old and new password 
    critical [Verify Old Password]
    API -->> Tenant:  Verify Old Password (/oauth/token)
    note right of Tenant : Verify old password by executing ROPG flow 
    end
    API ->> Tenant  : Obtain Management API token 
    Tenant -->> API: Return Response (Access Token)
    note left of API : Obtain Management API token using <br> client credentails flow 
    API ->> Tenant : Update the new password 
    note left of API: Update new password by calling Management <br> API Users Patch (PATCH /api/v2/users) endpoint 
    Tenant -->> API: Return Response (HTTP status code 200) 
    Tenant -->> API: Return Response (HTTP status code 200) 
    API -->> App: Return Response 
    App -->> User: Show Success Message 
    App ->> Tenant : Logout the user from app and force re-login

    %% section [D] User Navigates to Universal Login Page
    User->>App:  Return to Login Page
    App->>Tenant:  Redirect to Universal Login Page
    Tenant-->>User : Universal Login Page
    note right of User: * User can now log in with the new password *
    %% end
```

Pros & Cons: 

* This features requires application to handle the collection of old and new password. Thefore, application must ensure that the communication is safe between application and backend API. 
* The user must be logged out after successful password reset and force the user to login again 

### Option 4: Change User Passwrod without sending an email or directly setting it from the application

> Today, Auth0 supports two methods of changing a password: (1) the interactive password reset flow or (2) via the Auth0 Management API. In cases whereby Customers have a requirement to offer users the ability to change their password without resetting it via email, a custom Change Password Flow implemented via the API is their only documented solution. This introduces security risk as the application is forced to handle user passwords. Additionally, if the customer requires the user to re-authenticate then this may introduce a dependency on the Resource Owner Password Grant, which is expected to be omitted from the OAuth 2.1 specification. This wiki page describes a method that leverages a Password Change Ticket and an Auth0 Post-Login Action to implement a Custom Change Password flow that first authenticates the user with their existing credentials and then enables them to change their password. Both of those steps are completed directly within Universal Login. 

>This solution relies on Forced Re-authentication, which must be implemented correctly to avoid being bypassed by malicious actors. Specifically, the Relying Party must always validate the auth_time claim within the ID Token. See more. An intentional design consideration of this solution is that the ID token is never returned to the Relying Party. Additionally, the Change Password event is successfully executed before the point at which the ID token would usually be returned to the Relying Party. Therefore, the Post-Login Action must instead implement a check to ensure that the prompt=login or max_age=0 query parameters have been passed into the request to the /authorize endpoint before generating the Password Change Ticket. 

```mermaid
sequenceDiagram
    actor User 
    participant App as Application 
    participant Tenant as Okta CIC (Auth0)
    autonumber
    %% section [A] User Initiates Login
    User->>App:  Login 
    App->>Tenant:  /authorize
    note left of Tenant :  Universal Login Redirect
    Tenant-->>User:  Universal Login Page
    %% end

    %% section [B] User Initiates Password Reset
    User->>App: User goes goes to Profile page and trigger change password flow
    App->>Tenant:  Redirected to Universal Login Page 
    note right of App: User is redirected to the /authorize endpoint with prompt=login param . <br> Pass addtional scope parameter to triger password reset flow (scope=update:password). <br> Additionally, pass login_hint 
    Tenant-->>User: Returns Universal Login Page and prompts to enter existing email and password
    User ->> Tenant: User submit existing email password
    Tenant ->> Tenant : Existing email and password gets validated

    opt If MFA Enforced
    Tenant->>User :  MFA Challenge
    User -->> Tenant:  MFA Response
    end

    Tenant->>Tenant:  Post Login  Password cahnge Actions gets triggered 
    note left of Tenant: The action obtains a management API token <br> by execting client credentails flow and calls Managment API Password change ticket endpoint 
    Tenant -->> User : Render the user to password change URL 
    User->>Tenant:  User enters new Password 
    Tenant-->>Tenant:  New Password Set
    Tenant -->> User : Password Reset Confirmation Screen 
    note left of Tenant : Confirmation screen shows button <br> nagivate back to the application Login Page / Home Page
    %% end

    %% section [D] User Navigates to Universal Login Page
    User->>App:  Return to Login Page
    App->>Tenant:  Redirect to Universal Login Page
    Tenant-->>User : Universal Login Page

    note right of User: * User can now log in with the new password *
    %% end
```

Consideration: 

* Low secuirty risk similar to Option 1 and Option 2. 
* Does not require user sending password reset magic link 

Sample Post Login Action 

```
/**
* Handler that will be called during the execution of a PostLogin flow.
*
* @param {Event} event - Details about the user and the context in which they are logging in.
* @param {PostLoginAPI} api - Interface whose methods can be used to change the behavior of the login.
*/

const auth0Sdk = require("auth0");

exports.onExecutePostLogin = async (event, api) => {

  const ManagementClient = auth0Sdk.ManagementClient;

  // This will make a Management API call
  const managementClientInstance = new ManagementClient({
    // These come from a machine-to-machine application
    domain: 'YOUR_DOMAIN',
    clientId: 'YOUR_CLIENT_ID',
    clientSecret: event.secrets.YOUR_CLIENT_SECRET,
    scope: "create:user_tickets"
  });

/**
* Important Security Consideration: This script MUST check that the 'prompt=login' or 'max_age=0' query 
* parameters were passed into the request to the /authorize endpoint. This mitigates the risk of malicous 
* user agents stripping these query parameters out to bypass re-authentication.
*/
  if(event.transaction?.requested_scopes?.includes('update:password') && 'max_age' in event.request.query){
    try {
      let ticket = await managementClientInstance.createPasswordChangeTicket({
        user_id: event.user.user_id,
        client_id: event.client.client_id
      });
    console.log(ticket);
    api.redirect.sendUserTo(ticket.ticket);
    } catch (err) {
      console.log(err);
    }
  }
};
```