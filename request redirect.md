You can choose to have something else handle the response for your request
You can redirect the request to a completely different URL, or you can **dispatch** the request 

to some other component in your web app(typically a JSP)

***

1. Client types a URL into the browser bar
2. The request goes to the server/Container
3. The servlet decides that the request should go to completely different URL
4. The servlet calls sendRedirect(aString) on the response and that's it
5. The HTTP response has a status code "301" and a "Location" header with a URL as the value
6. The broswer gets the response , sees the "301" status code, and looks for a "Location" header
7. The browser makes a new request using the URL that was the value of the "Location" header in the previous response. The user might notice that the URL in the broswer bar changed
8. There's nothing unique about the request, even though it happened to be to be triggered by a redirect
9. The server gets the thing at the requested URL. Nothing special here
10. The HTTP response is just like any other response... except it isn't coming from the location the client typed in.
11. The browser renders the new page. The user is surprised.