Authentication
====================
Causemo utilizes [oauth2](https://tools.ietf.org/html/rfc6749) to ensure all requests to the API servers are valid. Requests to the API servers need to contain a valid oauth token, otherwise it will be rejected. 

## Grant Types

### Token
  This grant type is used for clients that are not 'privileged' because they are not able to keep a secret or are third party platforms using our system as a login provider. This grant type will pop up a login page on causemo.com and do all the authentication on causemo servers, then redirect back to the client requesting token after user logs in. For example, you would use this is if you are a static web page. Since the end users can see and inspect the javascript code, a static web page cannot keep a secret. Thus it must use a token grant type.
  - **Endpoint**: [GET] ```/auth/authorize```
    - Required Query Params:
	      - ```grant_type```: 'token'
	      - ```client```: A client public key
	      - ```redirect_uri```: The client redirect URI. This must match the one registered with. 
		- Optional Query Params:
			- ```state```: Used as a passthrough to the ```redirect_uri```. This allows the caller to pass any state information back to their app.
 - Response: Redirected back to ```redirect_uri``` containing token information (if successful login) or error (if failure). 
		- Success Query Params: (PLUS any passthrough parameters)
			- ```token```: The oauth token granted
			- ```token_type```: ```bearer``` This is needed to be passed in with token 
			- ```expires_in```: milliseconds until when the token expires. 
		- Failure Query Params:
			- `error`: The reason for failure (ie: access_denied)
  - **Try it**:
	  - http://dev-api.causemo.com/auth/authorize?grant_type=token&client=causeMoWww1&redirect_uri=http://causemo.com
	  - The causemo login page will appear enter a user name and password
	  - After login it will redirect you to: http://www.causemo.com/?token=dbc63cd9ace3f05588705350b28e8a7ede7f5c8ac51b4a8094fe19c441702a20235399fa45464de79a1a4eaa66fb97d61291fe7ced1667&token_type=bearer&expires_in=172799990

### Password
  This grant is used by 'privileged' clients because they are able to keep a secret and not expose it to users. With this grant type, clients are able to login using their own login page and send a request to causemo to validate the user credentials on their behalf. Causemo will then receive the request and respond back accordingly with a JSON object. An example on when you would use this is from server-to-server communication. For example, a website with a server backend can send the login request to Causemo from its server. This client is considered to be able to keep a secret because it can keep its secret key on its server and not expose it to end-users.
  - **Endpoint**: [POST] `/auth/token`
    - Required Query Params:
      - `grant_type`: `password`
      - `username`: Causemo user's username
      - `password`: Causemo user's password
    - Required Header:
      - `Authorization: 'Basic xxxxxxxxxx'`, where 'xxxxxxxxxx' is the base64 encoded client public/secret key (Basic auth).
    - Response: A JSON object with the following properties:
      - Success:
        - `token`: The oauth token granted
        - `token_type`: `bearer`
        - `expires_in`: milliseconds until when the token expires. 
      - Failure:
        - `status`: status code
        - `message`: A friendly user message
  - **Try it**:
    - Base64 encode your client public key and secret key as such `PUBLIC_KEY:SECRET_KEY`. 
```
curl -X POST -H "api-version: 1" -H "Authorization: Basic <VALUE_FROM_ABOVE>" -i --data "" "http://dev-api.causemo.com/auth/token?grant_type=password&username=<USER>&password=<PASSWORD>"
```
- Replace `<VALUE_FROM_ABOVE>` with base64 value
- Replace `<USER>` with user name
- Replace `<PASSWORD>` with user password
- Repsonse will look as such: 
```javascript
{"token":"fbd12f796b9997a02638eeca509e642868860dc9fad9ee9e3e5420d2b30be2b67f7a306562e92ca5766bb02acfe964659001f949d0f276","expiresIn":172799993,"tokenType":"bearer"}
```
