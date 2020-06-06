---
layout: post
title: "Getting Started with React Hooks in SPFx"
date: 2020-05-03
---

Introduction
------------

In the latest versions of SPFx, React 16.8 is supported which means we can take advantage of React Hooks for working with our React components. Hooks are functions that let you use state and other React lifecycle features in functional components without writing a class. This transition from class based components to functional components might seem daunting at first to people experienced with creating React class components. This article aims to serve as a starting guide to make this transition.

Sample Solution
---------------

![React List Items SPFx Web Parts](https://media-exp1.licdn.com/dms/image/C5612AQGg8v2KuTL04Q/article-inline_image-shrink_1500_2232/0?e=1596672000&v=beta&t=Aq6qrJF-imBmo1TOSF0MtoYgu_PxIokBPIZVtSbxoCU)

We will have a look at a sample solution to see how a SharePoint list's items can be fetched and rendered using one web part that uses React class components and another one that uses functional components and built-in *useState*, *useEffect* and *useContext* Hooks. Each web part renders a main React component that has a [PnP List Picker React control](https://sharepoint.github.io/sp-dev-fx-controls-react/controls/ListPicker) and a child *ListItemsViewer* React component that lists the list items whenever the List in List Picker is updated.

> The entire source code for this sample can be found at <https://github.com/aakashbhardwaj619/react-list-items>.

### Class Component (*ListItemsClassic*)

[![ListItemsClassic.tsx](https://media-exp1.licdn.com/dms/image/C5612AQFNUU9RCvYkhQ/article-inline_image-shrink_1500_2232/0?e=1596672000&v=beta&t=jRhdfV7YbaYZbu60RbNBoUfmJ2tBtQU91hpmdbvwFX4)](http://github.com/aakashbhardwaj619/react-list-items/blob/master/src/webparts/listItemsClassic/components/ListItemsClassic.tsx)

### Class Component (*ListItemsViewer*)

[![ListItemsViewer.tsx](https://media-exp1.licdn.com/dms/image/C5612AQFI-eMyyGMMQw/article-inline_image-shrink_1500_2232/0?e=1596672000&v=beta&t=AA-GKCmn34hI2RECjWHNy46R402UKx8lGHbPN2ZYWug)](http://github.com/aakashbhardwaj619/react-list-items/blob/master/src/webparts/listItemsClassic/components/ListItemsViewer.tsx)

### Functional Component (*ListItemsHooks*)

[![ListItemsHooks.tsx](https://media-exp1.licdn.com/dms/image/C5612AQFqyUp6AUbl5w/article-inline_image-shrink_1500_2232/0?e=1596672000&v=beta&t=eHzSO5h3XbzR-ZDfRH_cYecfyL__Ix7pbY8JqKRm4C8)](http://github.com/aakashbhardwaj619/react-list-items/blob/master/src/webparts/listItemsHooks/components/ListItemsHooks.tsx)

Functional Component (*ListItemsViewer*)

[![ListItemsViewer.tsx](https://media-exp1.licdn.com/dms/image/C5612AQEkuXlvhJkeRQ/article-inline_image-shrink_1500_2232/0?e=1596672000&v=beta&t=g_sIfL2hLu2tpVkoE4w3sbCMbdNSCf8VgCfoe-E8Dpk)](http://github.com/aakashbhardwaj619/react-list-items/blob/master/src/webparts/listItemsHooks/components/ListItemsViewer.tsx)

useState
--------

In the class based *ListItemsClassic* component we have one state variable *lists* that gets its initial value set in the constructor and gets updated using *this.setstate* method when a list is selected in the list picker.

In the functional *ListItemsHooks *component instead of a constructor to initialise the value of state variable *lists* we are using the *useState* hook for initialising and updating this state variable.

The [*useState*](https://reactjs.org/docs/hooks-state.html)hook declares a state variable and returns it along with a method to update its value. The argument passed to this hook becomes the initial value of the state variable and it returns an array that consists of the state variable along with its update method. The names for the variable and the method can be anything. Multiple variables can be declared using the below format and calling their update methods will replace the variable unlike *this.setState* that merges the updated values with older ones.

```
const [varName, varUpdateMethod] = useState(varInitialValue);
```

In the *ListItemsHooks *component *lists* is our state variable that is initialised with a blank value and *setLists* is the method that is called to update its value when the list is changed in List Picker control.

> All instances of this.state.lists are replaced with lists in ListItemsHooks

useContext
----------

In the classic web part we are passing down the *context* property down from our main *ListItemsClassic *component to our child *ListItemsViewer* component. For a small tree of components this has no issue. However if there is a big hierarchy of components and the last child component in the tree needs to access this context, it has to be passed through all of the intermediate components. This can get cumbersome and to avoid it the *useContext* hook comes into play.

In the *ListItemsHooks* component we create a context object using the [*createContext*](https://reactjs.org/docs/context.html#reactcreatecontext) API. By wrapping our HTML in this context object's Provider component and setting *props.context* as its value any nested components can now use this context value. Even the last component in the component tree can access this context object using the [*useContext*](https://reactjs.org/docs/hooks-reference.html#usecontext) hook without passing down the context as a prop through all intermediate components.

By creating an object using the *useContext* hook in the child component we get the context object that was declared in our main component and we can access its properties or methods without using *this.props*.

> All instances of this.props.context are replaced with context in ListItemsViewer

useEffect
---------

In the class based *ListItemsViewer* component we are using the *componentDidUpdate* method to fetch the SharePoint list items whenever the *lists* property is changed in the parent *ListItemsClassic *component.

In Function Components the *componentDidMount, componentDidUpdate and componentWillUnmount *lifecycle methods are replaced with the *useEffect* hook.

The [*useEffect*](https://reactjs.org/docs/hooks-effect.html) method accepts a method as its argument that gets called every time the component renders and updates. Passing an empty array as the second argument to *useEffect* imitates the behaviour of *componentDidMount* as it calls the argument method only on first render. However, if any prop or state variable is passed in this array, then the argument method would be called on first render and every time the passed in variable is updated. This is similar to comparing the *lists* updated property with previous property and then fetching the list items in class based *ListItemsViewer*'s *componentDidUpdate.*

```
useEffect(() => {

  //Run the method on render

}, [val]); //Only re run the method when val changes
```

Conclusion
----------

Thus we saw how we can make the transition from creating React class based components to functional components using hooks for developing new SPFx solutions. For in depth explanation of hooks' concepts the official documentation at <https://reactjs.org/docs/hooks-intro.html> can be referred to.

There are some other really useful built in hooks as well like *useReducer, useRef*, etc. (Refer <https://reactjs.org/docs/hooks-reference.html>). Other than the built in hooks, reusable custom hooks can also be created with their own independent state for complex functionality. A good example can be seen at <https://rabiawilliams.com/spfx/reusable-custom-hooks/>.
