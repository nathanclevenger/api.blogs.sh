---
title: 'Principles of secure API Architecture design'
date: 2023-03-26
---

# Principles of Secure API Architecture Design

APIs have become a critical technology in the modern software development landscape, allowing businesses to easily build applications on top of their existing infrastructure, and providing developers with the tools they need to build powerful, data-driven applications. However, with the growing usage of APIs, the security of these applications has become increasingly important to both businesses and users. Therefore, it is crucial to follow proper security principles when designing and developing API architecture. 

In this blog post, we will discuss some of the essential principles of secure API architecture design. 

## Zero Trust Model

The Zero Trust Model (ZTM) approach to security is one of the most effective ways to protect against modern threats. ZTM assumes that all users and devices, whether inside or outside of the organization's network, are not trusted by default. Access to resources is granted on a need-to-know basis, without relying on traditional perimeter-based security. 

In API architecture, the ZTM approach means that APIs and APIs clients should be treated as untrusted by default. APIs should only allow access to authorized parties with proper credentials and authentication mechanisms. Proper authorization mechanisms must be enforced to ensure that only authorized users are obtaining access to the data. 

With the ZTM approach, API architecture can control who can access data, and which data can be accessed by them, thereby ensuring the security of data from internal and external threats.

## Use of OAuth for Authentication

OAuth (Open Authorization) is the industry-standard protocol for authorization. It provides an easy-to-use and standardized mechanism for handling identity and authorization. While OAuth itself may not be secure, when implemented correctly, it can provide a secure method of authorization, which allows users to authenticate themselves to API servers without revealing their credentials. 

For API architecture, OAuth can be used to protect against unauthorized access to data. An access token can be passed to the API server after the user's credentials are verified. If the user is authorized, the API can grant the user access to the data. This mechanism ensures that only authenticated and authorized parties have access to the API.

## Input Validation

Input validation is a crucial step in preventing attacks such as cross-site scripting (XSS) and SQL injection. APIs must validate all user input before processing it. APIs should reject any input that does not conform to expected formats, lengths, or valid character sets. This process ensures that the API is not vulnerable to attacks like XSS, SQL injection, and other similar attacks.

The following is a sample code that shows how input validation could be implemented for a basic login API.

```
router.post('/login', function (req, res) {
  const { email, password } = req.body;
  
  if (!email || !password) {
    return res.status(404).send({ message: 'Email and Password are Required' });
  }

  const saltedPassword = password + appSecret;
  const passwordHash = crypto.createHash('sha256').update(saltedPassword).digest('hex');

  const user = db.users.find(u => u.email === email && u.passwordHash === passwordHash);

  if (!user) {
    return res.status(404).send({ message: 'User not found' });
  }

  const token = jwt.sign({ email, scope: user.scope }, jwtSecret);
  res.json({ token });
});
```

The code checks that both email and password are present in the input submission. If either is missing, the API will return the appropriate error message. If the email and password are present, the API will search for the specific user accurately, based on the provided set of credentials. If the user cannot be found, the API returns a 404 error. If the user is located, the API validates the users' authorization scope and returns them a JSON Web Token (JWT).

## Rate limiting

API rate limiting is a method to protect against mass attacks that consume system resources, including Distributed Denial of Service (DDoS) attacks. API request limiters ensure that the 'API server' optimizes the available resources by limiting the number of requests it receives. Setting request limits is one of the better methods to protect an API from being attacked or overloaded. 

The following code sample shows a simple rate limiter that limits the number of requests to an API server.

```
let violations = {}; 

// Middleware to limit the number of requests per second 
export const limit = ratelimit({
  windowMs: 1000,
  max: 10,
  onLimitReached: (req, res, opts) => {
   if (violations[opts.key]) {
        // Increase count if requests exceed the limit 
        violations[opts.key] += 1;
      } else {
        // Setting count to 1 if this is the first violation 
        violations[opts.key] = 1;
      }
      console.log("Too many requests;")
  }
}); 
``` 

The code ensures that users cannot exceed ten requests against any API endpoint. If more requests are made within a second, the API server logs the violation, and the server takes an appropriate action.  

## Dependency Auditing 

APIs regularly depend on outside systems and libraries to function correctly. Any dependencies on libraries pose a risk. A flaw in one library can have a considerable impact on the entire network. Regularly auditing the dependencies for vulnerabilities and patches is crucial. Although third-party libraries are convenient for API design, they could also be the source of vulnerabilities. 

The following code shows how to audit system dependencies with minimal effort.

```
$ npm install -g nsp
npm audit
npm audit fix
``` 

This code provides insights into your system dependencies and packages that may have vulnerabilities. It also automatically installs patches that are compatible with your API architecture.

## Conclusion

APIs are essential in facilitating fast and secure data exchange between systems. Ensuring the security of APIs is vital for businesses to build customer trust as data breaches and attacks can have devastating consequences. This blog post discussed some of the essential principles for secure API architecture design, including the Zero Trust Model, input validation, OAuth authentication, rate limiting, and dependency auditing. By following these principles, businesses can secure their APIs against modern threats and protect important data from unauthorized access.
