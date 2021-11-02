# WEB - Tokfrans (490pts)

The goal of the task is to find a flag somewhere by following the link to `http://challenges.ctf.crate.foi.se:18263`. We know that it is possible to login on the page by using the credentials:
acc: `zeke`
pass: `irule42`.

## Exploring the website

Upon entering the site we get prompted with a login form.

<img src="https://raw.githubusercontent.com/Liimpo/Writeups/main/FOI%202021/images/tokfrans_1.png" alt="MarineGEO circle logo" style="height: 500px; width:400px;"/>

When logged in the only information displayed to us is: 

```
Logged in as zeke
No flags to show. 
```

By opening the cookies page it is possible to notice that the site is saving information about the user. By looking at the structure of the value the information is most probably a JWT token. The token is the following:

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiemVrZSIsInNjb3BlcyI6WyJ1c2VyIl0sInNwZWVkIjo5MDAwLCJncm95IjoiYjliSUFXMUZseUtjUUd5ZWZ1REwzRjRvcjF6aDBzUlBGOTNSN24wOSIsImV4cCI6MTYzNTg2NjY1MX0.A1bpRkFhAjpX3W7lR9uXbM7KuF3WA94t5ab6PQltpYI`

By opening the JWT in a tool such as https://jwt.io/ we can deduce its header and payload. This gives us the information that the token is encrypted using HS256 and the __interesting__ information in the payload is `"user": "zeke"`and `"scopes": ["user"]`

If we are able to spoof the token as another user there is a possibility that there is a flag awaiting 🥳🥳

The first step is to find the key used in the hashing algorithm, if a weak key was used we should be able to create a spoofed token!



## Cracking the JWT token

To be able to crack the token I had to download a tool found at https://github.com/x1sec/gojwtcrack , which is a dictionary attack tool against HS256. By using this with a wordlist such as rockyou if the passphrase is weak I should be able to get it.

By executing the command in the picture I got the following result (e is the file where I put my token):

<img src="https://raw.githubusercontent.com/Liimpo/Writeups/main/FOI%202021/images/tokfrans_2.png" alt="MarineGEO circle logo" style="height: 121px; width:628px;"/>

**BINGO!!** A weak passphrase was used and it is : `queenoftherabbitkillers`. 



## Forging the new token

With the secret key at hand, the next step would be to access another user with other privileges in the web page. Why not try the most simple one which is <u>__admin__</u> of course. (:

By going back to the JWT tool I change the values `"user": "zeke"`to `"user": "admin"`and the `"scopes": ["user"] ` to `"scopes": ["admin"]`. I also enter the passphrase that was extracted in the last section in the key field. This might be able to give me access to the admin page! The new token is now instead the string:

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJzY29wZXMiOlsiYWRtaW4iXSwic3BlZWQiOjkwMDAsImdyb3kiOiJiOWJJQVcxRmx5S2NRR3llZnVETDNGNG9yMXpoMHNSUEY5M1I3bjA5IiwiZXhwIjoxNjM1ODY2NjUxfQ.o3eXjZOi_gjHE4XwjADgOvZxklMNx1FfveLpsH37Dqs`

Let's replace the old token with our new one and refresh the page.

**BOOM**, It worked. We are now logged in as admin and the flag are displayed on the website: `cratectf{th0se_t0ken_s3cre7ts_ar3_pr3c1ou5}        `

