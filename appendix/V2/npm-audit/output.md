# Contracts

```bash

@openzeppelin/contracts  4.5.0 - 4.9.5
OpenZeppelin Contracts base64 encoding may read from potentially dirty memory - https://github.com/advisories/GHSA-9vx6-7xxf-x967
fix available via `npm audit fix`
node_modules/@openzeppelin/contracts

braces  <3.0.3
Severity: high
Uncontrolled resource consumption in braces - https://github.com/advisories/GHSA-grv7-fg5c-xmjg
fix available via `npm audit fix`
node_modules/braces

follow-redirects  <=1.15.5
Severity: moderate
Follow Redirects improperly handles URLs in the url.parse() function - https://github.com/advisories/GHSA-jchw-25xp-jwwc
follow-redirects' Proxy-Authorization header kept across hosts - https://github.com/advisories/GHSA-cxjh-pqwp-8mfp
fix available via `npm audit fix`
node_modules/follow-redirects

undici  <=5.28.3
Undici's Proxy-Authorization header not cleared on cross-origin redirect for dispatch, request, stream, pipeline - https://github.com/advisories/GHSA-m4v8-wqvr-p9f7      
Undici's fetch with integrity option is too lax when algorithm is specified but hash value is in incorrect - https://github.com/advisories/GHSA-9qxr-qj54-h672
Undici proxy-authorization header not cleared on cross-origin redirect in fetch - https://github.com/advisories/GHSA-3787-6prv-h9w3
fix available via `npm audit fix`
node_modules/undici

4 vulnerabilities (2 low, 1 moderate, 1 high)
```