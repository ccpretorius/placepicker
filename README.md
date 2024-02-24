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
