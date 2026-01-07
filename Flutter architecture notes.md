#flutter
# Architecture Overview
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

##### Widgets
While the rule of keeping all the in the viewmodel is valid for most situations, it isnt when it comes to some widgets. For example: a button with a loading state. If it is used 10 times in the application theres no point in maintaining 10 separate states according to the rules. 

So we follow a simple rule:
**If the state is fully contained, keep it within the widget.**

This makes sure that 