---
banner: "https://thoughtfocus.com/wp-content/uploads/2025/03/flutter_banner-scaled-1.jpg"
tags:
  - flutter
banner_y: 0.5
---
## MVVM - Model View View-Model architecture

- Model : the data layer 
- View : stateful and stateless widgets that render the UI
- View-Model : A class responsible for handling a specific view's (yes the second aspect) business logic and state. 

###### The more important concepts of MVVM are as such:
- ***Service***: A class responsible for raising the business logic and state up a level that lets the view models share the same logic and state.
- ***Repository***: A class responsible for retrieving data outside our app. Part of the "model" layer.
- ***Abstraction***: A class that is a wrapper around an external resources. Part of the "model" layer.

![[Pasted image 20260106163415.png]]

###### Rules to be followed when using the MVVM architecture
1. Views should not use the services directly. Instead use the viewmodels only.
2. View should not contain any logics (as much as possible ). Put the logic in the viewmodel as much as possible.
3. Viewmodels should not use other view models. Make a Service and share it among the viewmodels.
4. Viewmodels should not have access to BuildContext. That is to be done by view.
5. Dependencies should always be injected (added) through the constructor.

# View Viewmodel

###### ***View:*** Basically the UI of the page or UI of a custom widget of the page.
###### ***Pages:*** The UI of an entire single screen. Should only use one viewmodel to handle all if not most of the logic of the page. 

### Widgets
While the rule of keeping all the in the viewmodel is valid for most situations, it isnt when it comes to some widgets. For example: a button with a loading state. If it is used 10 times in the application theres no point in maintaining 10 separate states according to the rules. 

So we follow a simple rule:
**If the state is fully contained, keep it within the widget.**
This makes easy to reuse the widget across the application without the need to create unnecessary additional separate states. 

#### 3 ways to store states:
1. **Widget state:** Stored in the widget only when the state is fully contained within that particular widget. 
2. **Page state:** Stored within the page(ViewModel)
3. **App state**: Stored in a 'service' and only exposed to the page through a ViewModel.

#### Every page must have exactly one ViewModel
there are 3 purposes of the ViewModel:
1. Remove business logic from View.
2. Make all business logic fully testable
3. Making the state uni-directional (flowing from a 'higher' class down to the view)

# App State
## Service
Similar to a ViewModel, but is used for multiple views/ app-wide. Not exclusive for a single view/page.

But it eventually needs to be passed to the app so it has to pass through a viewmodel anyways. This is done by injecting it into the viewmodel's constructor.

So anytime we have multiple views dependent on a single state, all the viewmodels should depend on that service.

All properties should be ValueNotifiers and ViewModel will expose getters on them for the View to react.

###### ***ViewModel before implementing service logic:***

```dart
class TodoPageViewModel {
  TodoPageViewModel()

  final ValueNotifier<List<Todo>> todosNotifier = ValueNotifier([]);

  void add(Todo todo) {
    todosNotifier.value = [...todosNotifier.value, todo];
  }
}
```

###### ***ViewModel after implementing service logic:***

```dart
class TodoService {
  final ValueNotifier<List<Todo>> todosNotifier = ValueNotifier([]);

  void add(Todo todo) {
    todosNotifier.value = [...todosNotifier.value, todo];
  }
}

class TodoPageViewModel {
  TodoPageViewModel({required TodoService todoService}) : _todoService = todoService;

  final TodoService _todoService;

  get ValueNotifier<List<Todo>> todosNotifier => _todoService.todosNotifier;

  void add(Todo todo) {
    _todoService.add(todo);
  }
}
```

Now the `TodoService` handles the notifier rather than the ViewModel, and the ViewModel would expose that to the view.

This is useful for a couple of reasons:
1. In some cases, multiple views share a single state
2. You can encapsulate a shared state, and the ViewModels can focus on the view.
3. You avoid circular dependencies, which could lead to unexpected issues.

## Dependency Injection

***Dependencies should always be injected through the constructor***

Eg:
```dart
class MyService {}

class MyViewModel {
  MyViewModel({required MyService myService}) : _myService = myService;

  final MyService _myService
}

final myViewModel = MyViewModel(myService: WeNeedTheServiceInstanceHere);
```

### Pass  dependencies
Multiple ways to instantiate a new instance and use it throughout the app

Example: **Inherited Widget** 
Create an instance, insert in the Inherited Widget. Any widget below the Inherited Widget in the widget tree can access the instance.

other approach: **Service Locator**

#### Service Locator
- Just another fancy way of using a `Map` data structure.
- Used to create and reference instance of classes.
- To make custom service Locator, gotta use `GetIt` package.


# Model: The Data Layer
**Part of the architecture that handles the data that is to be stored in the app**

Part of the data layer that interacts with our application are called Repositories.
- Theyre the abstraction layer between the app and external resources (like databases, HTTP requests, file systems, caches, packages etc)
- Provide a consistent interface for the app to interact with various data sources, regardless of their nature. 
- This is useful as it allows the app to:
	- fetch data from multiple sources and use the best.
	- switch between diff sources without changing app logic.
	- combine data from various sources into single interface.
		`For example, a repository might fetch user data from a local database, retrieve their recent activity from a REST API, and check their subscription status from a third-party service - all while presenting a unified interface to the rest of the application.`

## The Abstraction Layer
### Just a fancy class that is used for wrapping external packages/dependencies

`For example, suppose you use a package to check your phone's battery. In that case, this allows you to control that dependency's interface more easily and allows for easier testing.`

```dart
class BatteryAbstraction {
  /// wrap the interface of the native battery package or native calls
}
```

```dart
class HomeViewModel {
  HomeViewModel({required BatteryAbstraction batteryAbstraction}) : _batteryAbstraction = batteryAbstraction;

  final BatteryAbstraction _batteryAbstraction;

  // now you can use the abstraction as you wish are during tests it's very straightforward to fake or mock out the abstraction class.
}
```

### Important closing points:
- you don't have to use all of this for a simple todo app.
- The level of abstraction that we covered here is similar to what big companies use
	- Coz they **need** things abstracted this way for teams to work efficiently together. And testing.
	
- If you use a camera package and only need to take a picture, with a 'CameraAbstraction', can only expose the picture-taking feature. 
	- Then, during testing, you only need to mock this one feature instead of the entire camera package functionality.

- In the future, if you need more functionality, add it to the abstraction, update your tests, and your code is simple and easy to review for the team.

### How to handle Remote Data

***Always make sure to have some form of local database in-device.***

`For example:`
- Video call application can get away without loading in case of no-connection.
- but a todo list or notes app or a game or fitness tracker cannot get away coz they should work despite no-connection.

We tackle this by having two data sources, one local and one remote. Then use the Repository to choose which data source to call.

