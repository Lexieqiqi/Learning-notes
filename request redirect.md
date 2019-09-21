You can choose to have something else handle the response for your request
You can redirect the request to a completely different URL, or you can **dispatch** the request to some other component in your web app(typically a JSP)

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

- response.sendRedirect("http://www.xxx.com")
- **relative**
- response.sendRedirect("foo/stuff.html") -> "http://..../foo/stuff.html"
- **complete**
- response.sendRedirect("/foo/stuff.html") -> ".../...com/foo/stuff.html"


**A request dispatch does the work on the server side**
That's the big difference between a redirect and a request dispatch
- redirect, makes the **client** do the work
- request dispatch, makes something else on the **server** do the work.

1. User types a servlet's URL into the browser bar...
2. The request goes to the server/Container
3. The servlet decides that the request should go to another part of the web app(in this case, a JSP)
4. The servlet calls 
 RequestDispatcher view = request.getRequestDispatcher("result.jsp");
 view.forward(request,response);
5. The browser gets the response in the usual way, and renders it for the user.Since the browser location bar didn't change, the user does not know that the JSP generated the response.

**Init Parameters to the rescue**
the servlets can have initialization parameters as well.
In the DD(web.xml) file:
    <servlet>
        <init-param>
            <param-name>adminEmail</param-name>
            <param-value>likewecare@wickedlysmart.com</param-value>
        </init-param>
    </servlet>
In the servlet code:
    out.println(getServletConfig().getInitParameter("adminEmail"));
getServletConfig() -> Every servlet inherits his method
getInitParameter() -> above's method

**You can't use servlet init parameters until the servlet is initialized**
When the container initializes a servlet, it makes a unique ServletConfig for the servlet.
The Container "reads" the servlet init parameter from the DD and gives them to the ServletConfig, the passes the ServletConfig to the servlet's init() method.

**The servlet init parameters are read only Once- when the Container initializes the servlet**
When a container makes a servlet, it reads the DD and creats the name/value pairs for the ServletConfig. The Container never reads the init parameter aggain? Once the parameters are in the ServletConfig, they won't be read again until/unless you redeploy the servlet.

1. Container reads the Deployment Descriptor for the servlet, including the servlet init parameters. 
2. Container creates a new ServletConfig instance for this servlet.
3. Container creates a name/value pair of Strings for each servlet init parameter. Assume we have only one.
4. Container gives the ServletConfig references to the name/value init parameters.
5. Container creates a new instance of the servlet class.
6. Container calls the servlet's init() method, passing in the reference to the ServletConfig.

**How can a JSP get servlet init parameters?**

A servletConfig is for servlet configuration. So if you want other parts of your application to use the same info you put in the servlet's init parameters in the DD, you need something more.

**Context init parameters to the rescue**
Context parameters are available to the **entire webapp**, not just a single servlet. So that means any servlet and JSP in the app automatically has access to the context init parameters, so we don't have to worry about configuring the DD for every servlet, and when the value changes, you only have to change it once place.
In the DD(web.xml) file:
    <context-param>
        <param-name>adminEmail</param-name>
        <param-value>clientheeaderror@wickedlysmart.com</param-value>
    </context-param>
**Important** - The <context-param> is for the whole app, so it's not nested inside an individual <servlet> element!! Put <context-param> inside the <web-app> but outside any <servlet> declaration!!

In the servlet code:
    out.println(getServletContext().getInitParameter("adminEmail"));
Every servlet inherits a getServletContext() method.
It returns a servletContext object. and one of this object's methods is get InitParameter()

== ServletContext context = getServletContext();
== out.println(context.getInitParameter("adminEmail"));

**ServletConfig is one per servlet**
**ServletContext is one per webapp**

***

There's only one ServletContext for an entire web app, and all the parts of the web app share it. But each servlet in the app has its own ServletConfig. Tehe container makes a ServletContext when a web app is deployed, and makes the context available to each Servlet and JSP(which becomes a servlet) in the web app.


**javax.servelet.ServletContextListener**

We need a separate object that can:
- Get notified when the context is initialized(app is being deployed)
    + Get the context init parameter from the ServletContext
    + Use the init parameter lookup name to make a database connection
    + Store the database connection as an attribute, so that al parts of the web app can access it.
- Get notified when the context is destroyed (the app is undeployed or goes down)
    + Close the database connection

A servletContextListener class:
    import javax.servlet.*;

    public class MyServletContextListener implements ServletContextListener {
        public void ContextInitialized(ServletContextEvent event){
            //code to initalize the database connection
            //and store it as a context attribute
        }
        public void contextDestroyed(ServletContextEvent event){
            // code to close the database connection
        }
    }

1. Create a listener class
 To listen from ServletContext events, write a listener class that implements ServletContextListner.
2. Put the class in the WEB-INF/classes
3. Put a <listener> element in the web.xml Deployment Descriptor
    <listener>
        <listener-class>
            com.example.MyServletContextListener
        </listener-class>
    </listener>
