---
banner: https://thoughtfocus.com/wp-content/uploads/2025/03/flutter_banner-scaled-1.jpg
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

# Service
Similar to a ViewModel, but is used for multiple views/ app-wide. Not exclusive for a single view/page.

But it eventually needs to be passed to the app so it has to pass through a viewmodel anyways. This is done by injecting it into the viewmodel's constructor.

So anytime we have multiple views dependent on a single state, all the viewmodels should depend on that service.

All properties should be ValueNotifiers and ViewModel will expose getters on them for the View to react.

***Before:***

```
class TodoPageViewModel {
  TodoPageViewModel()

  final ValueNotifier<List<Todo>> todosNotifier = ValueNotifier([]);

  void add(Todo todo) {
    todosNotifier.value = [...todosNotifier.value, todo];
  }
}
```

***After:***

```
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


