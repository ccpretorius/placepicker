# placepicker

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
