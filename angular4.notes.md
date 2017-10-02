# Angular 4 notes

## Main concepts

- **Component tree:** An hierachy of components, this is usually what the programmer defines, responsible to render the application and to change the application as the user interacts with it.

- **App component:** Is the component on top of the component root. It is the component that is initialialy called by angular runtime and then invoques the other components that made the application.

- **component:** is responsible for "drawing" an area in a browser page, and to handle the user events done on that area, in response to the events the "drawing" area is refreshed. A component can invoque other components to handle a sub-area of its "drawing" area, thus making a tree component.

    Conceptual purpose of a component:

             [Initial "drawing" area] --> [handle events] <--> [Update drawing area]

- **Template language** : A component, in most cases defines, how its "drawing" area are going to be filled using a template language in textual format. This language is similar to html but has a specific syntax for interacting with a angular component.

- **View tree** : this is a tree that is build automatically by the angular runtime from the application component tree. Every element, pure HTML or Angular component present in the template is represented in a tree of elements.

- **Dom tree**: This is the tree used by the browser to render the application and to interact with the user.

- **Angular tree dependecies**:

                                                 |             |  == renderization ==>   |
          [Component Tree]   == Compilation ==>  | [View tree] |                         | [Dom tree]
                                                 |             | <== user interaction == |


## Template syntax

## Component

- A component is defined by a class that declares:
    - A name to be used in the
    - A string with a template that references components
    - define a set of css styles applyed only to the component

There is the Component tree

# ViewContainerRef

ViewContainerRef is an object that injected