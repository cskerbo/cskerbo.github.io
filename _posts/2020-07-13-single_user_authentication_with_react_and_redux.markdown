---
layout: post
title:      "Single User Authentication with React and Redux"
date:       2020-07-13 20:37:51 +0000
permalink:  single_user_authentication_with_react_and_redux
---


I recently had a unique need for a project in which I needed secure user authentication - for a single user. The use case was a personal website in which the user could access an admin page. The admin page would allow the user to add content to the site by utilizing some backend CRUD functionality.

I found it difficult to find informaiton on the best method to implement this unique scenarion while using React and Redux, so I hope that in sharing my implementation it's helpful to others with the same need.

*Note*: I am using a Rails API as a backend, but I won't dive into that here. My setup is straight-forward, and I am using a User Controller to authenticate users and assing a JWT to local storage once authenticated. 

## Creating userActions

As stated, this was a very specific use-case, so my actions needed were different than usual. I only need to:

* Login
* Authorize
* Logout

I had no need to create addtional users, so I opted to load the database with the sole user instead of implementing code to create users - it seemed wasteful in this particular instance.

```
export const userLoginFetch = user => {
    return dispatch => {
      return fetch("/login", {
        method: "POST",
        headers: {
          'Content-Type': 'application/json',
          Accept: 'application/json',
        },
        body: JSON.stringify({user})
      })
        .then(resp => resp.json())
        .then(data => {
          if (data.message) {
            alert(data.message)
          } else {
            localStorage.setItem("token", data.jwt)
            dispatch(loginUser(data.user))
          }
        })
    }
  }

  export const authorizeUser = () => {
    return dispatch => {
      const token = localStorage.token;
      if (token) {
        return fetch("/user_is_authed", {
          method: "GET",
          headers: {
            'Content-Type': 'application/json',
            Accept: 'application/json',
            'Authorization': `Bearer ${token}`
          }
        })
          .then(resp => resp.json())
          .then(data => {
            if (data.status === 200) {
              console.log(data)
              dispatch(loginUser(data.user))
            } else {
              console.log(data)
              localStorage.removeItem("token")
            }
          })
      }
    }
  }
  
  const loginUser = userObj => ({
      type: 'LOGIN_USER',
      payload: userObj
  })

  export const logoutUser = () => ({
    type: 'LOGOUT_USER',
  })
```

The above Redux actions, when called, connect to my Rails API which then runs through the user authentication process.

## Creating the Login Container

I created a basic login form with it's own state. The state is then dispatched as is typical when utilizing Redux. You can see how when this data is dispatched, it carries over to the actions I outlined above.

```

import React, { Component } from "react";
import {userLoginFetch} from '../actions/userActions';
import {connect} from 'react-redux';
import { Form, Button } from 'react-bootstrap';


class LoginContainer extends Component {

    state = {
        username: "",
        password: ""
      }

    handleChange = event => {
        this.setState({
        [  event.target.name]: event.target.value
        });
    }

    handleSubmit = event => {
        event.preventDefault()
        this.props.userLoginFetch(this.state)
    }

        
    render() {
    return(
        <div className='row' style={{justifyContent: 'center'}}>
            <div className='col-md-4'>
            <h1>Log In</h1>
            <Form onSubmit={this.handleSubmit}>
                <Form.Group>
                    <Form.Label>Username</Form.Label>
                    <Form.Control required value={this.state.username} onChange={this.handleChange} name='username' type="text" placeholder="username"/>
                </Form.Group>
                <Form.Group>
                    <Form.Label>Password</Form.Label>
                    <Form.Control required value={this.state.password} onChange={this.handleChange} name='password' type="password" placeholder="password"/>
                </Form.Group>
                <Button type="submit">Submit</Button>
            </Form>
        </div>
        </div>
    )
}
} 

const mapDispatchToProps = dispatch => ({
    userLoginFetch: userInfo => dispatch(userLoginFetch(userInfo))
  })

export default connect(null, mapDispatchToProps)(LoginContainer);
```

## It's that easy.
With these two solutions in place, you already have layed the groundwork for user authentication. From here, you can:

* Add logic to any part of your application that displays either the login container (above), or the content you'd like to show (if the user is authenticated).
* Add user creation, updating, and deletion logic as needed.
