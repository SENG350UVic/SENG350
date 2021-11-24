# H1: M5: Reverse Engineering Programming Styles

Name: Jigyasa Chaudhary  
V00899304  
Group: 2

## **Part 2: Additional System Programming Styles - Flask**

### **Programming Style 1: Cookbook Style**

Flask, like Twisted and Node.js, uses [cookbook programming style](https://learning-oreilly-com.ezproxy.library.uvic.ca/library/view/exercises-in-programming/9781482227376/K22536_C004.xhtml#c04_sec002) in its codebase. This allows for the division of a target problem into subproblems in a way that shared data is manipulated strategically to achieve the solved result.  

Flask uses Werkzeug-like test client implemented in the class `FlaskClient` with additional features to defer the cleanup of the request context stack.

Consider the following code block from `testing.py`  
In the class *[FlaskClient](https://github.com/pallets/flask/blob/main/src/flask/testing.py)*, Cookbook programming style is used to create the basic test client for an application where the exit condition is based on the `preserve_context` shared variable.

```python
class FlaskClient(Client):

    application: "Flask"
    preserve_context = False

    def __init__(self, *args: t.Any, **kwargs: t.Any) -> None:
        super().__init__(*args, **kwargs)
        self.environ_base = {
            "REMOTE_ADDR": "127.0.0.1",
            "HTTP_USER_AGENT": f"werkzeug/{werkzeug.__version__}",
        }
    ...

    def __enter__(self) -> "FlaskClient":
        if self.preserve_context:
            raise RuntimeError("Cannot nest client invocations")
        self.preserve_context = True
        return self

    def __exit__(
        self, exc_type: type, exc_value: BaseException, tb: TracebackType
    ) -> None:
        self.preserve_context = False

        while True:
            top = _request_ctx_stack.top

            if top is not None and top.preserved:
                top.pop()
            else:
                break
```

---
The problem that Flask solves with cookbook style is the need to create a test object . In the above code segment, the solution is the test client created for the purpose of testing. There are two shared objects: `application` and `preserve_context`, of which, `preserve_context` is the object that clarifies the Cookbook programming style in Flask. Initially, a remote test client is created using `__init__()`. On analysis, we find the instance change values when the client enters the application and `__enter__()` is called. When the client exits the application, the shared object is set to `False` again. We see a loop which facilitates pops request contexts until the stack is empty or a non-preserved one us found.

The constraints in the Flask program are the same constraints as Twisted and Node.js, that come with the Cookbook programming style. The class methods do not call other methods or jump to another subsection of the code. Each function is a sub-task and, when called repetitively in the correct order of operations, furthers the problem's solution.

This solution add benefits to the Flask system in a neat code and structured test system. As discussed above, we can run into the codebase containing massive files if Cookbooks are not implemented properly. Flask implements excellent documentation which aids in the Cookbook style being implemented strategically so that readability and modifiability of the code are maintained. Compared to Twisted, Flask's cookbook implementation is similar where they both increase the readablity of the code, however FLask takes an additional step with their already well-documented source code.

---

### **Programming Style 2: Abstract Things**

Like Twisted, Flask uses [Abstract Things](https://learning-oreilly-com.ezproxy.library.uvic.ca/library/view/exercises-in-programming/9781482227376/K22536_C013.xhtml#c13_sec002) programming style in its code base.  
This programming style is used to divide a larger problem into *abstract things*, each described by specific operations, to cover the overall problem. Abstract Things style is used with other styles like *interfaces*, *decorators*, etc.

Flask is an micro-web framework i.e. it does not require many particular tools or libraries; thus, it, like Twisted, has many generic system entities. Having abstract solutions and definitions in the Flask Abstraction provides a broad range of functions that allow a higher level of control of the Application in question. The source code contains all the Flask class which is encouraged to be subclassed, located in the [`app.py`](https://github.com/pallets/flask/blob/main/src/flask/app.py) module. The Flask object impleements a WSGI application and acts as the central registry for the view functions, the URL rules, templace configuration and more. Below is a subsection of this module.

```python
class Flask(Scaffold):
    ...
    def __init__(
        self,
        import_name: str,
        static_url_path: t.Optional[str] = None,
        static_folder: t.Optional[t.Union[str, os.PathLike]] = "static",
        static_host: t.Optional[str] = None,
        host_matching: bool = False,
        subdomain_matching: bool = False,
        template_folder: t.Optional[str] = "templates",
        instance_path: t.Optional[str] = None,
        instance_relative_config: bool = False,
        root_path: t.Optional[str] = None,
    ):
        super().__init__(
            import_name=import_name,
            static_folder=static_folder,
            static_url_path=static_url_path,
            template_folder=template_folder,
            root_path=root_path,
        )

        if instance_path is None:
            instance_path = self.auto_find_instance_path()
        elif not os.path.isabs(instance_path):
            raise ValueError(
                "If an instance path is provided it must be absolute."
                " A relative path was given instead."
            )
    ...
    @property
    def preserve_context_on_exception(self) -> bool:
        """Returns the value of the ``PRESERVE_CONTEXT_ON_EXCEPTION``
        configuration value in case it's set, otherwise a sensible default
        is returned.

        .. versionadded:: 0.7
        """
        rv = self.config["PRESERVE_CONTEXT_ON_EXCEPTION"]
        if rv is not None:
            return rv
        return self.debug
    ...
    @property
    def got_first_request(self) -> bool:
        """This attribute is set to ``True`` if the application started
        handling the first request.

        .. versionadded:: 0.8
        """
        return self._got_first_request
    ...
    def create_url_adapter(
        self, request: t.Optional[Request]
    ) -> t.Optional[MapAdapter]:
        if request is not None:
            if not self.subdomain_matching:
                subdomain = self.url_map.default_subdomain or None
            else:
                subdomain = None

            return self.url_map.bind_to_environ(
                request.environ,
                server_name=self.config["SERVER_NAME"],
                subdomain=subdomain,
            )
        if self.config["SERVER_NAME"] is not None:
            return self.url_map.bind(
                self.config["SERVER_NAME"],
                script_name=self.config["APPLICATION_ROOT"],
                url_scheme=self.config["PREFERRED_URL_SCHEME"],
            )

        return None
    ...
    def __call__(self, environ: dict, start_response: t.Callable) -> t.Any:
        """The WSGI server calls the Flask application object as the
        WSGI application. This calls :meth:`wsgi_app`, which can be
        wrapped to apply middleware.
        """
        return self.wsgi_app(environ, start_response)
```

The closest problem here is the need for flexibility while keeping the codebase templated so modification requirements are minimal.  
As we know, Flask’s architectural drivers include scalability to accommodate complex web apps, and flexibility to be able to build them how a developer pleases. Subclassing Flask is a great facilitator which allowss the developer to build on the existing methods and modify based on individual preferences. This reduces the workload of a developer by providing them with the foundations but allowing them to take their own spin for their respective applications.

The constraints in the Flask code are the ones that Abstract Things programming style brings. The code defines numerous methods which the developer has to pick apart when modifying. Only on implementation, the developer solves each operation's functionality. Here is where the programming style helps as each application's solution may differ, forcing the developer to define the functionality for each method per the problem's scope.

The implementation of Abstract Things programming style results in pros and cons for the Flask framework. As mentioned before, using the technique allows a streamlined workflow from the architect to the developer. However, not being as modular as Twisted, Flask developers have to go into the source code and find the method they need to adapt to their application and modify i accordingly. Twisted implements this style well and defines their abstractions well so that this con does not outweigh the benefits of this programming style. Flask on the other hand has an advantage of having a single class which allows for all the functional tools to exist in one place. Finding the source code does not require searching multiple directories. Having central control over the applications, with readily available tools, the Developer is ensured a user-friendly experience with the codebase.

---
