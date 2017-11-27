---
layout: angular
title: HTTP Client
description: An HTTP Client interacts with the remote server.
sideNavGroup: advanced
prevpage:
  title: Hierarchical Dependency Injectors
  url: /angular/guide/hierarchical-dependency-injection
nextpage:
  title: Lifecycle Hooks
  url: /angular/guide/lifecycle-hooks
---
<!-- FilePath: src/angular/guide/server-communication.md -->
<?code-excerpt path-base="examples/ng/doc/server-communication"?>

[HTTP](https://tools.ietf.org/html/rfc2616) is the primary protocol for browser/server communication.

<div class="l-sub-section" markdown="1">
  The [`WebSocket`](https://tools.ietf.org/html/rfc6455) protocol is another important communication technology;
  it isn't covered in this page.
</div>

Modern browsers support the following HTTP-based APIs:

* [XMLHttpRequest (XHR)](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 


* [JSONP](https://en.wikipedia.org/wiki/JSONP)


Additionally, other browsers support [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

The Dart [http][] library simplifies app programming with the **XHR** and **JSONP** APIs.

The <live-example>live example</live-example> exemplifies the usage of the **XHR** and **JSONP** APIs .

## Demos

The following demos explain server communication. 

- [The Tour of Heroes *HTTP* client demo](#http-client).
- [Cross-Origin Requests: Wikipedia example](#cors).

The following code shows how to use the root `AppComponent`:

<?code-excerpt "lib/app_component.dart" title?>
```
  import 'package:angular/angular.dart';

  import 'src/toh/hero_list_component.dart';
  import 'src/wiki/wiki_component.dart';
  import 'src/wiki/wiki_smart_component.dart';

  @Component(
      selector: 'my-app',
      template: '''
        <hero-list></hero-list>
        <my-wiki></my-wiki>
        <my-wiki-smart></my-wiki-smart>
      ''',
      directives: const [
        HeroListComponent,
        WikiComponent,
        WikiSmartComponent
      ])
  class AppComponent {}
```

<div class="l-main-section"></div>
# Providing HTTP services  {#http-providers}

The Dart `BrowserClient` client communicates with the server through an HTTP request/response protocol.
The `BrowserClient` client is a group of services in the Dart [http][] library.

To provide HTTP services, follow these steps:
1. Configure the app to use server communication facilities.
2. Register the `BrowserClient` client as a service provider with the dependency injection system. 

<div class="l-sub-section" markdown="1">
For more information about providers, see the [Dependency Injection](dependency-injection.html) page.
</div>

The followin code shows how to use the `bootstrap()` method:

<?code-excerpt "web/main.dart (v1)" title?>
```
  import 'package:angular/angular.dart';
  import 'package:server_communication/app_component.dart';

  void main() {
    bootstrap(AppComponent, [
      provide(BrowserClient, useFactory: () => new BrowserClient(), deps: [])
    ]);
  }
```

<div id="http-client"></div>
## The Tour of Heroes HTTP client demo

This demo is a shorter version of the [Tour of Heroes](/angular/tutorial) app. In this demo, the heroes are received from the server and displayed as a list. A user can add new heroes and save them to the server. The app uses the Dart `BrowserClient` client to communicate via **XMLHttpRequest (XHR)** .
The following image displays the process of adding and saving heroess:

<img class="image-display" src="{% asset_path 'ng/devguide/server-communication/http-toh.gif' %}" alt="ToH mini app" width="282">

The following code is the `HeroListComponent` component used in the demo:

<?code-excerpt "lib/src/toh/hero_list_component.html" title?>
```
  <h1>Tour of Heroes</h1>
  <h3>Heroes:</h3>
  <ul>
    <li *ngFor="let hero of heroes">{!{hero.name}!}</li>
  </ul>

  <label>New hero name: <input #newHeroName /></label>
  <button (click)="addHero(newHeroName.value); newHeroName.value=''">Add Hero</button>

  <p class="error" *ngIf="errorMessage != null">{!{errorMessage}!}</p>
```

In the code, the list of heroes is specified as `ngFor`. A user can enter the names of new heroes in the input box and add them to the database by clicking the *Add Hero* button.

A [template reference variable](template-syntax.html#ref-vars), `newHeroName`, accesses the value of the input box in the `(click)` event binding. When the user clicks the button, that value is passed to `addHero` method of the component and then,
the event binding clears the method to accept the name of a new hero. If an error occurs, it is displayed below the *Add Hero* button.

<div id="oninit"></div>
<div id="HeroListComponent"></div>
### The *HeroListComponent* class

The following code is *HeroListComponent* class:

<?code-excerpt "lib/src/toh/hero_list_component.dart (class)" region="component" title?>
```
  class HeroListComponent implements OnInit {
    final HeroService _heroService;
    String errorMessage;
    List<Hero> heroes = [];

    HeroListComponent(this._heroService);

    Future<Null> ngOnInit() => getHeroes();

    Future<Null> getHeroes() async {
      try {
        heroes = await _heroService.getHeroes();
      } catch (e) {
        errorMessage = e.toString();
      }
    }

    Future<Null> addHero(String name) async {
      name = name.trim();
      if (name.isEmpty) return;
      try {
        heroes.add(await _heroService.create(name));
      } catch (e) {
        errorMessage = e.toString();
      }
    }
  }
```

Angular injects `HeroService` into the constructor and the `HeroList` component calls the service to fetch and save data. For more information about injection, see the [injects](dependency-injection) page.

The component does not interact directly with the Dart `BrowserClient` client. It only delegates the data to the `HeroService`.

<div class="l-sub-section" markdown="1">
Note **Always delegate data access to a supporting service class**.
</div>

Although _at runtime_ the component requests heroes as soon as they are created, the `get` method of the service in the component's constructor is not called immediately. It is called inside the`ngOnInit` life cycle hook and relies on the Angular to call `ngOnInit` when it instantiates this component. For more information, see the [lifecycle hook](lifecycle-hooks) page.

<div class="l-sub-section" markdown="1">
  Note
  Components are easier to test and debug when their constructors are simple and tasks such as calling a remote server are handled through a separate method.
  </div>

The hero service `getHeroes()` return the [Future][]
values of the current hero list and the `create()` asynchronous method returns the newly added hero.
The `getHeroes()` and `addHero()` methods in the hero list specify the actions taken when the asynchronous method calls succeed or fail.

For more information about `Future`, see the articles listed on the [articles]({{site.dartlang}}/articles) page about asynchronous
programming. Additionally, you can see the tutorial on,
 [_Asynchronous Programming: Futures_]({{site.dartlang}}/tutorials/language/futures).

With a basic understanding of the component you can understand `HeroService`.

<div id="HeroService"></div>
## Fetch data with _http.get()_  {#fetch-data}

In the abovementioned samples, the app created fake interaction with the server by
returning mock heroes in a service as follows: 

<?code-excerpt "../toh-4/lib/src/hero_service.dart"?>
```
  import 'dart:async';

  import 'package:angular/angular.dart';

  import 'hero.dart';
  import 'mock_heroes.dart';

  @Injectable()
  class HeroService {
    Future<List<Hero>> getHeroes() async => mockHeroes;
  }
```

The following code is used to revise the  `HeroService` to get the heroes from the server using the Dart `BrowserClient` client service:

<?code-excerpt "lib/src/toh/hero_service.dart (revised)" region="v1" title?>
```
  import 'dart:async';
  import 'dart:convert';

  import 'package:angular/angular.dart';
  import 'package:http/http.dart';

  import 'hero.dart';

  @Injectable()
  class HeroService {
    static final _headers = {'Content-Type': 'application/json'};
    static const _heroesUrl = 'api/heroes'; // URL to web API
    final Client _http;

    HeroService(this._http);

    Future<List<Hero>> getHeroes() async {
      try {
        final response = await _http.get(_heroesUrl);
        final heroes = _extractData(response)
            .map((value) => new Hero.fromJson(value))
            .toList();
        return heroes;
      } catch (e) {
        throw _handleError(e);
      }
    }

    Future<Hero> create(String name) async {
      try {
        final response = await _http.post(_heroesUrl,
            headers: _headers, body: JSON.encode({'name': name}));
        return new Hero.fromJson(_extractData(response));
      } catch (e) {
        throw _handleError(e);
      }
    }
```

The following code shows how the Dart `BrowserClient` client service is injected into the `HeroService` constructor:

<?code-excerpt "lib/src/toh/hero_service.dart (ctor)"?>
```
  HeroService(this._http);
```

The following code show how to call the `_http.get`service:

<?code-excerpt "lib/src/toh/hero_service.dart (getHeroes)" region="http-get" title?>
```
  static const _heroesUrl = 'api/heroes'; // URL to web API
  Future<List<Hero>> getHeroes() async {
    try {
      final response = await _http.get(_heroesUrl);
      final heroes = _extractData(response)
          .map((value) => new Hero.fromJson(value))
          .toList();
      return heroes;
    } catch (e) {
      throw _handleError(e);
    }
  }
```

The resource URL is passed to `get`, it calls the server which returns heroes.

<div class="l-sub-section" markdown="1">
  Set up the in-memory web api as described in the [tutorial](/angular/tutorial). After the setup, the server returns heroes.
  Alternatively, you can target a JSON file by changing the endpoint URL as follows:

<?code-excerpt "lib/src/toh/hero_service.dart (endpoint-json)"?>
```
  static const _heroesUrl = 'heroes.json'; // URL to JSON file
```
</div>

<div id="extract-data"></div>
## Process the response object

As explained earlier, the `getHeroes()` method uses an `_extractData()` helper method to map the `_http.get` response object to heroes as follows:

<?code-excerpt "lib/src/toh/hero_service.dart (excerpt)" region="extract-data" title?>
```
  dynamic _extractData(Response resp) => JSON.decode(resp.body)['data'];
```

The `response` object does not hold the data in a format that can be used directly through the app. The response data is parsed into a JSON object.

### Parse to JSON

The response data are in JSON string form. The user must parse the string into objects, by calling the `JSON.decode()` method from the `dart:convert` library as follows:

<div class="l-sub-section" markdown="1">
  The decoded JSON does not list the heroes. The server wraps JSON results in an object with a `data`
  property. The object must be unwrapped to get the heroes. 
  This web API behavior is driven by security concerns listed on the 
  [security concerns](https://www.owasp.org/index.php/OWASP_AJAX_Security_Guidelines#Always_return_JSON_with_an_Object_on_the_outside) page.
</div>

<div class="alert is-important" markdown="1">
      Not all servers return an object with a `data` property.
</div>

<div id="no-return-response-object"></div>
### Do not return the response object

The `getHeroes()` method may erroneously return HTTP response.
The data service hides the server interaction details from the user.
The component that calls the `HeroService` requires only heroes. 

<div id="error-handling"></div>
### Always handle errors

An important aspect of working with input and output is finding and resolving errors. One method to handle errors is by sending a message with decription of the error and solution to the user.

This app does not generate appropriate error messages for `getHeroes` as shown below:

<?code-excerpt "lib/src/toh/hero_service.dart (excerpt)" region="error-handling" title?>
```
  Future<List<Hero>> getHeroes() async {
    try {
      final response = await _http.get(_heroesUrl);
      final heroes = _extractData(response)
          .map((value) => new Hero.fromJson(value))
          .toList();
      return heroes;
    } catch (e) {
      throw _handleError(e);
    }
  }

  Exception _handleError(dynamic e) {
    print(e); // for demo purposes only
    return new Exception('Server error; cause: $e');
  }
```
{%comment%} block error-handling - TODO: describe `_handleError`?
  The `catch()` operator passes the error object from `http` to the `handleError()` method.
  The `handleError` method transforms the error into a user-friendly message,
  logs it to the console, and returns the message as a new, failed Observable via `Observable.throw`.
{%endcomment%}

<div id="subscribe"></div>
### _HeroListComponent_ error handling  {#hero-list-component}

In the `HeroListComponent`, the call to
`_heroService.getHeroes()` is wrapped in a `try` clause. When an error is
generated, the `errorMessage` variable is assigned the error as follows: 

<?code-excerpt "lib/src/toh/hero_list_component.dart (getHeroes)" title?>
```
  Future<Null> getHeroes() async {
    try {
      heroes = await _heroService.getHeroes();
    } catch (e) {
      errorMessage = e.toString();
    }
  }
```

<div class="l-sub-section" markdown="1">
  To create a failure scenario, reset the api endpoint to a bad value in `HeroService`. Ensure that the api endpoint is restored to its original value. 

</div>

<div id="create"></div><div id="update"></div>
## Send data to the server  {#post}

This page describes the functionality to create new heroes and save them in the code.

Create a method for the `HeroListComponent` to call a `create()` method. This method takes
the name of a new hero.The following code sample explains this scenario:

<?code-excerpt "lib/src/toh/hero_service.dart (create-sig)"?>
```
  Future<Hero> create(String name) async {
```

To implement this scenario, the user must know the  server's API for creating heroes.
[This sample's data server](/angular/tutorial/toh-pt6#simulate-the-web-api) follows typical REST guidelines.
A [`POST`](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.5) request must be created
at the same endpoint as `GET` heroes.
The new hero data should arrive in the body of the request,
structured like a `Hero` entity but without the `id` property.
The following code sample shows the body of the request: 

<?code-excerpt?>
```javascript
  { "name": "Windstorm" }
```

The server generates the `id` and returns the `JSON` representation
of the new hero including the generated id. The hero is inside a response object
with its own `data` property.

 `create()` is implemented as follows:

<?code-excerpt "lib/src/toh/hero_service.dart (create)" title?>
```
  Future<Hero> create(String name) async {
    try {
      final response = await _http.post(_heroesUrl,
          headers: _headers, body: JSON.encode({'name': name}));
      return new Hero.fromJson(_extractData(response));
    } catch (e) {
      throw _handleError(e);
    }
  }
```

### Headers

In the `headers` object, the `Content-Type` specifies that the body represents JSON.

### JSON results

Similar to `getHeroes()`, use the `_extractData()` helper to [extract the data](#extract-data)
from the response.

As described earlier,  in the `HeroListComponent`, the `addHero()` method waits for the 
asynchronous  service `create()` method to create a hero. When `create()` is finished,
`addHero()` displays the new hero in the `heroes` list as follows.

<?code-excerpt "lib/src/toh/hero_list_component.dart (addHero)" title?>
```
  Future<Null> addHero(String name) async {
    name = name.trim();
    if (name.isEmpty) return;
    try {
      heroes.add(await _heroService.create(name));
    } catch (e) {
      errorMessage = e.toString();
    }
  }
```

## Cross-Origin Requests: Wikipedia example  {#cors}

The most common approach to server communication is by making `XMLHttpRequests` using the Dart `BrowserClient` service.
However, it does not work in all scenarios. 

For security reasons, web browsers block `XHR` calls to a remote server whose origin is different from the origin of the web page.
The *origin* is the combination of URI scheme, hostname, and port number.
This is called the [same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy).

<div class="l-sub-section" markdown="1">
  Modern browsers allow `XHR` requests to servers from a different origin if the server supports the
  [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) protocol.
  You can enable the user credentials in the [request headers](#headers).
</div>

Some servers do not support CORS but do support an older, read-only alternative called [JSONP](https://en.wikipedia.org/wiki/JSONP).
Wikipedia is an example of this server. 

<div class="l-sub-section" markdown="1">
  
For more information about JSONP, see  [Stack Overflow answer](http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about/2067584#2067584).
</div>

### Search Wikipedia

The following simple search shows suggestions from Wikipedia as the user
types in a text box:

<img class="image-display" src="{% asset_path 'ng/devguide/server-communication/wiki-1.gif' %}" alt="Wikipedia search app (v.1)" width="282">

Wikipedia offers a modern `CORS` API and a legacy `JSONP` search API.

<div class="alert is-important" markdown="1">
  The remaining content of this section is coming soon.
  For more information about how to access Wikipedia via the `JSON` API, see
  [example sources]({{site.ghNgEx}}/server-communication/tree/{{site.branch}})
  </div>

{%comment%}
==============================================================================
==============================================================================
==============================================================================

//- block wikipedia-jsonp+ unused portion from TS
Wikipedia offers a modern `CORS` API and a legacy `JSONP` search API. This example uses this API.
The Angular `Jsonp` service extends the `BrowserClient` service for JSONP and also restricts the `GET` requests.
All other HTTP methods throw an error because `JSONP` is a read-only facility.

Wrap the interaction with an Angular data access client service inside a dedicated service. For example, `WikipediaService`.

<?xcode-excerpt "lib/src/wiki/wikipedia_service.dart" title linenums?>

The constructor expects Angular to inject its `Jsonp` service, which
is available because `JsonpModule` is in the root `@NgModule` `imports` array
in `app.module.ts`.

### Search parameters  {#query-parameters}

The [Wikipedia "opensearch" API](https://www.mediawiki.org/wiki/API:Opensearch)
expects four parameters (key/value pairs) to arrive in the request URL's query string.
The keys are `search`, `action`, `format`, and `callback`.
The value of the `search` key is the user-supplied search term in Wikipedia.
The other three are the fixed values "opensearch", "json", and "JSONP_CALLBACK", respectively.

<div class="l-sub-section" markdown="1">
    The `JSONP` technique requires you to pass a callback function name to the server in the query string: `callback=JSONP_CALLBACK`.
    The server uses that name to build a JavaScript wrapper function in its response, which Angular ultimately calls to extract the data.
    </div>

To search for articles with the term, "Angular", construct the query string called `jsonp` as follows:

<?xcode-excerpt "lib/src/wiki/wikipedia_service_1.dart (query-string)"?>

In more parameterized example, build the query string with the Angular `URLSearchParams` helper:

<?xcode-excerpt "lib/src/wiki/wikipedia_service.dart (search parameters)" region="search-parameters" title?>

Call `jsonp` with *two* arguments: the `wikiUrl` and an options object whose `search` property is the `params` object.

<?xcode-excerpt "lib/src/wiki/wikipedia_service.dart (call jsonp)" region="call-jsonp" title?>

`Jsonp` flattens the `params` object into the same query string and sends the request to the server.

### The WikiComponent  {#wikicomponent}

The abovementiond service can query the Wikipedia API.
The component (template and class) takes user input and displays search results.

<?xcode-excerpt "lib/src/wiki/wiki_component.dart" title linenums?>

The template presents an `<input>` element *search box* to gather search terms from the user,
and calls a `search(term)` method after each `keyup` event.

The component's `search(term)` method delegates to the `WikipediaService`, which returns an
Observable list of string results (`Observable<string[]>`).
Instead of subscribing to the Observable inside the component, as in the `HeroListComponent`,
the app forwards the Observable result to the template (via `items`) where the `async` pipe
in the `ngFor` handles the subscription. For more information, see [async pipes](pipes.html#async-pipe)
on the [Pipes](pipes.html) page.

<div class="l-sub-section" markdown="1">
  The [async pipe](pipes.html#async-pipe) has read-only components, which do not interact with the data.

  `HeroListComponent` can't use the pipe because `addHero()` pushes newly created heroes into the list.
</div>

## A wasteful app  {#wasteful-app}

The Wikipedia search makes multiple calls to the server.This method is inefficient and potentially expensive on mobile devices with limited data plans.

### 1. Wait for the user to stop typing

The code calls the server after every keystroke.
and sends a request to the server after the user stops typing.
The following sample code shows how the refactoring affects the code: 

<img class="image-display" src="{% asset_path 'ng/devguide/server-communication/wiki-2.gif' %}" alt="Wikipedia search app (v.2)" width="250">

### 2. Search when the search term changes

IF a user enters the word *angular* in the search box and pauses for a while, the app issues a search request for *angular*.

If the user deletes the last three letters, *lar*, and re-types *lar* before pausing again.
The search term is still _angular_. Therore, the app should not make another request. 

### 3. Cope with out-of-order responses

The user enters *angular*, pauses, clears the search box, and enters *http*.
The app issues two search requests, one for *angular* and the other for *http*.

The user does not know which response was recorded first. When there are multiple requests in-flight, the app presents the responses in the original request order.
In this example, the app displays the results for the *http* search
irrespective of the order of the response. 

<div id="more-observables"></div>
## More fun with Observables

A user can make changes to the `WikipediaService` service. However, for a better
user experience, create a copy of the `WikiComponent`. This can be done through,
Observable operators.

In the following code, `WikiSmartComponent` component is displayed next to the original `WikiComponent`:

<code-tabs>
  <?code-pane "lib/src/wiki/wiki_smart_component.dart" linenums?>
  <?code-pane "lib/src/wiki_component.dart" linenums?>
</code-tabs>

While the templates are virtually identical,
there are more RxJS such as, `debounceTime`, `distinctUntilChanged`, and `switchMap` operators in the new version.
These are imported as [described above](#rxjs-library).

<div id="create-stream"></div>
### Create a stream of search terms

The `WikiComponent` passes a new search term directly to the `WikipediaService` after every keystroke.

The `WikiSmartComponent` class turns the keystrokes into an Observable _stream of search terms_
with the help of a `Subject`, which is imported from RxJS:

<?xcode-excerpt "lib/src/wiki/wiki_smart_component.dart (import-subject)"?>

The component creates a `searchTermStream` as a subject of type `string`.
The `search()` method adds each new search box value to that stream through the `next()` method of the subject.

<?xcode-excerpt "lib/src/wiki/wiki_smart_component.dart (subject)"?>

### Listen for search terms

The `WikiSmartComponent` records the *stream of search terms* and
processes that stream _before_ calling the service.

<?xcode-excerpt "lib/src/wiki/wiki_smart_component.dart (observable-operators)"?>

* <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/debounce.md" target="_blank" rel="noopener" title="debounce operator"><i>debounceTime</i></a>
waits for the user to stop typing for at least 300 milliseconds.

* <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/distinctuntilchanged.md" target="_blank" rel="noopener" title="distinctUntilChanged operator"><i>distinctUntilChanged</i></a>
ensures that the service is called only when the new search term is different from the previous search term.

* The <a href="https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/flatmaplatest.md" target="_blank" rel="noopener" title="switchMap operator"><i>switchMap</i></a>
calls the `WikipediaService` with a fresh, debounced search term and coordinates the stream(s) of service response.

The role of `switchMap` is particularly important.
The `WikipediaService` returns a separate Observable of string arrays (`Observable<string[]>`) for each search request.
The user could issue multiple requests before a slow server has had time to reply,
which means a backlog of response Observables could arrive at the client, at any moment, in any order.

The `switchMap` returns Observable that _combines_ all `WikipediaService` responses. Observables,
re-arrange them in their original request order, and delivers only the most recent search results to subscribers.

//- Skip Cross-Site Request Forgery section for now.
//- Drop "in-memory web api" appendix since we refer readers to the tutorial.
==============================================================================
==============================================================================
==============================================================================
{%endcomment%}

See the full source code in the <live-example></live-example>.

[Future]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-async/Future-class.html
[http]: https://pub.dartlang.org/packages/http
