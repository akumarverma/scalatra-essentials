### Chapter 1 & 2
* Scalatra is lightweight web framework, sometimes called microframework
* it mainly route the HTTP request to actions. The actual logic is being carried out by action code which depends on multiple libraries
* 
* scalatra application run on any Java servlet container such as tomcat, jetty and JBoss
* it is easy to install, lightweight and fast
* import org.scalatra._
* Scalatra route consists of 
    1. HTTP verb
    2. a route matcher
    3. route action
* each controller class should extend trait ScalatraServlet which allow accessing class through HTTP
#### Installing Scalatra
* a new project can be created using giter8 or g8 for short
* it fetches the template from git, present few options to users.
* it apply user input to fetched template and save the folder to user local system
```aidl
g8 scalatra/scalatra-sbt
or
sbt new scala/scala-seed.g8
```
* Once the template is saved on local system, we can move to top level directory and run the sbt command
```aidl
./sbt
```
* sbt command will download all required dependencies mentioned in project/build.scala file
* a scalatra application can be started with
```aidl
jetty:start
~jetty:start -> it will restart the application for each changes
jetty:stop -> stop the server
exit -> return from sbt console
```
* The g8 tool provides a set of functionalities such as scalatraCore, specs2 testing library, logger and jetty web server
* to add additionaly librearies, we have to add new libraries in libraryDependencies section of project/build.scala file
* the scalatra route matcher can take path parameter or query parameters
* the parameters in scalatra is extracted using param("paramName") method
* sample routing function
```aidl
  get("/pages/:slug") {
  contentType = "text/html"
  PageDao.pages find (_.slug == params("slug")) match {
    case Some(page) => page.title
    case None => halt(404, "not found")
     }
}
```
#### Writing tests
* scalatra support specs2, scalatest testing framework.
* the default is spec2s
* normaally g8 template create 2 folders under src(main and test)
* test cases are residing uder test folder
* the main point of a test specificaion are:
    * import org.scalatra.test.specs2._
    * the test class extends ScalatraSpec traits(class PagesControllerSpec extends ScalatraSpec)
    * mount the controller(addServlet(classOf[PagesController], "/*"))
    * checking status( status must_== 200)
    * checking body (body must contain("Bacon"))
* the test specifications can be run using ~test command in sbt
```aidl
~test
```
#### deployments
* run package command under sbt to generate the war file 
* the war file will be placed under target folder
* once the war file is generated, it can be copied to any servlet container such as tomcat, JBoss etc
```aidl
package
```
### Chapter 3(Routing)
* routers are glue that binds incoming HTTP request to block of code implementing application logic
* A route consists of three components: HTTP Method, route matcher and action
* scalatra support only 8 HTTP methods
* CRUD Mapping
* C(Create) === POST,PUT
* R(retrieve) === GET
* U(Update) === PUT,PATCH
* D(Delete) === DELETE
* HEAD method is identical to GET, except it only checks the existance of the resource but do not return any response
* OPTIONS method sets the header and return an allowed methods
```aidl
 options("/artists/Frank_Zappa") {
    response.setHeader("Allow", "GET, HEAD")
}
```
* forbidding all requests
```aidl
options{
    forbidden()
```
* TRACE and CONNECT HTTP method is not implemented in Scalatra
#### Overriding the methods
* some web browsers support only a subset of HTTP method(GET,POST)
* the actual method name is passed either in body[param("body")] or available in request header as 
X-HTTP-Override-headers
* Scalatra provides a trait MethodOverrideSupport that enables servlet to support method override
#### Route Matchers
* Scalatra supports three kind of route matchers
    * path expression
    * regular expressions
    * boolean expression
* path expression starts with "/" and refers to mount point of your application
* path expression can be static path expression or take path parameters
* path parameters can't be empty and it matches to next special character /,? or #
* a path parameter ending with ? make the path parameter as optional
```aidl
get("/artists/?")
```
* if the path parameter can special character /, we can use splat(/*) which match everything
```aidl
get("/downloads/*")
```
* the * above matches everything after downloads which may contain /
##### Regular expressions
```aidl
get("""/best-of/(\d{4})""".r ) -> matches a 4 digit number
get("""/best-of/(\d{3}0)s""".r ) -> matches a 3 digit number plus the last digit as 0 followed by s
```
##### Boolean expression
* a boolean expression acts as a guard to path or regular expression
* based on boolean expression, the appropriate router will be called
```aidl
get("/listen/*".r, isMobile(request)))
or
get("/listen/*".r, isDesktop(request)))
```
#### Chapter 4(Working with user input)
* Once a matching route is found, following things happens:
    1. parameters are fetched from URI and placed in Params map
    2. before method is invoked for preprocessing
    3. request action is executed
    4. after method is invoked for post processing
    5. response is send back to requesting agent
* if no matching is found, NotFound method is invoked which can be customized
* the default status for notFound is 404
#### kind of HTTP parameters
* path parameters
* query parameters
* form parameters or POST parameters
* normally if parameters are used to identify a resource, it should be in path parameters
* if parameters are used to define SORT, or LIMIT, it should be query parameters
#### params vs multiParams
* a parameter can appear multiple times in URI, but param will only fetch first one
* we can use multiParam, which will return a SEQ for all parameters
#### delaing with non-existing unexpected request
* using Option
```aidl
val searchQuery = params("search_query") -> will throw error if search_query is not provided in request

val searchQueryOption = params.get("search_query") -> returns a Option
searchQueryOption match {
case some(x) if x.trim.length > 0 => ...
case None => ...
```   
* using default value
```aidl
val searchQueryOption = params.getOrElse("search_query","") -> returns a Option 2nd aggument in case of null

```
* using halt
```aidl
get("/results") {
          val search_query =
            params.getOrElse("search_query",
            halt(200, "Please provide a search query"))
        "You searched for '" + search_query + "'"
        }
```
#### Typed parameters
* if the incoming parameters don't match with type, it return None
* GetAs support all premitive type such as Int,Long, Short,Byte, Float,Double, Boolean and Java.util.data

```aidl
params.getAs[Double]("price") //=> Option(9.99)

val name = params.getAs[Name]("name").getOrElse(
      halt(BadRequest("Please provide a name")))
```
#### Filters
* filters are similar as servlet folters
* before filter runs before each request
* after filter run after each request
```aidl
fun before {
contentType='html/text'
dbConnect

after {
dbDisconnect()
}
```
* it is also possible to define filters for a given route
```aidl
before("/hackers") {
   DataBase.connect
}
```
* we can also run filters on HTTP methods
```aidl
 before("/hackers", request.requestMethod == Post) {
          DataBase.connect;
}
```
#### Getting request Header
```aidl
request.getHeader("Accepts").split(",").contains("html/text")
```
#### Cookies
```aidl
cookies.get("counter")

cookies.update("counter",50.toString)
```
#### Halt
* if you want to stop execution in either before or action code, we can use halt
* we can supply status, reason,headers and even response body
```aidl
halt(status = 403,
reason = "Forbidden",
headers = Map("X-Your-Mother-Was-A" -> "hamster",
          "X-And-Your-Father-Smelt-Of" -> "Elderberries"),
     body = <h1>Go away or I shall taunt you a second time!</h1>)
}
```
#### ActionResults
* actionResult is another way of generating response
* most of the HTTP response code is having a mapped ActionResults
* OK(200), CREATED(201),BadRequest(400), NotFound(404), Forbidden(403) etc are different ActionResults

### chapter 5(Handling JSON)
* Scaltra JSON handling perform two things in request processing
    * it parses incoming josn messages to json value
    * it write the json response as text to response
*  JSOn support libraries
```aidl
libraryDependencies ++= Seq(
  "org.scalatra" %% "scalatra-json" % ScalatraVersion,
  "org.json4s"   %% "json4s-jackson" % "3.3.0")
```
* the controller requiring json support should mixin with JacksonJsonSupport
* JacksonJsonSupport is responsible to parse incoming JSON value as JValue and returing JValue as JSOn text in reponse
* we should define a implicit val for defaultformats
```aidl
implicit val jsonFormats = DefaultFormats
```

### Chapter 8(Testing)
* Scatatra comes with a test DSL
* Scalatra Support both scalatest and specs2
* it can be used for unit as well as integration testing
* Specs2 is default testing framework for Scalatra, the module dependency is added into project/built.scala file
```
libraryDependencies ++= Seq(
  "org.scalatra" %% "scalatra-specs2" % ScalatraVersion % "test"
Defines the scalatra-specs2 dependency in test scope.
 )`
```
* a typical test specification
* the test class extends scalatraSpec trait
* test framework does not support path parameters, boolean guards or regular expression
* the response status can be tested as status must_==200
* the response header can be asserted as header("content-type")
* the response body can be assrted as body must contain(...)
```aidl
import org.scalatra.test.specs2._
class FoodServletSpec extends ScalatraSpec{
    ...
    addServlet(classOf[FoodServlet], "/*")
    ...
}
```
#### testing as Jvalue
```aidl
val json = JsonMethods.parse(body)
    json \ "name" must_== JString("potatos")
```
#### Sequntial
* by default specs2 run all test in parallel
* sequential forces test cases to run the sequential but all other specs will still run in parallel

### Chapter 9(Configuration,built and deployment)
#### Environment
* the current running application environment is accessed using environment method of ScatraBase
* the default environment is DEVELOPMENT, it can also be accessed using isDevelopment
```aidl
get("/me") {
          environment match {
            case "DEVELOPMENT" =>
              println("oh hai")
            case "PRODUCTION" =>
case _ => }
}
```
* the environment is set using org.scalatra.environment key either using system parameter or in web.xml file
```aidl
web.xml file

<context-param>
    <param-name>org.scalatra.environment</param-name>
    <param-value>production</param-value>
  </context-param>
```
* using system parameters
```aidl
-Dorg.scalatra.environment=DEVELOPMENT
```
#### Application configuration
* the default application file is stored in src/main/resources/application.conf
* The application configuration is loaded using configFactory load method
```aidl
val cfg = ConfigFactory.load
val webBase = cfg.getString("webBase")
println(webBase)
```
* it is common practice to create an scala object(appConfig) to read values from configuration file and pass it to application servlet
* the config is loaded in lifecycle, ScalatraBootstrap
```aidl
class Chapter09(appConfig: AppConfig) extends ScalatraServlet {
  get("/shorten-url") {
    val token = UrlShortener.nextFreeToken
    f"${appConfig.webBase}/$token"
  }
}

class ScalatraBootstrap extends LifeCycle {
  override def init(context: ServletContext) {
     val conf = AppConfig.load
     sys.props(org.scalatra.EnvironmentKey) = conf.env.asString
     val app = new Chapter09(conf)
     context.mount(app, "/*")
  } 
}
```
* The application configuration can also be provided while starting the JVM
```aidl
-Dconfile.file=opt/user/appl/application.conf
```
#### Building Sacalatra Application
* sbt is built tool for scala
* src/main/scala contains source code
* src/main/resources contains application configuration
* src/main/webapp contains the web assets which will be made available public
* src/test contains test specification
* target folder contains code generates during build
* the application built definition is defined as build.sbt or project/build.scala file
```aidl
libraryDependencies +=
   "org.scalatra" %% "scalatra-specs2" % "2.4.0" % "test"
```
* The library dependencies has parts like group id, artifect id,version and scope
* reusable functionality is contained in plugins. 
* the file project/plugins.sbt contains a list of plugins that should be added to project
#### deploying a WAR archive
* a war file is typical way to deploy the web application to a servlet container
* a WAR file can be placed on webapp folder on tomcat server
* by default, the servlet uses the WAR file name as context path to the application
* a WAR file is self-contained version of web application which contains resources requires to run a application
* a servlet container receives a WAR file and host a web application
* a AWR file is generated using package command
#### deploying Scalatra application to a Docker container
* docker allow us to bundle a service as an image
* a container is a running instance of docker image isolated from the host environment it is running
* refer to book for further settings

### Chapter 12(Asynchronous programming)
* Servlet maintain a thread pool to handle incoming request. it pulls a thread from the pool to serve the request.
* the thread to alive and tied to the request unless request is fulfilled.
* default thread poll size for tomcat apache is 200
* if an request takes 1 second time, it means .5% of servlet capacity is blocked for 1 second
* the process will be much efficient, it the actual job is being carried out by a different thread
* in this approach, the servlet thread will able to handle more request
* we can add this asynchronous programming in scalatra using Future and Akka actor model
#### Futures
* futures let us program with the result which we don't have yet
* it allow us to transform the result when available
* we can invoke callback onSucuss,onComplete,OnFailure
* if a scalatra action uses Future, the request will be suspended and thread will be released
* when the future completes, it wake up the thread and send the response
* Future run in its own thread, it is seperate from servlet thread pool
```aidl
class CrawlController extends ScalatraServlet with FutureSupport {
  protected implicit def executor = ExecutionContext.global
  get("/") {
    contentType = "text/html"
    Future {...}
}
}
```
* we should not be modifying request in Future since request is part of servlet thread pool
* if we want to use request inside Future, we should be using asyncResult instead of Future
```aidl
new AsyncResult { ...}


```
* asyncResult expects that the called function should return Future
#### Using Akka Actors
* Akka provide a construct called actor which is a small data structure responsible for sending and receiving messages
* actor is lightweight so millions of actors might be running at the same time
* like Futures, it also run on its own thread
* unlike Futures, Actor can be running on distributed environment
* adding akka dependencies
```aidl
"com.typesafe.akka" %% "akka-actor" % "2.3.4",
```
* add actor system and actors in scalatra boot strap 
```aidl
class ScalatraBootstrap extends LifeCycle {
  override def init(context: ServletContext) {
    val system = ActorSystem()
    val grabActor = system.actorOf(Props[GrabActor])
    context.mount(new AkkaCrawler(system, grabActor), "/akka/*")
  }
```
* pass actor system and actors as parameters in controller class
* set default timeout for actor class
```aidl
implicit val defaultTimeout = new Timeout(2 seconds)
```
* send message to actor in route actions
```aidl
  get("/") {
    contentType = "text/html"
    new AsyncResult {
      val is = grabActor ? new URL(params("url"))
    }
}
```
* operator ? is ask pattern in akka
* 



### Chapter 13(Swagger)
#### REST API support
* JacksonJsonSupport is added to controller to support JSON parsing
* add contextType as JSON in before filter
* importing defaultFormats allows conversion between case class and JSON
* 
```aidl
class ApiController extends ScalatraServlet with JacksonJsonSupport

protected implicit lazy val jsonFormats: Formats = DefaultFormats

before() {
  contentType = formats("json")
}
```
#### self documenting using swagger
* swagger is a framework for describing, producing, consuming and visualizing REST APIs
* it is a JSON/YAML based specification describing your API
* swagger-ui is a project that reads swagger JSON file and generate web page describing your API
* The machine readable JSON is usied by swagger-ui HTML to autogenerate documents for API
* Scalatra integrate with Swagger in order to autogenerate documentation
* swagger also allow user to invoke API methods with forms generated using swagger
* Scalatra swagger support annotate the controller and methods so that swagger JSOn can be autogenerated automatically
#### swagger support in project
* add scalatra-swagger dependency
```aidl
 "org.scalatra" %% "scalatra-swagger" % ScalatraVersion,
```
* change in controller, adding JacksonSwaggerBase
* this will allow to generate swagger JSON documents for all API methods
```aidl
import org.scalatra.swagger.{Swagger, JacksonSwaggerBase}
class HackersSwagger(implicit val swagger: Swagger)
          extends ScalatraServlet with JacksonSwaggerBase

class ApiController()(implicit val swagger: Swagger)
          extends ApiStack with SwaggerSupport {
```
* change the servlet bootstrap class also to add APi-docs controller
```aidl
import org.scalatra.swagger.{ApiInfo, Swagger}
implicit val apiInfo = new ApiInfo(...)
implicit val swagger = new Swagger("1.2", "1.0.0", apiInfo)
context.mount(..., "/api-docs")

```
#### Documenting the routes
* we can give description about the controller using applicationDescription protected field
```aidl
protected val applicationDescription = ".... "
```
* we provide an apiInfo object to each route to generate documents for eah route
* we have to add apiOperations to describe about the route, its parameters, response
* api operations, its return type and name
```aidl
val listHackers = (apiOperation[List[Hacker]]("listHackers") --> API operation, its return type and name
summary("Show all hackers") --> operation summary
notes("Shows all available hackers.")) --> any special behaviour

get("/", operation(listHackers)) {
   Hacker.all.toList
}
```
* parameters to the swagger documents can be passed as
```aidl
val getHacker = (apiOperation[Hacker]("getHacker")
  summary("Retrieve a single hacker by id")
parameters(
Parameter(
      "id",
      DataType.Int,
      Some("The hacker's database id"),
      None,
      ParamType.Path,
       required = true)
))
```








