# apogee-http

The HTTP layer of [Apogee](https://github.com/venomous-maker): `Request`,
`Response`, HTTP errors, the middleware contract and pipeline, API
resources with pagination, signed cookies, uploads, and a JSON writer. It
sits on the standard library's HTTP server (`std/net`), which keeps the
transport: event loops, keep-alive, TLS, compression.

## Request

```grav
req.input("user.name")              // JSON body, form, or query string; dotted paths into JSON
req.text("q")  req.integer("page", 1)  req.number("price")  req.boolean("remember")
req.date("from")  req.list("tags")  req.all()  req.only(["a", "b"])  req.except(["password"])
req.has("x")  req.filled("x")  req.missing("x")  req.merge(values)
req.param("id")  req.paramInt("id")                 // route values
req.file("avatar").store("storage/avatars")         // uploads
req.method()            // a POST form's _method (or X-HTTP-Method-Override) gives PUT/PATCH/DELETE
req.pathIs(["admin/*"])  req.routeIs(["admin.*"])  req.segment(2)  req.url()  req.fullUrl()
req.expectsJson()  req.wantsJson()  req.ajax()  req.bearerToken()  req.ip()  req.cookie("x")
req.set("tenant", t)  req.get("tenant")             // what middleware leaves for later
req.make::<Mailer>()                                // from the request's container scope (made on first use)
```

More, in Laravel's spelling:

```grav
req.string("q")  req.float("price")  req.is(["admin/*"])          // (text, number, pathIs)
req.input("items.*.sku")            // wildcards: every match, as an Array<any>
req.hasAll([...])  req.anyFilled([...])  req.isNotFilled("x")  req.keys()  req.map("address")
req.whenHas("coupon", fn(v: any) { ... }).whenFilled("q", fn(v: any) { ... }, fn() { ... }).whenMissing("x", fn() { ... })
req.only(["user.name"])             // keeps nesting: {"user": {"name": ...}}
req.mergeIfMissing(values)  req.replace(values)  req.post("field")  req.queryAll()  req.date("when", "%d/%m/%Y")
req.old("email")  req.flash()  req.flashOnly([...])  req.flashExcept(["password"])   // with a session
req.validate(rules)  req.validated()                                                // with apogee-validation
req.route("post")  req.userAs::<User>()  req.allFiles()  req.hasCookie("x")  req.content()
req.prefers(["json", "html"])  req.accepts("text/csv")  req.acceptsJson()  req.format()  req.pjax()  req.prefetch()
req.ip()  req.ips()  req.scheme()  req.httpHost()  req.port()                       // through trusted proxies
req.fullUrlWithQuery({"page": 2})  req.fullUrlWithoutQuery(["page"])  req.fingerprint()
```

`Request.trustedProxies` (addresses, `"10.0.*"`, or `"*"`) decides whose
X-Forwarded-For/-Proto/-Host to believe. `accepts`, `prefers` and `format`
read the Accept header by quality.

Every parse (query, form, JSON body, cookies, files) happens once per
request, on first use. A JSON body that does not parse is a 400.

## Response

```grav
Response.json(anything, 201)     // plain values, maps, arrays, models, objects
Response.html(html)  Response.text("ok")  Response.noContent()  Response.created(user, "/users/7")
Response.redirect("/login")  Response.back(req)  Response.file(path)  Response.download(path, "report.pdf")
Response.stream("text/event-stream", fn(w: ResponseWriter) { ... })
res.header("X-Frame-Options", "DENY").withCookie(cookie).withStatus(202).noCache()
```

## Errors

`throw new HttpException(403)`, `abort(404)`, `abortIf(cond, 403)`,
`abortUnless(cond, 401, "Sign in first")`. An `HttpException` renders
itself: `{"message": ...}` for a client that expects JSON, a small HTML
page otherwise, with any headers it carries (`withHeader("Retry-After",
"30")`). Exceptions that implement `Renderable` choose their own response.

## Middleware

```grav
class RequireJson implements Middleware {
    fn handle(req: Request, next: fn(Request) -> Response) -> Response {
        if (!req.expectsJson()) { abort(406) }
        return next(req).header("Vary", "Accept")
    }
}
let run = Pipeline.compose(layers, handler)      // joined once; each request is plain calls
```

`FnMiddleware` makes a function into one; `TerminableMiddleware` also runs
after the response is sent.

## API resources

```grav
class UserResource extends JsonResource {
    fn toArray(req: Request) -> HashMap<string, any> {
        let u = self.resource as User
        return self.fields()
            .add("id", u.id).add("name", u.name)
            .when(req.user != null, "email", u.email)
            .whenNotNull("bio", u.bio)
            .build()
    }
}
new UserResource(user).toResponse(req)                                    // {"data": {...}}
ResourceCollection.paged(page, fn(u: any) -> JsonResource { return new UserResource(u) }).toResponse(req)
// {"data": [...], "meta": {"current_page", "per_page", "total", "last_page", "from", "to"},
//  "links": {"first", "last", "prev", "next"}}
```

`Paginator` holds a page and its place in the whole (`Paginator.slice` pages
an in-memory list); `with(req)` adds top-level members; `JsonResource.wrapKey`
changes or removes the `"data"` wrapper.

## Cookies

`CookieJar(appKey)` makes signed cookies (HMAC-SHA256 over name and value,
compared in constant time): the client can read them but not change them
unnoticed. `jar.make(name, value, minutes)` is HttpOnly and SameSite=Lax by
default; `jar.read(req, name)` gives `""` for a missing or altered one.

## JSON

`JsonWriter.encode(v)` writes any plain value, map, array, `JsonValue`,
`DateTime`, `Decimal`, `Arrayable`, object with a `toMap()`, or reflected
object (its public fields, minus `@Hidden` ones) straight into one buffer,
with no intermediate tree. `Plain.parse` reads JSON back into plain values;
`Plain.dig(v, "a.b.0")` walks them.

## Tests

```bash
grav test tests/http.grav
```

MIT licensed.
