# The Shark 


Category: `network`

Date: `May 31,2023`


We've been handed a network dump file named `wooow.pcapng` and tasked with uncovering authentication information from HTTP requests transmitted through this network. 

In addition to the file, we've been provided with a URL hosting a simple login form that accepts a username and password.

URL: `http://ctf.uut.ac.ir:8080`

![Screenshot from 2023-05-31 22-20-10](https://github.com/behnambm/writeups/assets/26994700/872c6adc-a137-4f16-893d-0194abe58e58)

Upon examining the JavaScript code, we see that the submitted form takes the password and sends its `MD5` hash to the server.

```js
<body>
<div class="login-container">
    <h2>Login</h2>
    <form action="/login" method="post" onsubmit="convertPasswordToMd5(event)">
        
        
        <div class="form-group">
            <label for="username">Username:</label>
            <input type="text" id="username" name="username" class="form-control" />
        </div>
        <div class="form-group">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" class="form-control" />
        </div>
        <button type="submit" class="btn btn-primary">Login</button>
    </form>
</div>
</body>

<script>
    function convertPasswordToMd5(event) {
        event.preventDefault(); // Prevent form submission

        var passwordInput = document.getElementById('password');
        var passwordValue = passwordInput.value;
        var md5Password = md5(passwordValue); // Convert password to MD5

        passwordInput.value = md5Password;

        event.target.submit(); // Submit the form with the converted password
    }
</script>
```

---


My initial thought is to use `Wireshark` to open the `pcapng` file and examine its contents.

![first-wireshark](https://github.com/behnambm/writeups/assets/26994700/b6fa926b-cfd9-4a39-8731-a46e39c43008)

This reveals a variety of traffic, including `POST` and `GET` requests. To locate authentication credentials, we must delve into the requests and search for any useful payloads being sent to the server.

Let's proceed accordingly. Right-clicking on a `POST` HTTP request, we select "follow" and then choose "HTTP Stream" from the submenu.

A new window opens.

![Screenshot from 2023-05-31 22-02-40](https://github.com/behnambm/writeups/assets/26994700/3941b8a6-1f85-496a-b6ee-64989f196989)

As highlighted in the screenshot, I've identified the most informative part of the request:

`username=user1&password=47476ffc934306fc1983acb832cd207d`

Now we have something!

However, upon closer inspection, we notice other HTTP requests as well, both `GET` and `POST`. Should we open each request individually?

Fortunately, there's a better way: a tool called `tcpflow` that does the same but more elegantly.

`sudo tcpflow -r wooow.pcapng`

![Screenshot from 2023-05-31 22-15-21](https://github.com/behnambm/writeups/assets/26994700/df5e34b8-3975-4ee3-a6b3-39c39e4f43f3)


`tcpflow` reconstructs and stores all these files separately, making network analysis and debugging so much easier.

Since we know that the client-server communication took place on port `8080`, we don't need to sift through all these files.

`cat *.08080 | grep password`

This command will only display the contents of files ending in `.08080`.

![Screenshot from 2023-05-31 22-29-49](https://github.com/behnambm/writeups/assets/26994700/b902934b-1726-412d-a504-346b46998981)

Recalling that the login form converts the password to `MD5`, we have two options:

- Test the hash against common words using online websites to find the actual password.
- Bypass the login form and directly send the `POST` request to the login endpoint.


The second option is preferable, so let's proceed. 

We only need to send a maximum of `4` requests to the server.

![Screenshot from 2023-05-31 22-33-16](https://github.com/behnambm/writeups/assets/26994700/1cf6912b-4579-4dba-b6e5-4470f17696dd)

This is how we make requests to the server using a tool called `Postman`.

![Screenshot from 2023-05-31 22-34-38](https://github.com/behnambm/writeups/assets/26994700/bbcf5409-0d5b-40bf-9ff9-67e11be05d0f)

Now we know that the username and password are correct, but still, there's no flag.


By taking a look at this we can guess that maybe this website uses `SESSION ID` or some kind of `Token` to maintain communication state. 

> Pay attention to the `Admin` button.

![Screenshot from 2023-05-31 22-37-19](https://github.com/behnambm/writeups/assets/26994700/08f60a9c-2b4a-4168-ae95-cd4e10243f04)

Here, we notice that the server sends a `JSESSIONID` back to the client.

As we can't simply click on the `Admin` button, we need to employ a browser. Let's copy the `JSESSIONID` and navigate to `http://ctf.uut.ac.ir:8080`, adding it to the website's `Cookie` storage.

![Screenshot from 2023-05-31 22-48-19](https://github.com/behnambm/writeups/assets/26994700/d98f27da-b561-4cb0-83ca-ae516a60ddd5)

Now, refresh the page and click on the `Admin` button.

![Screenshot from 2023-05-31 22-49-21](https://github.com/behnambm/writeups/assets/26994700/a7ab0331-8d1d-4812-84c3-2af8dbbdad4c)

There you gooooooo! We've got the flag! 🎉