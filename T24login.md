That's an excellent and very specific question that gets to the technical heart of how T24's modern API architecture functions. The method of returning a session token is different depending on whether you're using the older T24 Browser/OFS (Open Financial Service) or the newer, more modern APIs.

Here is a breakdown of how a user access session token is returned.

### 1\. The Traditional T24 Model (Browserweb/OFS)

In the older architecture, the concept of a "token" was more of an internal T24 session identifier.

  * **Generation**: When a user successfully signs on, the T24 core system, via its `OFS.SESSION.MANAGER` routine, generates a unique, proprietary token. This token is a number with a specific format that encodes information like the user number, time, and sequence number.
  * **Storage**: This token is stored in T24 database files (e.g., `F.OS.TOKEN`) and linked to the user's session.
  * **Return**: The token is **appended to the response** that T24 sends back to the application server. The application server then separates the token from the response data. This token is then used by the browser for all subsequent requests to the T24 core, essentially acting as the session ID for that connection. The token itself is not returned as a JSON object; it's part of the proprietary message format.

### 2\. The Modern API Model (Transact Explorer)

The modern Transact Explorer and its associated APIs use a completely different, industry-standard approach based on **JSON Web Tokens (JWTs)**. This is a crucial distinction.

Here is the step-by-step process of how a user access token is returned in this modern architecture:

1.  **Authentication Request**: A user or client application sends a login request (e.g., a username and password) to T24's **API Gateway** or a dedicated authentication service. This service acts as the **Authorization Server**.

2.  **Validation and Profile Retrieval**: The Authorization Server validates the credentials (or the SAML assertion in an SSO scenario). Upon successful validation, it retrieves the user's profile from the T24 `USER` record. This includes the assigned T24 `GROUP`s and other permissions.

3.  **JWT Generation**: The Authorization Server then generates a JWT. A JWT is a self-contained, digitally signed token that contains a payload of information called "claims." These claims can include:

      * **User Information**: `user_id`, `sign.on.name`, etc.
      * **Permissions**: The T24 `GROUP`s and the specific business functions the user is allowed to perform.
      * **Expiration**: A timestamp indicating when the token will expire.
      * **Issuer**: Who issued the token (T24's Authorization Server).

4.  **Token Return**: The T24 Authorization Server returns the generated JWT to the client application (e.g., the Transact Explorer browser). The token is returned as part of a JSON response, typically in a field named `access_token`.

    ```json
    {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2bJ",
      "token_type": "Bearer",
      "expires_in": 3600
    }
    ```

5.  **Subsequent API Calls**: The client application (e.g., your browser) then stores this token locally. For every subsequent API call to perform a T24 action (e.g., `GET /accounts`), the client includes this token in the `Authorization` header of the HTTP request, formatted as `Authorization: Bearer <your_jwt>`.

6.  **Token Validation by the API Gateway**: The T24 API Gateway intercepts every incoming API request. It reads the JWT from the `Authorization` header and performs two crucial checks:

      * **Integrity**: It verifies the digital signature of the token to ensure it has not been tampered with.
      * **Validity**: It checks if the token is still valid (not expired).
      * **Authorization**: It reads the claims within the token to determine if the user has permission to perform the requested action. For example, if the API call is to `POST /funds-transfer` and the token's claims do not include `funds.transfer:input` permission, the request is denied.

In the modern API model, the token is a secure, self-contained credential that proves the user's identity and permissions for the duration of the token's validity, making the process stateless and highly scalable.
