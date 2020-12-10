- [Route Management](#route-management)
  - [Page Navigation without named routes](#page-navigation-without-named-routes)
    - [How to use](#how-to-use)
    - [Receive Data from another screen](#receive-data-from-another-screen)
  - [Page Navigation with named routes](#page-navigation-with-named-routes)
    - [How to use](#how-to-use-1)
    - [Send data to named Routes](#send-data-to-named-routes)
    - [Dynamic urls links](#dynamic-urls-links)
  - [Page Navigation Methods](#page-navigation-methods)
  - [Middleware](#middleware)
    - [Priority](#priority)
    - [Redirect](#redirect)
    - [onPageCalled](#onpagecalled)
    - [OnBindingsStart](#onbindingsstart)
    - [OnPageBuildStart](#onpagebuildstart)
    - [OnPageBuilt](#onpagebuilt)
    - [OnPageDispose](#onpagedispose)
  - [Observers](#observers)
  - [SnackBars](#snackbars)
    - [Properties](#properties)
  - [Dialogs](#dialogs)
  - [BottomSheets](#bottomsheets)
  - [Nested Navigation](#nested-navigation)

# Route Management

This is the complete explanation of all there is to Getx when the matter is route management.
If you are going to use routes/snackbars/dialogs/bottomsheets without context, or use the high-level Get APIs, you need to simply add "Get" before your MaterialApp, turning it into GetMaterialApp and enjoy!

```dart
GetMaterialApp( // Before: MaterialApp(
  home: MyHome(),
)
```

## Page Navigation without named routes

### How to use

You can easily switch between you pages:

```dart
/// To navigate to a new screen:
Get.to(NextScreen());

// To close snackbars, dialogs, bottomsheets, or anything you would normally close with Navigator pop(context);
Get.back();

// To go to the next screen and no option to go back to the previous screen (for use in SplashScreens, login screens and etc.)
Get.off(NextScreen());

// To go to the next screen and cancel all previous routes (useful in shopping carts, polls, and tests)
Get.offAll(NextScreen());

```

**NOTE:** when yuo use this method (when you use named routes this is not necessary) you have to place the `Get.put(PageController())` in the build function in the view and not as property.
otherwise the controller will not be initialized. [#818](https://github.com/jonataslaw/getx/issues/818)

```dart
class MyPage extends StatelessWidget {
  MyPage({Key key}) : super(key: key);

  // final Controller controller = Get.put(PageController()); THIS WILL NOT WORK

  @override
  Widget build(BuildContext context) {
    final Controller controller = Get.put(PageController());

    return Scaffold(
      appBar: AppBar(
        title: Text('page2'),
      ),
    );
  }
}
```

### Receive Data from another screen

To navigate to the next route, and receive or update data as soon as you return from it:

```dart
var data = await Get.to(Payment());

// on other screen, send a data for previous route:
Get.back(result: 'success');

// And use it:
if(data == 'success') madeAnything();
```

Don't you want to learn our syntax?
Just change the Navigator (uppercase) to navigator (lowercase), and you will have all the functions of the standard navigation, without having to use context
Example:

```dart

// Default Flutter navigator
Navigator.of(context).push(
  context,
  MaterialPageRoute(
    builder: (BuildContext context) {
      return HomePage();
    },
  ),
);

// Get using Flutter syntax without needing context
navigator.push(
  MaterialPageRoute(
    builder: (_) {
      return HomePage();
    },
  ),
);

// Get syntax (It is much better, but you have the right to disagree)
Get.to(HomePage());


```

## Page Navigation with named routes

### How to use

- If you prefer to navigate by namedRoutes, Get also supports this.

```dart
// To navigate to nextScreen
Get.toNamed("/NextScreen");

// To navigate and remove the previous screen from the tree.
Get.offNamed("/NextScreen");

//To navigate and remove all previous screens from the tree.
Get.offAllNamed("/NextScreen");
```

To define routes, use GetMaterialApp:

```dart
void main() {
  runApp(
    GetMaterialApp(
      initialRoute: '/',
      getPages: [
        GetPage(name: '/', page: () => MyHomePage()),
        GetPage(name: '/second', page: () => Second()),
        GetPage(
          name: '/third',
          page: () => Third(),
          transition: Transition.zoom  
        ),
      ],
    )
  );
}
```

To handle navigation to non-defined routes (404 error), you can define an unknownRoute page in GetMaterialApp.

```dart
void main() {
  runApp(
    GetMaterialApp(
      unknownRoute: GetPage(name: '/notfound', page: () => UnknownRoutePage()),
      initialRoute: '/',
      getPages: [
        GetPage(name: '/', page: () => MyHomePage()),
        GetPage(name: '/second', page: () => Second()),
      ],
    )
  );
}
```

### Send data to named Routes

Just send what you want for arguments. Get accepts anything here, whether it is a String, a Map, a List, or even a class instance.

```dart
// Send the data
Get.toNamed("/NextScreen", arguments: 'Get is the best');

// On your class or controller receive the data
print(Get.arguments); // =>  out: Get is the best
```

### Dynamic urls links

Get offer advanced dynamic urls just like on the Web. Web developers have probably already wanted this feature on Flutter, and most likely have seen a package promise this feature and deliver a totally different syntax than a URL would have on web, but Get also solves that.

```dart
Get.offAllNamed("/NextScreen?device=phone&id=354&name=Enzo");

// on your controller/bloc/stateful/stateless class:
print(Get.parameters['id']); // =>  out: 354
print(Get.parameters['name']); // => out: Enzo
```

You can also receive NamedParameters with Get easily:

```dart
void main() {
  runApp(
    GetMaterialApp(
      initialRoute: '/',
      getPages: [
      GetPage(
        name: '/',
        page: () => MyHomePage(),
      ),
       //You can define a different page for routes with arguments, and another without arguments, but for that you must use the slash '/' on the route that will not receive arguments as above.
       GetPage(
        name: '/profile/:user',
        page: () => UserProfile(),
      ),
     ],
    )
  );
}

// Send data on route name
Get.toNamed("/profile/34954");

//On second screen take the data by parameter
print(Get.parameters['user']); //=>  out: 34954
```

And now, all you need to do is use Get.toNamed() to navigate your named routes, without any context (you can call your routes directly from your BLoC or Controller class), and when your app is compiled to the web, your routes will appear in the url <3

## Page Navigation Methods

| Method                                                                                                                      | Need           | Method in Navigator                        | Descriptions                                                                                                                   |
| --------------------------------------------------------------------------------------------------------------------------- | -------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| [Get.to()](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L473)            | Widget         | Navigation.push()                          | Pushes a new (Widget) page to the stack                                                                                        |
| [Get.until()        ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L580) | -              | Navigation.popUntil()                      | Calls pop several times in the stack until `predicate` returns true                                                            |
| [Get.toNamed()      ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L525) | String         | Navigation.pushNamed()                     | Pushes a new named page to the stack.                                                                                          |
| [Get.offNamed()     ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L553) | String         | Navigation.pushReplacementNamed()          | Pop the current named page in the stack and push a new one in its place                                                        |
| [Get.offUntil()     ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L604) | Route<Dynamic> | Navigation.pushAndRemoveUntil()            | Push the given page, and then pop several pages in the stack until `predicate` returns true                                    |
| [Get.offNamedUntil()](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L628) | String         | Navigation.pushNamedAndRemoveUntil()       | Push the given named page, and then pop several pages in the stack until `predicate` returns true                              |
| [Get.offAndToNamed()](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L650) | String         | Navigation.popAndPushNamed()               | Pop the current named page and pushes a new page to the stack in its place                                                     |
| [Get.removeRoute()  ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L667) | Route<Dynamic> | Navigation.removeRoute()                   | Remove a specific route from the stack                                                                                         |
| [Get.offAllNamed()  ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L691) | String         | Navigation.pushNamedAndRemoveUntil()       | Push a named page and pop several pages in the stack until `predicate` returns true. predicate` is optional                    |
| [Get.back()         ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L724) | -              | Navigation?.popUntil() or Navigation.pop() | if your set `closeOverlays` to true, Get.back() will close the currently open snackbar/dialog/bottomsheet AND the current page |
| [Get.close()        ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L750) | -              | Navigation.popUntil()                      | Close as many routes as defined by `times`                                                                                     |
| [Get.off()          ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L785) | Widget         | Navigation.pushReplacement()               | Pop the current page and pushes a new page to the stack                                                                        |
| [Get.offAll()       ](https://github.com/jonataslaw/getx/blob/master/lib/get_navigation/src/extension_navigation.dart#L846) | Widget         | Navigation.pushAndRemoveUntil()            | Push a named page and pop several pages in the stack until `predicate` returns true. predicate` is optional                    |

## Middleware

The GetPage has now new property that takes a list of GetMiddleWare and run them in the specific order.

**Note**: When GetPage has a Middlewares, all the children of this page will have the same middlewares automatically.

### Priority

The Order of the Middlewares to run can pe set by the priority in the GetMiddleware.

```dart
final middlewares = [
  GetMiddleware(priority: 2),
  GetMiddleware(priority: 5),
  GetMiddleware(priority: 4),
  GetMiddleware(priority: -8),
];
```
those middlewares will be run in this order **-8 => 2 => 4 => 5**

### Redirect

This function will be called when the page of the called route is being searched for. It takes RouteSettings as a result to redirect to. Or give it null and there will be no redirecting.

```dart
GetPage redirect( ) {
  final authService = Get.find<AuthService>();
  return authService.authed.value ? null : RouteSettings(name: '/login')
}
```

### onPageCalled

This function will be called when this Page is called before anything created
you can use it to change something about the page or give it new page

```dart
GetPage onPageCalled(GetPage page) {
  final authService = Get.find<AuthService>();
  return page.copyWith(title: 'Welcome ${authService.UserName}');
}
```

### OnBindingsStart

This function will be called right before the Bindings are initialize.
Here you can change Bindings for this page.

```dart
List<Bindings> onBindingsStart(List<Bindings> bindings) {
  final authService = Get.find<AuthService>();
  if (authService.isAdmin) {
    bindings.add(AdminBinding());
  }
  return bindings;
}
```

### OnPageBuildStart

This function will be called right after the Bindings are initialize.
Here you can do something after that you created the bindings and before creating the page widget.

```dart
GetPageBuilder onPageBuildStart(GetPageBuilder page) {
  print('bindings are ready');
  return page;
}
```

### OnPageBuilt

This function will be called right after the GetPage.page function is called and will give you the result of the function. and take the widget that will be showed.

### OnPageDispose

This function will be called right after disposing all the related objects (Controllers, views, ...) of the page.

## Observers

If you want listen Get events to trigger actions, you can to use routingCallback to it

```dart
GetMaterialApp(
  routingCallback: (routing) {
    if(routing.current == '/second'){
      openAds();
    }
  }
)
```

If you are not using GetMaterialApp, you can use the manual API to attach Middleware observer.

```dart
void main() {
  runApp(
    MaterialApp(
      onGenerateRoute: Router.generateRoute,
      initialRoute: "/",
      navigatorKey: Get.key,
      navigatorObservers: [
        GetObserver(MiddleWare.observer), // HERE !!!
      ],
    ),
  );
}

//Create a MiddleWare class
class MiddleWare {
  static observer(Routing routing) {
    /// You can listen in addition to the routes, the snackbars, dialogs and bottomsheets on each screen.
    ///If you need to enter any of these 3 events directly here,
    ///you must specify that the event is != Than you are trying to do.
    if (routing.current == '/second' && !routing.isSnackbar) {
      Get.snackbar("Hi", "You are on second route");
    } else if (routing.current =='/third'){
      print('last route called');
    }
  }
}
```

Now, use Get on your code:

```dart
class First extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        leading: IconButton(
          icon: Icon(Icons.add),
          onPressed: () {
            Get.snackbar("hi", "i am a modern snackbar");
          },
        ),
        title: Text('First Route'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Open route'),
          onPressed: () {
            Get.toNamed("/second");
          },
        ),
      ),
    );
  }
}

class Second extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        leading: IconButton(
          icon: Icon(Icons.add),
          onPressed: () {
            Get.snackbar("hi", "i am a modern snackbar");
          },
        ),
        title: Text('second Route'),
      ),
      body: Center(
        child: RaisedButton(
          child: Text('Open route'),
          onPressed: () {
            Get.toNamed("/third");
          },
        ),
      ),
    );
  }
}

class Third extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Third Route"),
      ),
      body: Center(
        child: RaisedButton(
          onPressed: () {
            Get.back();
          },
          child: Text('Go back!'),
        ),
      ),
    );
  }
}
```

## SnackBars

To have a simple SnackBar with Flutter, you must get the context of Scaffold, or you must use a GlobalKey attached to your Scaffold

```dart
final snackBar = SnackBar(
  content: Text('Hi!'),
  action: SnackBarAction(
    label: 'I am a old and ugly snackbar :(',
    onPressed: (){}
  ),
);

// Find the Scaffold in the widget tree and use
// it to show a SnackBar.
Scaffold.of(context).showSnackBar(snackBar);
```

With Get:

```dart
Get.snackbar('Hi', 'i am a modern snackbar');
```

With Get, all you have to do is call your Get.snackbar from anywhere in your code or customize it however you want!

```dart
Get.snackbar(
  "Hey i'm a Get SnackBar!", // title
  "It's unbelievable! I'm using SnackBar without context, without boilerplate, without Scaffold, it is something truly amazing!", // message
  icon: Icon(Icons.alarm),
  shouldIconPulse: true,
  onTap:(){},
  barBlur: 20,
  isDismissible: true,
  duration: Duration(seconds: 3),
);
```

### Properties

| Property                         | Type                            | Description                                                                                                                                                                                                                                                     |
| -------------------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| snackbarStatus                   | Function(SnackbarStatus status) | A callback for you to listen to the different Snack status                                                                                                                                                                                                      |
| title                            | String                          | The title displayed to the user                                                                                                                                                                                                                                 |
| message                          | String                          | The message displayed to the user.                                                                                                                                                                                                                              |
| titleText                        | Widget                          | Replaces `title`. Although this accepts a `Widget`, it is meant to receive `text` or `RichText`                                                                                                                                                                 |
| messageText                      | Widget                          | Replaces `message`. Although this accepts a `Widget`, it is meant to receive `text` or `RichText`                                                                                                                                                               |
| backgroundColor                  | Color                           | Will be ignored if `backgroundGradient` is not null                                                                                                                                                                                                             |
| leftBarIndicatorColor            | Color                           | If not null, shows a left vertical colored bar on notification. It is not possible to use it with a `Form` and I do not recommend using it with `LinearProgressIndicator`                                                                                       |
| boxShadows                       | List<BoxShadow>                 | `boxShadows` The shadows generated by Snack. Leave it null if you don't want a shadow. You can use more than one if you feel the need. Check [this example](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/material/shadows.dart)      |
| backgroundGradient               | Gradient                        | Makes `backgroundColor` be ignored                                                                                                                                                                                                                              |
| icon                             | Widget                          | You can use any widget here, but I recommend `Icon` or `Image` as indication of  kind of message you are displaying. Other widgets may break the layout                                                                                                         |
| shouldIconPulse                  | bool                            | An option to animate the icon (if present). Defaults to true.                                                                                                                                                                                                   |
| maxWidth                         | double                          | Used to limit Snack width (usually on large screens)                                                                                                                                                                                                            |
| margin                           | EdgeInsets                      | Adds a custom margin to Snack                                                                                                                                                                                                                                   |
| padding                          | EdgeInsets                      | Adds a custom padding to Snack The default follows material design guide line                                                                                                                                                                                   |
| borderRadius                     | double                          | Adds a radius to all corners of Snack. Best combined with `margin`. I do not recommend using it with `showProgressIndicator` or `leftBarIndicatorColor`.                                                                                                        |
| borderColor                      | Color                           | Adds a border to every side of Snack I do not recommend using it with `showProgressIndicator` or `leftBarIndicatorColor`                                                                                                                                        |
| borderWidth                      | double                          | Changes the width of the border if `borderColor` is specified                                                                                                                                                                                                   |
| mainButton                       | FlatButton                      | A `FlatButton` widget if you need an action from the user.                                                                                                                                                                                                      |
| onTap                            | OnTap                           | A callback that registers the user's click anywhere. An alternative to `mainButton`                                                                                                                                                                             |
| isDismissible                    | bool                            | Determines if the user can swipe or click the overlay (if `overlayBlur` > 0) to dismiss. It is recommended that you set `duration` != null if this is false. If the user swipes to dismiss or clicks the overlay, no value will be returned.                    |
| showProgressIndicator            | bool                            | True if you want to show a `LinearProgressIndicator`                                                                                                                                                                                                            |
| progressIndicatorController      | AnimationController             | An optional `AnimationController` when you to control the progress of your `LinearProgressIndicator`                                                                                                                                                            |
| progressIndicatorBackgroundColor | Color                           | A `LinearProgressIndicator` configuration parameter                                                                                                                                                                                                             |
| progressIndicatorValueColor      | Animation<Color>                | A `LinearProgressIndicator` configuration parameter.                                                                                                                                                                                                            |
| snackStyle                       | SnackStyle                      | Snack can be floating or be grounded to the edge of the screen. If grounded, I do not recommend using `margin` or `borderRadius`. `SnackStyle.FLOATING` is the default If grounded, I do not recommend using a `backgroundColor` with transparency or `barBlur` |
| forwardAnimationCurve            | Curve                           | The `Curve` animation used when show() is called.`Curves.easeOut` is default                                                                                                                                                                                    |
| reverseAnimationCurve            | Curve                           | The `Curve` animation used when dismiss() is called. `Curves.fastOutSlowIn` is default                                                                                                                                                                          |
| animationDuration                | Duration                        | Use it to speed up or slow down the animation duration                                                                                                                                                                                                          |
| barBlur                          | double                          | Default is 0.0. If different than 0.0, blurs only Snack's background. To take effect, make sure your `backgroundColor` has some opacity. The greater the value, the greater the blur.                                                                           |
| overlayBlur                      | double                          | Default is 0.0. If different than 0.0, creates a blurred overlay that prevents the user from interacting with the screen. The greater the value, the greater the blur.                                                                                          |
| overlayColor                     | Color                           | Default is `Colors.transparent`. Only takes effect if `overlayBlur` > 0.0. Make sure you use a color with transparency here e.g. Colors.grey`600`.withOpacity(0.2).                                                                                             |  
| userInputForm                    | Form                            | A `TextFormField` in case you want a simple user input. Every other widget is ignored if this is not null.                                                                                                                                                     |


If you prefer the traditional snackbar, or want to customize it from scratch, including adding just one line (Get.snackbar makes use of a mandatory title and message), you can use
`Get.rawSnackbar();` which provides the RAW API on which Get.snackbar was built.

## Dialogs

To open dialog:

```dart
Get.dialog(YourDialogWidget());
```

To open default dialog:

```dart
Get.defaultDialog(
  onConfirm: () => print("Ok"),
  middleText: "Dialog made in 3 lines of code"
);
```

You can also use Get.generalDialog instead of showGeneralDialog.

For all other Flutter dialog widgets, including cupertinos, you can use Get.overlayContext instead of context, and open it anywhere in your code.
For widgets that don't use Overlay, you can use Get.context.
These two contexts will work in 99% of cases to replace the context of your UI, except for cases where inheritedWidget is used without a navigation context.

## BottomSheets

Get.bottomSheet is like showModalBottomSheet, but don't need of context.

```dart
Get.bottomSheet(
  Container(
    child: Wrap(
      children: <Widget>[
        ListTile(
          leading: Icon(Icons.music_note),
          title: Text('Music'),
          onTap: () => {}
        ),
        ListTile(
          leading: Icon(Icons.videocam),
          title: Text('Video'),
          onTap: () => {},
        ),
      ],
    ),
  )
);
```

## Nested Navigation

Get made Flutter's nested navigation even easier.
You don't need the context, and you will find your navigation stack by Id.

- NOTE: Creating parallel navigation stacks can be dangerous. The ideal is not to use NestedNavigators, or to use sparingly. If your project requires it, go ahead, but keep in mind that keeping multiple navigation stacks in memory may not be a good idea for RAM consumption.

See how simple it is:

```dart
Navigator(
  key: Get.nestedKey(1), // create a key by index
  initialRoute: '/',
  onGenerateRoute: (settings) {
    if (settings.name == '/') {
      return GetPageRoute(
        page: () => Scaffold(
          appBar: AppBar(
            title: Text("Main"),
          ),
          body: Center(
            child: FlatButton(
              color: Colors.blue,
              onPressed: () {
                Get.toNamed('/second', id:1); // navigate by your nested route by index
              },
              child: Text("Go to second"),
            ),
          ),
        ),
      );
    } else if (settings.name == '/second') {
      return GetPageRoute(
        page: () => Center(
          child: Scaffold(
            appBar: AppBar(
              title: Text("Main"),
            ),
            body: Center(
              child:  Text("second")
            ),
          ),
        ),
      );
    }
  }
),
```
