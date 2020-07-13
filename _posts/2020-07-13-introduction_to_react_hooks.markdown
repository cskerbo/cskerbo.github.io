---
layout: post
title:      "Introduction to React Hooks"
date:       2020-07-13 19:56:19 +0000
permalink:  introduction_to_react_hooks
---


### What are hooks?

Hooks are a fairly new addition to React. They were added in 2019 and allow you to create components that have a state or lifecycle methods without building it as a Class.

Simply put: *they give functional components state*, which was previously not possible.

### Why use hooks?

* Hooks let you utilize state, just like you would in a class, but with less lines of code. This means a cleaner project, easier to understand code, and less complexity.
* Hooks provide better performance. While this is very minimal in smaller applications, if you're working in a large application with large componenet trees - this becomes very important.

### Let's take a look at utilizing state prior to hooks

Say we want to add a simple 'like button' to our blog post. We know in this case we will need to have a state in place, something that will track the number of times a post has been liked. In this case, we would have to create a class like so:

```
  class Counter extends Component {
    constructor(props) {
        super(props);
    
        this.state = {
          likes: 0
        };
    };

    handleOnClick = () => {
        this.setState((prevState) => ({
            likes: prevState.likes + 1
        }))
    }
    
    render() {
      return (
      <div >
          <Button onClick={this.handleOnClick}>Like</Button>
          <p>{this.state.likes} Likes</p>
          </div>
      )
    }
  }

export default Counter;
```

Ok, doesn't seem to bad right? Pretty typical and straight-forward. I certainly wouldn't call it messy.

### Now let's see how hooks can help

Consider the same need we had above, but now let's implement hooks.

```
  export default function Counter() {
    const [likes, setLikes] = useState(0);

      return (
      <div >
          <Button onClick={() => setLikes(likes + 1)}>Like</Button>
          <p>{likes} Likes</p>
          </div>
      )
    }
```

Wow, suddenly implementing a Class for this 'like button' seems like a lot of unnecessary code. *Look at how many lines of code we are able to remove by utilizing hooks, and notice how much easier this code is to understand*.

Hooks are quite easy to pick up on if you're already familiar with React:

* You can define as many constants as you wish - you simply define your state (in this case, 'likes'), and define the update function ('setLikes'). From there, you ensure it's assigned to 'useState' and define your initial value ('0', in this case).
* Once you've defined your constant(s), you can utilize your state and update it as needed (with less code!) throughout your function.

**Take a look at some of your old code and be amazed at how many Classes you can refactor into functions with state!**
