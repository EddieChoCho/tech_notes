# Vulnerabilities of Password Reset Feature

* Token of rest-password URL
    * Token should not be re-used.
    * Token should have expiration time.
    * If a new request has been created, the token of the old one should be deprecated.

* Rate limiting
    * !X-Forwarded-For

## References

* [有缺陷的重設密碼機制如何演變成帳號奪取漏洞？以 Matters 為例](https://tech-blog.cymetrics.io/posts/huli/reset-password-vulnerability/)