How to Use Redux Toolkit & RTK Query in React
Managing states in the entire application became very complicated and tough as the requirements of websites increased. In a React project, Context API and useReducer hook are not sufficient always. To handle such complex states, Redux is one of the most necessary tools in almost every React project. In this blog, I’ll demonstrate how can you use Redux Toolkit and RTK Query in a React Project.

Here are two useful links: Project Redux Toolkit & RTK Query || Boilerplate

What is Redux
Redux is a predictable state container for JavaScript applications. One major benefit of using Redux is that it allows centralizing the state of JS applications in a single store. This makes it easier to reason about the state of applications and helps to avoid the complexity of managing states across different components of applications.

Configure Store
Firstly, inside the src/app/store.js file, configure your store like this:

import { configureStore } from "@reduxjs/toolkit";

export const store = configureStore({
    reducer: {},
})
After that, you must provide the store to the whole React application by the following-

import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import { Provider } from "react-redux";
import { store } from "./app/store.js"; //Configured Store in store.js file

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}> // Provide store in the entire App
      <App />
    </Provider>
  </React.StrictMode>
);
Write your first slice
Now you have to create a slice and write the name of this slice, an initialState and reducers.

import { createSlice } from "@reduxjs/toolkit"

const initialState = {
    name: "",
    email: "",
    role: "",
}

const authSlice = createSlice({
    name: 'auth',
    initialState,
    reducers: {}
})
The reducers will help to change states which can be done anywhere in the application. Besides that don’t forget to export the slice and reducers.

reducers: {
        signInReducer: (state, { payload }) => {
            state.email = payload.email,
            state.name = payload.name,
            state.role = payload.role
        },
        signOutReducer: (state) => {
            state.email = "",
        }
    },

export const { signInReducer, signOutReducer } = authSlice.actions
export default authSlice.reducer
Import and provide the slice in the store.

import authSlice from "../features/auth/authSlice";

export const store = configureStore({
    reducer: {
        auth: authSlice,
    }
})
Now, you can import and use the reducers in other files like in src/page/Login.js file

import React from 'react'
import { signInReducer } from "../../features/auth/authSlice";
import { useDispatch } from "react-redux";

export default function Login() {
const handleSignIn = () => {
    dispatch(signInReducer())
  }

  return (
    <div>Sign In Page
        <button onClick={handleSignIn}> Sign In </button>
    </div>
  )
}
With all these, you can use the states in the entire application which you’ve provided in the slice. Here in src/page/Home.js -

import React from 'react'
import { useSelector } from "react-redux";

export default function Home() {
const auth = useSelector(state => state.auth)

  return (
    <div>
        User Name: {auth.name}
    </div>
  )
}
Handling Async Function
To handle async functions, you have to use extraReducers function in the slice and createAsyncThunk higher-order function. You also can handle pending, fulfilled and rejected cases like the following-

import { createAsyncThunk, createSlice } from "@reduxjs/toolkit"

const initialState = {
    name: "",
    email: "",
    role: "",
    isLoading: false,
    isError: false,
    isSuccess: false,
    error: ""
}

export const signUpThunk = createAsyncThunk("auth/signUp", 
    async ({ email, password }) => {
    const data = await thunkFunction(email, password)
    return data.user.email
})

const authSlice = createSlice({
    name: 'auth',
    initialState,
    reducers: {
     // ... Our Reducers
    },
    extraReducers: (builder) => {
       builder.addCase(signUpThunk.pending, (state) => {
                state.isLoading = true
                state.isError = false
                state.error = ''
            }).addCase(signUpThunk.fulfilled, (state, action) => {
                state.isLoading = false
                state.user.email = action.payload
            }).addCase(signUpThunk.rejected, (state, action) => {
                state.user.email = ''
                state.isError = true
                state.isLoading = false
                state.error = action.error
            })
}
Here is the signUpThunk function will take two parameters the name of the function as string and an HoF function. This function can be called any other file.

import React from 'react'
import { signUpThunk } from "../features/auth/authSlice";

const data = { email: "riyad@gmail.com", password: "1234" }

export default function Signup() {

const onSubmit = (data) => {
    dispatch(signUpThunk({ email: data.email, password: data.password }))
    reset()
  };

  return (
    <div>Signup Page
        <button onClick={() => onSubmit(data)}> Sign In </button>
    </div>
  )
}
RTK Query
In this section, I’ll show you how can you implement RTK Query.

First, you need to create API using createAPI function in src/features/api/apiSlice.js file

import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'

const URL = "https://demourl.com"

export const apiSlice = createApi({
    reducerPath: "api", // Give it a name
    baseQuery: fetchBaseQuery({ baseUrl: URL }), // Define Base URL
    endpoints: (build) => ({
        products: build.query({
            query: (data) => ({
                url: '/products',
                method: 'GET',
                body: data
            }),
        })
    })
})

export const { useProductQuery } = authSlice
Before you start using the useProductQuery hook, you must provide it in the store. Don’t forget to contact the middleware also.

import { configureStore } from "@reduxjs/toolkit";
import { apiSlice } from "../features/api/apiSlice";
import apiSlice from "../features/api/apiSlice";

export const store = configureStore({
    reducer: {
        auth: authSlice,
        [apiSlice.reducerPath]: apiSlice.reducer
    },
    middleware: (getDefaultMiddleware) => 
                getDefaultMiddleware().concat(apiSlice.middleware)
})
Now we can use the useProductQuery hook anywhere.

import React from 'react'
import { useRegistrationMutation } from "../../features/api/apiSlice";

export default function Products() {
const { data, isError, isFetching, isSuccess,error } = useProductQuery()

  return (
    <div>Products Page
      {data.products.map( product => <h4> {product.name} </h4> )}
    </div>
  )
}
You can handle the async result by showing a simple toast:

import { Toaster } from "react-hot-toast";
import { Toaster } from "react-hot-toast";
import React, { useEffect, useState } from "react";
import { useRegistrationMutation } from "../../features/api/apiSlice";

export default function Products() {
const { data, isError, isFetching, isSuccess,error } = useProductQuery()

useEffect(() => {
    if (isFetching) {
      toast.loading("Processing...", {id: 'pending', duration: 2000})
    }
    if (isSuccess) {
      toast.success("Successfully Register your account", {id: 'success'})
    }
    if (isError) {
      toast.error("Failed to Register your account", {id: 'fail'})
    }
  }, [isFetching, isSuccess, isError, error, data])

  return (
    <div>Products Page
      <Toaster />
      {data.products.map( product => <h4> {product.name} </h4> )}
    </div>
  )
}
Redux
Redux Thunk
Redux Toolkit
Rtk Query
