
#Exercise 2 - Request an auth token
Maybe you want to use auth tokens, maybe you don't. Either way they are great for demonstrating a simple REST API POST. Taking a look at 'BIG-IP/02-bigip-get_auth_token.yml' you will notice a few new options:

`      method: POST
      body: { "username":"{{ username }}", "password":"{{ password }}", "loginProviderName":"tmos" }
      body_format: json
`

`method:` we need to say its a HTTP POST

`body:` because a POST requires data (we are 'POSTing' to a resource after all).

`body_format:` because we need to tell it 'what' we are sending. For the BIG-IP API we are posting JSON

Another change you may have noticed, we are being more specific with the object in the JSON response. First we print out the 'entire' response with:

`  - name: Full output
    debug: msg={{ (auth_token.content|from_json) }}
`

Next, we provide an example of just printing the 'token' using:

`  - name: Return the token
    debug: msg={{(auth_token.content|from_json)["token"]["token"]}}
`

Note the reference to `["token"]["token"]`. I fyou look at the structure of the JSON payload from the 'Full output', you will see how the JSON structure reflects this.
