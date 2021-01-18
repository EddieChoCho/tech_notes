# JWT - JSON Web Tokens
## When should you use JSON Web Tokens?
* Authorization: Single Sign On is a feature that widely uses JWT nowadays, because of its small overhead and its ability to be easily used across different domains.
* Information Exchange: Because JWTs can be signed you can be sure the senders are who they say they are. Additionally, as the signature is calculated using the header and the payload, you can also verify that the content hasn't been tampered with.

## How do JSON Web Tokens work?
* In authentication, when the user successfully logs in using their credentials, a JSON Web Token will be returned.

* You should not keep tokens longer than required.

* You should not store sensitive session data in browser storage due to lack of security.

* Whenever the user wants to access a protected route or resource, the user agent should send the JWT, typically in the Authorization header using the Bearer schema.
    ```
    Authorization: Bearer <token>
    ```

* The server's protected routes will check for a valid JWT in the Authorization header, and if it's present, the user will be allowed to access protected resources. If the JWT contains the necessary data, the need to query the database for certain operations may be reduced. 

* If the token is sent in the `Authorization` header, Cross-Origin Resource Sharing (CORS) won't be an issue as it doesn't use cookies.

* Do note that with signed tokens, all the information contained within the token is exposed to users or other parties, even though they are unable to change it. This means you should not put secret information within the token.
    

## References
* [1][Introduction to JSON Web Tokens](https://jwt.io/introduction)
* [2][OWASP Cheat Sheet Series - Local-storage](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html#local-storage)