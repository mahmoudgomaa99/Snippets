# Using React-Query with Context API and AsyncStorage for Data Fetching and Persistence

In this document, we'll explore an architecture that combines `react-query`, the Context API, and `AsyncStorage` for efficient data fetching, state management, and local data persistence in a React application.

## Architecture Overview

This architecture integrates several key elements:

- `react-query`: Simplifies data fetching, caching, and state management.
- Context API: Allows components to share state seamlessly.
- `AsyncStorage`: Provides a mechanism for local data persistence on mobile platforms.

## Why Use This Architecture?

1. **Optimized Data Fetching**: `react-query` optimizes network requests with caching, background refetching, and data synchronization.

2. **Centralized State**: The Context API centralizes state management, enhancing code organization and reducing prop drilling.

3. **Local Data Persistence**: `AsyncStorage` enables data persistence across app sessions, ensuring user data is retained even after closing the app.

4. **Efficient and Clean Code**: Combining these technologies streamlines data management, simplifying components and reducing code duplication.

## Implementation Steps

1. **Install Dependencies**:

   - Install `react-query`, `@react-query/devtools`, `react-query/queryClientProvider`, and `@react-native-async-storage/async-storage`.

2. **Create Context and Reducer**:

   - Establish a context using the Context API to manage shared state.
   - Create a reducer to manage state changes, including fetched data.
   - Utilize `react-query` within the reducer to fetch data from the API.

3. **Persist Data with AsyncStorage**:

   - Enhance the reducer to update `AsyncStorage` when the state changes.

4. **Use Context in Components**:

   - Utilize the context in your components to access fetched data and handle loading/error states.

5. **Wrap Components with Context Provider**:

   - Surround components needing access to context with the Context Provider.
   - This ensures components can access shared state and the benefits of `react-query`.

6. **Utilize `QueryClientProvider`**:
   - Wrap your application with `QueryClientProvider` to provide `react-query`'s capabilities to components.

## Example Code

### MyContext.js

```jsx
import React, {createContext, useContext, useEffect, useReducer} from 'react';
import {useQuery, useQueryClient} from 'react-query';
import AsyncStorage from '@react-native-async-storage/async-storage';

const MyContext = createContext();

export const useMyContext = () => useContext(MyContext);

const fetchCountFromApi = async () => {
  const response = await fetch('API_URL_HERE'); // Replace with your API URL
  const data = await response.json();
  return data.count;
};

const initialState = {
  count: 0,
};

const reducer = async (state, action) => {
  let updatedState;
  switch (action.type) {
    case 'INCREMENT':
      updatedState = {...state, count: state.count + 1};
      break;
    case 'DECREMENT':
      updatedState = {...state, count: state.count - 1};
      break;
    case 'SET_INITIAL_COUNT':
      updatedState = {...state, count: action.payload};
      break;
    default:
      updatedState = state;
  }

  // Update AsyncStorage whenever state changes
  await AsyncStorage.setItem('count', JSON.stringify(updatedState.count));

  return updatedState;
};

export const MyContextProvider = ({children}) => {
  const queryClient = useQueryClient();

  const {
    data: count,
    isLoading,
    isError,
  } = useQuery('count', fetchCountFromApi);

  if (isError) {
    console.error('Error fetching count from API');
  }

  const [state, dispatch] = useReducer(reducer, initialState);

  // Load initial count value from AsyncStorage on component mount
  useEffect(() => {
    const loadCount = async () => {
      try {
        const storedCount = await AsyncStorage.getItem('count');
        if (storedCount !== null) {
          const parsedCount = JSON.parse(storedCount);
          dispatch({type: 'SET_INITIAL_COUNT', payload: parsedCount});
        }
      } catch (error) {
        console.error('Error loading count from AsyncStorage:', error);
      }
    };

    loadCount();
  }, []);

  const contextValue = {
    count,
    isLoading,
    state,
    dispatch,
  };

  return (
    <MyContext.Provider value={contextValue}>{children}</MyContext.Provider>
  );
};
```
