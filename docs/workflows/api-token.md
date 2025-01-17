---
title: API Tokens
sidebar_position: 6
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# API Token Overview

All [Beyond Identity API](https://developer.beyondidentity.com/api/v1) endpoints require authentication using an access token. The access token is generated through OAuth 2.0 or OIDC, using the authorization code flow or the client credentials flow. All access tokens are JWT. Two token formats are supported: self-contained tokens as JWS and referential tokens as JWE. The default token on creation is self-contained.

You can obtain an access token either through:

1. [Beyond Identity Admin Console](api-token#create-token-with-the-beyond-identity-admin-console)
2. [Beyond Identity APIs](api-token#create-token-with-the-beyond-identity-api)

The access token should be generated with your Beyond Identity Management API application in order to access any of the Beyond Identity APIs. The "Beyond Identity Management API" application is provided by default as part of the tenant onboarding process.

The access token must be provided in the Authorization header of an API request like the following:

```bash
curl https://api-us.beyondidentity.com/v1/... \
-X $HTTP_METHOD -H "Authorization: Bearer $TOKEN"
```

API access tokens are valid for 3 months (TTL 7776000 seconds). You can restrict the token's access with [scopes](../apis/scopes) by selecting a list in the Beyond Identity Admin Console or specifying a space-separated string of scopes in your API request, in accordance with [RFC6749#3.3](https://datatracker.ietf.org/doc/html/rfc6749#section-3.3). Token expiration can be configured from your "Beyond Identity Management API" application.

## Create Token with the Beyond Identity Admin Console

The simplest way to acquire an access token is through the Beyond Identity Admin Console. Under the "Apps" tab, select the "Beyond Identity Management API" application, navigate to the "API TOKENS" tab, and then click on "Create token". From there you can configure the token with a Name and specified list of Scopes.

<div style={{position: 'relative', paddingBottom: 'calc(73% + 20px)', height: '0'}}>
<iframe src='https://demo.arcade.software/OQge5lspW7TRuqvghZQd?embed&forceNoOpeningAnimation=true' frameBorder="0" style={{position: 'absolute', top: '0', left: '0', width: '100%', height: '100%'}}>
</iframe>
</div>

## Create Token with the Beyond Identity API

Alternatively, an access token may also be generated directly via API by requesting a token for the `Beyond Identity Management API` application. You will need the "Beyond Identity Management API" application's `CLIENT_ID`, `CLIENT_SECRET` and `APPLICATION_ID`. These values can be found either through the Beyond Identity Admin Console under the "Beyond Identity Admin" realm and selecting the "Beyond Identity Management API" application, or by [API](https://developer.beyondidentity.com/api/v1) after retrieving the management application. In accordance with [RFC6749#3.3](https://datatracker.ietf.org/doc/html/rfc6749#section-3.3), the scopes are expressed as a space-delimited string. If you do not specify a list of scopes, all of the available scopes will be assigned to your token on creation. For more information on available scopes, check out [Scopes](../apis/scopes).

### Client Credentials Flow

<Tabs groupId="api-token-platform" queryString>
<TabItem value="curl" label="Curl">

```bash title="/token"
curl https://auth-$REGION.beyondidentity.com/v1/tenants/$TENANT_ID/realms/$REALM_ID/applications/$MANAGEMENT_APPLICATION_ID/token \
-X POST \
-u "$MANAGEMENT_API_CLIENT_ID:$MANAGEMENT_API_CLIENT_SECRET" --basic \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=client_credentials&scope=$SCOPES"
```

</TabItem>
<TabItem value="node" label="Node">

```jsx title="/token"
const apiTokenResponse = await fetch(
  `https://auth-${REGION}.beyondidentity.com/v1/tenants/${TENANT_ID}/realms/${REALM_ID}/applications/${MANAGEMENT_APPLICATION_ID}/token`,
  {
    body: (() => {
      let formData = new URLSearchParams();
      formData.append('grant_type', 'client_credentials');
      return formData;
    })(),
    headers: {
      Authorization:
        'Basic ' +
        Buffer.from(
          `${MANAGEMENT_API_CLIENT_ID}:${MANAGEMENT_API_CLIENT_SECRET}`
        ).toString('base64'),
    },
    method: 'POST',
  }
);

const apiTokenResponseJson = await tokenResponse.json();
const accessToken = apiTokenResponseJson.access_token;
```

</TabItem>
<TabItem value="python" label="Python">

```python title="/token"
import requests

data = {
'grant_type': 'client_credentials',
'scope': 'SCOPES',
}

response = requests.post(
'https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/token',
data=data,
auth=('MANAGEMENT_API_CLIENT_ID', 'MANAGEMENT_API_CLIENT_SECRET'),
)
```

</TabItem>
<TabItem value="go" label="Go">

```go title="/token"
package main

import (
"fmt"
"io/ioutil"
"log"
"net/http"
"strings"
)

func main() {
client := &http.Client{}
var data = strings.NewReader(`grant_type=client_credentials&scope=SCOPES`)
req, err := http.NewRequest(http.MethodPost, "https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/token", data)
if err != nil {
  log.Fatal(err)
}
req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
req.SetBasicAuth("MANAGEMENT_API_CLIENT_ID", "MANAGEMENT_API_CLIENT_SECRET")
resp, err := client.Do(req)
if err != nil {
  log.Fatal(err)
}
defer resp.Body.Close()
bodyText, err := ioutil.ReadAll(resp.Body)
if err != nil {
  log.Fatal(err)
}
fmt.Printf("%s\n", bodyText)
}
```

</TabItem>
</Tabs>

### Authorization Code Flow

Using the authorization code flow is a two part process. First an authorization grant code must be obtained. This code is recieved through your callback specified in the `redirect_uri`. When extracting the code, your `state` and PKCE should be validated. Second you must use the grant code to create an access token.

Use the following curl examples below to obtain an authorization code and then create a token with that code. Note the following example uses `S256`, but using the `plain` `code_challenge_method` might be easier to get started as using PKCE requires storing the hash of the value passed as `code_challenge` so it can be passed to the token endpoint later.

1. Authenticate to obtain an authorization code:

<Tabs groupId="api-token-platform" queryString>
<TabItem value="curl" label="Curl">

```bash title="/authorize"
curl https://auth-$REGION.beyondidentity.com/v1/tenants/$TENANT_ID/realms/$REALM_ID/applications/$MANAGEMENT_APPLICATION_ID/authorize \
-d response_type=code \
-d client_id=$MANAGEMENT_API_CLIENT_ID \
-d redirect_uri=$REDIRECT_URI \
-d scope=openid \
-d state=$STATE \
-d code_challenge=$OPTIONAL_CODE_CHALLENGE \
-d code_challenge_method=S256 \
```

</TabItem>
<TabItem value="node" label="Node">

```jsx title="/authorize"
fetch(
  `https://auth-${REGION}.beyondidentity.com/v1/tenants/${TENANT_ID}/realms/${REALM_ID}/applications/${MANAGEMENT_APPLICATION_ID}/authorize`,
  {
    method: 'POST',
    body: new URLSearchParams({
      response_type: 'code',
      client_id: `${MANAGEMENT_API_CLIENT_ID}`,
      redirect_uri: `${REDIRECT_URI}`,
      scope: 'openid',
      state: `${STATE}`,
      code_challenge: `${OPTIONAL_CODE_CHALLENGE}`,
      code_challenge_method: 'S256',
    }),
  }
);
```

</TabItem>
<TabItem value="python" label="Python">

```python title="/authorize"
import requests

data = {
    'response_type': 'code',
    'client_id': 'MANAGEMENT_API_CLIENT_ID',
    'redirect_uri': 'REDIRECT_URI',
    'scope': 'openid',
    'state': 'STATE',
    'code_challenge': 'OPTIONAL_CODE_CHALLENGE',
    'code_challenge_method': 'S256',
}

response = requests.post(
    'https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/authorize',
    data=data,
)
```

</TabItem>
<TabItem value="go" label="Go">

```go title="/authorize"
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"strings"
)

func main() {
	client := &http.Client{}
	var data = strings.NewReader(`response_type=code&client_id=MANAGEMENT_API_CLIENT_ID&redirect_uri=REDIRECT_URI&scope=openid&state=STATE&code_challenge=OPTIONAL_CODE_CHALLENGE&code_challenge_method=S256`)
	req, err := http.NewRequest(http.MethodPost, "https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/authorize", data)
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	bodyText, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", bodyText)
}
```

</TabItem>
</Tabs>

2. Create an access token the with authorization code:

<Tabs groupId="api-token-platform" queryString>
<TabItem value="curl" label="Curl">

```bash title="/token"
curl https://auth-$REGION.beyondidentity.com/v1/tenants/$TENANT_ID/realms/$REALM_ID/applications/$MANAGEMENT_APPLICATION_ID/token \
-X POST \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=authorization_code&code=$CODE&scope=$SCOPES&client_id=$MANAGEMENT_API_CLIENT_ID&code_verifier=$OPTIONAL_CODE_VERIFIER"
```

</TabItem>
<TabItem value="node" label="Node">

```jsx title="/token"
fetch(
  `https://auth-${REGION}.beyondidentity.com/v1/tenants/${TENANT_ID}/realms/${REALM_ID}/applications/${MANAGEMENT_APPLICATION_ID}/token`,
  {
    method: 'POST',
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code: `${CODE}`,
      scope: `${SCOPES}`,
      client_id: `${MANAGEMENT_API_CLIENT_ID}`,
      code_verifier: `${OPTIONAL_CODE_VERIFIER}`,
    }),
  }
);
```

</TabItem>
<TabItem value="python" label="Python">

```python title="/token"
import requests

data = {
'grant_type': 'authorization_code',
'code': 'CODE',
'scope': 'SCOPES',
'client_id': 'MANAGEMENT_API_CLIENT_ID',
'code_verifier': 'OPTIONAL_CODE_VERIFIER',
}

response = requests.post(
'https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/token',
data=data,
)
```

</TabItem>
<TabItem value="go" label="Go">

```go title="/token"
package main

import (
"fmt"
"io/ioutil"
"log"
"net/http"
"strings"
)

func main() {
client := &http.Client{}
var data = strings.NewReader(`grant_type=authorization_code&code=CODE&scope=SCOPES&client_id=MANAGEMENT_API_CLIENT_ID&code_verifier=OPTIONAL_CODE_VERIFIER`)
req, err := http.NewRequest(http.MethodPost, "https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/token", data)
if err != nil {
  log.Fatal(err)
}
req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
resp, err := client.Do(req)
if err != nil {
  log.Fatal(err)
}
defer resp.Body.Close()
bodyText, err := ioutil.ReadAll(resp.Body)
if err != nil {
  log.Fatal(err)
}
fmt.Printf("%s\n", bodyText)
}
```

</TabItem>
</Tabs>

## Token Configuration

Token configuration, such as expiration and default allowed scopes, can be modified either through the Beyond Identity Admin Console or through API on [application update](https://developer.beyondidentity.com/api/v1#tag/Applications/operation/UpdateApplication).

In the Beyond Identity Admin Console under the "Apps" tab, select the "Beyond Identity Management API" application. Then tap on "EXTERNAL PROTOCOL" and scroll down to the bottom to see "Token Configuration".

![Token Configuration](./screenshots/api-token-configuration.png)

## Revoking Access Tokens

In order to revoke an access token, you must be passed authentication in the form of either Bearer or Basic.

In the case of Basic authentication, the passed authentication must come from the confidential Application that the token has been minted for. The passed access token must be signed by the same client id as the application.

In the case of Bearer authentication, the passed authentication must contain the scope "tokens:delete".

Note that passing an invalid token, or a token which has already been revoked or expired, will produce a success response, pursuant to [RFC7009§2.2](https://www.rfc-editor.org/rfc/rfc7009).

<Tabs groupId="api-token-platform" queryString>
<TabItem value="curl" label="Curl">

```bash title="/revoke"
curl https://auth-$REGION.beyondidentity.com/v1/tenants/$TENANT_ID/realms/$REALM_ID/applications/$MANAGEMENT_APPLICATION_ID/revoke \
-X POST \
-H "Authorization: Bearer $TOKEN" \
-H 'Content-Type: application/json' \
-d '{"token":"$TOKEN_TO_REVOKE"}'
```

</TabItem>
<TabItem value="node" label="Node">

```jsx title="/revoke"
fetch(
  `https://auth-${REGION}.beyondidentity.com/v1/tenants/${TENANT_ID}/realms/${REALM_ID}/applications/${MANAGEMENT_APPLICATION_ID}/revoke`,
  {
    method: 'POST',
    headers: {
      Authorization: 'Bearer TOKEN',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      token: `${TOKEN_TO_REVOKE}`,
    }),
  }
);
```

</TabItem>
<TabItem value="python" label="Python">

```python title="/revoke"
import requests

headers = {
    'Authorization': 'Bearer TOKEN',
    'Content-Type': 'application/json',
}

json_data = {
    'token': 'TOKEN_TO_REVOKE',
}

response = requests.post(
    'https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/revoke',
    headers=headers,
    json=json_data,
)
```

</TabItem>
<TabItem value="go" label="Go">

```go title="/revoke"
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"strings"
)

func main() {
	client := &http.Client{}
	var data = strings.NewReader(`{"token":"TOKEN_TO_REVOKE"}`)
	req, err := http.NewRequest(http.MethodPost, "https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/applications/MANAGEMENT_APPLICATION_ID/revoke", data)
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("Authorization", "Bearer TOKEN")
	req.Header.Set("Content-Type", "application/json")
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	bodyText, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", bodyText)
}
```

</TabItem>
</Tabs>

## Validating Access Tokens

There are two token formats that can be validated: `self-contained` or `referential`. When validating self-contained tokens, consider if you want to revoke your tokens. If revoked tokens are accepted (as in a MVP case), then the token can be validated [offline](api-token#offline-validation) by validating the signature and parsing the claims of the JWT token. In all other cases the token should be [introspected](api-token#introspection-online-validation).

### Introspection (Online Validation)

A successful introspect query will return information about the token. After receiving information about the token, validate that token has the expected scopes.

<Tabs groupId="api-token-platform" queryString>
<TabItem value="curl" label="Curl">

```bash title="/introspect"
curl https://auth-$REGION.beyondidentity.com/v1/tenants/$TENANT_ID/realms/$REALM_ID/introspect \
-X POST \
-u "$MANAGEMENT_API_CLIENT_ID:$MANAGEMENT_API_CLIENT_SECRET" --basic \
-H "Content-Type: application/x-www-form-urlencoded" \
-d '{"token":"$TOKEN_TO_INTROSPECT"}'
```

</TabItem>
<TabItem value="node" label="Node">

```jsx title="/introspect"
fetch(
  `https://auth-${REGION}.beyondidentity.com/v1/tenants/${TENANT_ID}/realms/${REALM_ID}/introspect`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      Authorization:
        'Basic ' +
        Buffer.from(
          `${MANAGEMENT_API_CLIENT_ID}:${MANAGEMENT_API_CLIENT_SECRET}`
        ).toString('base64'),
    },
    body: '{"token":"TOKEN_TO_INTROSPECT"}',
  }
);
```

</TabItem>
<TabItem value="python" label="Python">

```python title="/introspect"
import requests

headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
}

data = '{"token":"TOKEN_TO_INTROSPECT"}'

response = requests.post(
    'https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/introspect',
    headers=headers,
    data=data,
    auth=('MANAGEMENT_API_CLIENT_ID', 'MANAGEMENT_API_CLIENT_SECRET'),
)
```

</TabItem>
<TabItem value="go" label="Go">

```go title="/introspect"
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"strings"
)

func main() {
	client := &http.Client{}
	var data = strings.NewReader(`{"token":"TOKEN_TO_INTROSPECT"}`)
	req, err := http.NewRequest(http.MethodPost, "https://auth-REGION.beyondidentity.com/v1/tenants/TENANT_ID/realms/REALM_ID/introspect", data)
	if err != nil {
		log.Fatal(err)
	}
	req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
	req.SetBasicAuth("MANAGEMENT_API_CLIENT_ID", "MANAGEMENT_API_CLIENT_SECRET")
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
	bodyText, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s\n", bodyText)
}
```

</TabItem>
</Tabs>

### Offline Validation

In order to validate a token offline, the JWT header and claims must be decoded. Decode both and follow the steps below:

1. Parse the [host](https://datatracker.ietf.org/doc/html/rfc3986#section-3.2.2) of the URI in the `jku` header field. If it equals `auth-$REGION.beyondidentity.com`, then proceed. Otherwise, reject the token.
2. Download the public key for token validation from the `jku` in the JWT header. The key can be cached for the number of seconds specified by the [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#response_directives) max-age directive. Check the token signature against the key with matching `kid` downloaded from `jku`.
3. Check that either the application id or resource server identifier is listed in the `aud` of the JWT claims. It is sufficient if at least one of the allowed audiences is in the token `aud` claim.
4. Check timestamps in JWT claims where `nbf` <= current time as unix timestamp in seconds <= `exp`
5. Check that JWT claims target tenant `bi_t` and target realm `bi_r` match the tenant and realm for the given application.
6. Check that JWT claims has the expected scopes.
