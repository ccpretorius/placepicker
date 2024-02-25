# placepicker

## Explaining the clickable button pics

The button within your Places component is rendered as part of each place item in the list of places that are either available to visit or have been picked by the user. Specifically, the button is displayed in the following context:

Within a List Item (<li>) of Each Place: Each place from the places array passed to the Places component is listed as an <li> element within an unordered list (<ul>). For each place, a button is created.

Button Content: The button includes:

An <img> element displaying the image of the place, with the src attribute set to the place's image URL (place.image.src) and the alt attribute set to a descriptive text (place.image.alt) for accessibility.
An <h3> element containing the title of the place (place.title).
Button's Functionality: When clicked, the button triggers the onSelectPlace function passed as a prop to the Places component. This function is called with the ID of the selected place (place.id) as its argument. Depending on which Places component instance the button is a part of, this could either add the place to the user's picked places (if the button is in the "Available Places" list) or initiate the process to remove a place from the picked places (if the button is in the "I'd like to visit ..." list).

Visual and Structural Context: The button is visually part of a larger list that represents either the places a user has picked to visit or the places that are available to be picked. This list is displayed under a heading (<h2>) that titles the section as either "I'd like to visit ..." or "Available Places", based on the title prop passed to the Places component.

Conditional Rendering Based on Places Availability: The rendering of the button (within its respective place item) is conditional:

If places.length is greater than 0, the list of places (and thus the buttons) is shown.
If places.length is 0, a fallback text paragraph (<p>), defined by the fallbackText prop, is displayed instead, indicating that there are no places to display.
In summary, the button from your Places component is displayed as part of a dynamic list of places, either under the "I'd like to visit ..." section for places the user has picked or under the "Available Places" section for places available to pick, each serving different functionalities upon being clicked.

## Definition of Side-effects

- Side-effects are "tasks" that do not impact the current component render cycle, although they are necessary for the component to render correctly

## Functionality in code

### Sort the pics according to the nearest to the furthest from the user

- For this you need to determine the location of the user
- use the navigator object of the browser

  - navigator.location.getCurrentPosition()
  - the user will be asked permission to determine his current position
  - it will take some time to determine the position so the getCurrentPosition() method takes a callback funtion that will be executed once the position has been fetched
  - import and run your sortPlaceByDistance function inside your root app (App.jsx)
  - the getCurrentPosition method will automatically receive the lon and lat object for the location of the user from the browser as a first argument to the intake of the callback
  - pass all the available places and this position object into the sortPlacesByDistance function
  - place this into a const so it can be used

  ```
  navigator.geolocation.getCurrentPosition((position) => {
    const sortedPlaces = sortPlacesByDistance(AVAILABLE_PLACES, position.coords.latitude, position, coords.longitude);
  });
  ```

### Side-effects

- this code is not directly related to what we render on the screen and is therefore a side effect
- this code does not finish instantly, but will be called back at some point in the future when the component most likely has finished its execution already
- the sortedPlace is not available at the first rendering of the component, because getting the user's location will take some time (the callback has not finished executing yet)
- therefore you need state to update the places prop (when the call back resolves into sortedPlaces that updates the state)
- we can now pass the availablePlaces in place of the raw DUMMY_DATA in the rendering of the places prop in the return section of the the component
- however, this would call an infinite loop, because the app would render every time the user's location is fetched and set the state again and again
- useEffect deals with this problem

### useEffect

#### How to use useEffect

- import useEffect
- execute it inside the component function just like any other hook
- useEffect takes two arguments, the first is a function that wraps your side-effect code. So move the navigation function inside your useEffect as a return value to the wrapping function
- the second argument is an array of dependencies of that effect function

```
useEffect(() => {
    navigator.geolocation.getCurrentPosition((position) => {
      const sortedPlaces = sortPlacesByDistance(AVAILABLE_PLACES, position.coords.latitude, position, coords.longitude);

      setAvailablePlaces(sortedPlaces);
    });
  }, []);
```

- React will execute this function after the first rendering. If any dependancies has been listed, useEffect will only render again if the dependancy values change. In this case where there are no dependancies specified, but only an empty array, react will never again after its first execution executes again. If no dependacy array has been specified at all, then you would be back to a loop because the component would continue rerendering.
- You can set some fallback text now as a places prop in the return statement to show while the places are being fetched. In your Places module this would render conditionally.
- It is considered bad practice to use useEffect() unnecessary. It triggers after the first component render session and can be unnecessary.
- For instance when you want to use localStorage() provided by the browser to remember your selections on a website.
  localStorage.setItem('selectedPlaces', JSON.stringify([]))
  - pass an identifyer in quotes
  - pass a second argument for the value, but convert this first into a string.
  - but you need to get the local storage id's first as taken in by handleSelectPlaces(id)
    localStorage.getItem('selectedPlaces') || '[]' which you need to convert back to a js object with the JSON.parse() method (see below)
  - also provide some fallback code in case no id's has been stored yet (in which case it would yield "undefined"), but now an empty array
  - upon rendering for the first time with nothing selected and stored yet, the code will attempt to parse and empty string which would not be possible. The empty array [] is therefore enclosed in quotations because JSON.parse() needs to render from a string.
- save this into a const storedIds that can be spread into an array with the new id placed in front
  const storedIds = JSON.parse(localStorage.getItem('selectedPlaces') || '[]');
  localStorage.setItem('selectedPlaces', JSON.stringify([id, ...storedIds]))
- add a check to see if the id is already stored. If it is not found then run the update code

```
  function handleSelectPlace(id) {
    setPickedPlaces((prevPickedPlaces) => {
      if (prevPickedPlaces.some((place) => place.id === id)) {
        return prevPickedPlaces;
      }
      const place = AVAILABLE_PLACES.find((place) => place.id === id);
      return [place, ...prevPickedPlaces];
    });

    const storedIds = JSON.parse(localStorage.getItem('selectedPlaces') || '[]');
    if (storedIds.indexOf(id) === -1) {
    localStorage.setItem('selectedPlaces', JSON.stringify([id, ...storedIds]))
    }
  }
```

- This whole storage mechanism is a side-effect. However, not every side-effect needs useEffect()
  You cannot use useEffect inside a nested function to wrap around this code or use it in if statements etc. They must be used directly inside the root level of the component
- This side-effect however, does not update any state when the handleSelectPlace onClick event is run. And even if it did update state it would not cause an infinite loop because it run only after its being clicked
- At this stage local storage wont work upon reload. It first needs to be loaded when the app reloads which it does not do yet. Nether does it update your selected places when items are deleted

Removing an item and storing it:

```
  function handleRemovePlace() {
    setPickedPlaces((prevPickedPlaces) => prevPickedPlaces.filter((place) => place.id !== selectedPlace.current));
    modal.current.close();

    const storedIds = JSON.parse(localStorage.getItem("selectedPlaces") || '[]');
    localStorage.setItem(
      "selectedPlaces",
      JSON.stringify(
        storedIds.filter((id) => {
          id !== selectedPlace.current; //selectedPlace is available as a Ref
        })
      )
    );
  }
```

- Loading an item from local storage when the App starts
  - This is an example where useEffect is redundant. It is running synchronously with the App because it does not neet to first fetch navigation data. It can therefore be moved outside the App and the useEffect hook should then not be used with it.
  - Update const [pickedPlaces, setPickedPlaces] = useState(storedPlaces); with storedPlaces instead of an empty array and your selected places should load upon reload

#### Showing a modal with useEffect

The direct manipulation of the DOM (dialog.current.showModal(); and dialog.current.close();) outside of the React lifecycle methods (like useEffect) can lead to unintended behaviors or errors.

This code will therefore not render correctly without using useEffect:

```
import { useRef } from "react";
import { createPortal } from "react-dom";

function Modal({ open, children }) {
  const dialog = useRef();

  if (open) {
    dialog.current.showModal();
  } else {
    dialog.current.close();
  }

  return createPortal(
    <dialog className="modal" ref={dialog}>
      {children}
    </dialog>,
    document.getElementById("modal")
  );
}

export default Modal;

```

This code is a type of side-effect because even though it does have a direct effect on the UI it does not have a direct impact on the jsx code inside the returned createPortal. So it is not directly related to the component's render cycle.

```
if (open) {
dialog.current.showModal();
} else {
dialog.current.close();
}
```

So this code needs to be moved into a useEffect hook.

Here's why useEffect is necessary in this context:

1. React Lifecycle and DOM Manipulation:
   React's reconciliation process means that direct DOM manipulations might not align with React's virtual DOM updates. Manipulating DOM elements directly in the component body (outside of effects or lifecycle methods) can lead to inconsistencies because these manipulations can be overridden or missed during React's DOM updates.

2. Ref Initialisation:
   When your component first renders, dialog.current is null because the assignment to the ref happens after the component's output has been committed to the DOM. (At this stage the connection between the dialog and the ref in the CreatePortal has not been established.) Attempting to call .showModal() or .close() directly in the component body will throw an error because dialog.current is not yet assigned.

3. Conditional Logic Based on Props:
   Using useEffect allows you to correctly respond to changes in props or state. By placing your open/close logic inside a useEffect with open as a dependency, you ensure that:

The modal's open/close state is correctly managed in response to changes in the open prop.
DOM manipulations occur after the component has been mounted and dialog.current has been assigned.
Correct Implementation Using useEffect:

```
import { useRef, useEffect } from "react";
import { createPortal } from "react-dom";

function Modal({ open, children }) {
  const dialog = useRef();

  useEffect(() => {
    if (open) {
      dialog.current.showModal();
    } else {
      dialog.current.close();
    }
  }, [open]); // Dependency array with 'open' ensures effect runs when 'open' changes.

  return createPortal(
    <dialog className="modal" ref={dialog}>
      {children}
    </dialog>,
    document.getElementById("modal-root") // Ensure this is 'modal-root' or the correct ID.
  );
}

export default Modal;
```

The useEffect hook listens for changes to the open prop. When open changes, the effect will run (after the initial run of the component function), safely applying the appropriate DOM manipulation (showModal or close) after the component has mounted and dialog.current is guaranteed to be assigned. It safely established the connection between the dialog and the ref.

useEffect here does not prevent an infinite loop, it synchronise a value (the open prop) to a DOM API or to a certain behaviour.

This approach leverages React's lifecycle to ensure your DOM manipulations are both safe and in sync with React's updates, preventing errors and ensuring the expected behavior of your modal component.

At this point though you will still get a warning when the page renders. This is because all the dependancies required has not been listed in the dependancies array of the useEffect hook.

Other than the previous useEffect examples, this if..code here has dependancies

Depandancies is any value that causes the component function to execute again (like with props and state) if it is used inside useEffect(). Anything build into the browser (like objects and methods like we've seen with navigator) is not concidered dependancies. useEffect() only cares about dependancies that would cause the component function to execute again, because the useEffect() function should run whenever the component function executed, if one of its dependancies changed.

The open prop value will change in this application so it is a dependancy. At this stage you can click on the pics in the application but nothing happens. The modal does not open, because the effect function does not run again - which means that showModal is not called. Therefor you need to add "open" as a dependancy. Whenever the modal function has run or the value of the open prop change the effect function will run now.

Fixing a Small Bug
The <dialog> element can also be closed by pressing the ESC key on the keyboard. In that case, the dialog will disappear but the state passed to the open prop (i.e., the modalIsOpen state) will not be set to false.

Therefore, the modal can't be opened again (because modalIsOpen still is true - the UI basically now is not in sync with the state anymore).

To fix this issue, we must listen to the modal being closed by adding the built-in onClose prop to the <dialog>. The event is then "forwarded" to the App component by accepting a custom onClose prop on the Modal component.

The Modal component therefore should look like this:

import { useRef, useEffect } from 'react';
import { createPortal } from 'react-dom';

function Modal({ open, children, onClose }) {
const dialog = useRef();

useEffect(() => {
if (open) {
dialog.current.showModal();
} else {
dialog.current.close();
}
}, [open]);

return createPortal(

<dialog className="modal" ref={dialog} onClose={onClose}>
{children}
</dialog>,
document.getElementById('modal')
);
}

export default Modal;
In the App component, we can now set the handleStopRemovePlace function as a value for the onClose prop on the <Modal> component:

<Modal open={modalIsOpen} onClose={handleStopRemovePlace}>
  <DeleteConfirmation
    onCancel={handleStopRemovePlace}
    onConfirm={handleRemovePlace}
  />
</Modal>

## Setting a timeout for the modal

- setTimeout() is built into the browser. You want to set this in your DeleteConfrimation component
- the first argument is a function and the second one is a duration in milliseconds
- the function will be executed onece the duration expired
  setTimeout(() => {
  onConfirm();
  }, 3000);
  onConfirm will only be executed 3 seconds after the component is rendered
- In this situation the setTimeout in DeleteConfirmation component is automatically rendered in the App component as well as the Modal which is rendered even though not visible in the DOM. So because it is part of the App component it will always render when the App renders for the first time
- You can handle this by setting a condition like this in the App Component in the Modal when the Modal opens
  <Modal open={modalIsOpen} onClose={handleStopRemovePlace}>
  {modalIsOpen && <DeleteConfirmation onCancel={handleStopRemovePlace} onConfirm={handleRemovePlace} />}
  </Modal>
- The problem now is that if you select a picutre and escape from the modal, the timer remains activated and would still delete the picture as it keeps on going behind the scene after being activated by the first render.
- Note that the setTimout() code is again a side effect as it is not directly related to the output of the jsx code.
- So, how do one stop this code once the component disappears? The problem here is not setting the timer, but cleaning it up when the Modal function disappears. You can return a cleanup function from within the useEffect function for this purpose.
  \_ So this is how the useEffect function looks with the setTimeout method inside the DeleteConfirmation component

```
export default function DeleteConfirmation({ onConfirm, onCancel }) {

  useEffect(() => {
  setTimeout(() => {
    onConfirm();
  }, 3000)},[]);

  return (
    <div id="delete-confirmation">
      <h2>Are you sure?</h2>
      <p>Do you really want to remove this place?</p>
      <div id="confirmation-actions">
        <button onClick={onCancel} className="button-text">
          No
        </button>
        <button onClick={onConfirm} className="button">
          Yes
        </button>
      </div>
    </div>
  );
}
```

- This is how you add the cleanup function:

```
  useEffect(() => {
    const timer = setTimeout(() => {
      onConfirm();
    }, 3000);

    return () => {
      clearTimeout(timer);
    };
  }, []);
```

This will now remove the timer whenever the Modal is removed from the DOM

- You will be warned to add the onConfirm prop at this stage. However, the problem of the timer not stopping is now removed. Note that it does not run initially when the component runs the first time
- When you run props or state values inside your useEffect function, you should add them as dependancies. So add onConfirm as a dependancy
- When you add functions as dependancies there is a problem to anticipate, because it creates the danger of an infinite loop
- Every time react rerenders a function it creates a new object (functions are concidered to be objects). Even if the two objects have precisely the same layout, it will concider it to be two separate values, therefore it will rerender everytime it compares the two and an infinite loop is created
- The onConfirm prop is linked to the handleRemovePlace function which will recreate every time
- In our project we dont have this problem because when a state update is triggered here with onConfirm it sets setModalIsOpen to false which results in the DeleteConfirmation to be removed from the DOM which includes the onConfirm prop
- In the event of such a finite loop being produced (like when you temporarily disable or comment out setModalIsOpen in the App).
- There is another hook that can deal with this problem and which can be used whether or whether not the element is removed from the DOM as we do here

### useCallback is used to ensure that the function is not created all the time

- In the handeRemovePlace function we have commented out the setModalIsOpen(false) state setting which means the function will result in a resetting of the setTimeout function in an infinite loop. To prevent this wrap the whole funcion inside a useCallback function.
- Set it as a first argument to this useCallback function. It also takes a second argument that should be an array of dependancies
- return the useCallback as a function that would not be recreated whenever the component function is executed again.
- Any dependancies wrapped inside this useCallback function should be added into the array dependancy list of the useCallback

```
const handleRemovePlace = useCallback(function handleRemovePlace() {
    setPickedPlaces((prevPickedPlaces) => prevPickedPlaces.filter((place) => place.id !== selectedPlace.current));

    setModalIsOpen(false);

    const storedIds = JSON.parse(localStorage.getItem("selectedPlaces") || "[]");
    localStorage.setItem("selectedPlaces", JSON.stringify(storedIds.filter((id) => id !== selectedPlace.current)));
  }, []);
```

Good practice is to use useCallback when you pass functions as dependancies to useEffect

### Progressbar

- In order to show the user that the modal is on a timer (after 3 seconds the selected pic will disappear), you can set a progress bar
- Go to the DeleteConfirmation component and use the builtin progress element just before the last closing div
- You will obviously need state because a progress bar gets reset continuously. The more often you update it the smoother it will be. For this you can use another function built into the browser: setInterval(). It defines a function that will be executed every couple of milliseconds. You can pass the old state snapshot and the new remainning time which is that prevTime -10. Now you can set the value prop on the progress bar equal to remaining time. You also need to set a max prop so that the fill status can also be calculated by the browser

```
import { useEffect, useState } from "react";

const TIMER = 3000;

export default function DeleteConfirmation({ onConfirm, onCancel }) {
  const [remainingTime, setRemainingTime] = useState(TIMER);

  setInterval(() => {
    setRemainingTime((prevTime) => prevTime - 10);
  }, 10);

  useEffect(() => {
    const timer = setTimeout(() => {
      onConfirm();
    }, 3000);

    return () => {
      clearTimeout(timer);
    };
  }, [onConfirm]);

  return (
    <div id="delete-confirmation">
      <h2>Are you sure?</h2>
      <p>Do you really want to remove this place?</p>
      <div id="confirmation-actions">
        <button onClick={onCancel} className="button-text">
          No
        </button>
        <button onClick={onConfirm} className="button">
          Yes
        </button>
      </div>
      <progress value={remainingTime} max={TIMER} />
    </div>
  );
}
```

- However, at this stage the timer expires impromptu and three seconds later the pic deletes. This is because we have created an infite loop here. To prevent this we need to wrap the setInterval function with useEffect and the dependancies array.
- The bar now works properly, but the interval keeps on repeating. So you need to return a cleanup function that will be executed by react and store a reference to this interval in a const or var. Now you can pass the clearInterval function into the function. clearInterval() is provided by the browser

```
useEffect(() => {
    setInterval(() => {
      console.log("INTERVAL");
      setRemainingTime((prevTime) => prevTime - 10);
    }, 10);
  }, []);
```

Final code:

```
  useEffect(() => {
    const interval = setInterval(() => {
      console.log("INTERVAL");
      setRemainingTime((prevTime) => prevTime - 10);
    }, 10);

    return () => {
      clearInterval(interval);
    };
  }, []);
```

### Optimising the code

- At this stage when the interval rerenders every 10 miliseconds, so does the whole DeleteConfirmation function as well as all the jsx code. To optimise it you can outsource the part that deals with the progress bar to a new component so that it runs only inside that component and does not rerender then eveything inside the DeleteConfirmation function all the time
- Cleaned up DeleteConfirmation component:
  import { useEffect } from "react";

import ProgressBar from "./ProgressBar.jsx";

const TIMER = 3000;

export default function DeleteConfirmation({ onConfirm, onCancel }) {
useEffect(() => {
const timer = setTimeout(() => {
onConfirm();
}, 3000);

    return () => {
      clearTimeout(timer);
    };

}, [onConfirm]);

return (

<div id="delete-confirmation">
<h2>Are you sure?</h2>
<p>Do you really want to remove this place?</p>
<div id="confirmation-actions">
<button onClick={onCancel} className="button-text">
No
</button>
<button onClick={onConfirm} className="button">
Yes
</button>
</div>
<ProgressBar timer={TIMER} />
</div>
);
}

Sourced out ProgressBar component:
import { useState, useEffect } from "react";

export default function ProgressBar({ timer }) {
const [remainingTime, setRemainingTime] = useState(timer);

useEffect(() => {
const interval = setInterval(() => {
console.log("INTERVAL");
setRemainingTime((prevTime) => prevTime - 10);
}, 10);

    return () => {
      clearInterval(interval);
    };

}, []);

return <progress value={remainingTime} max={timer} />;
}
