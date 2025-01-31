---
title: "Error Boundaries"
path: "/error-boundaries"
order: "6A"
section: "Special Case React Tools"
description: ""
---

Frequently there's errors with APIs with malformatted or otherwise weird data. Let's be defensive about this because we still want to use this API but we can't control when we get errors. We're going to use a feature called `componentDidCatch` to handle this. This is something you can't do with hooks so if you needed this sort of functionality you'd have to use a class component.

This will also catch 404s on our API if someone give it an invalid ID!

A component can only catch errors in its children, so that's important to keep in mind. It cannot catch its own errors. Let's go make a wrapper to use on Details.js. Make a new file called ErrorBoundary.js

```javascript
// mostly code from reactjs.org/docs/error-boundaries.html
import { Component } from "react";
import { Link } from "react-router-dom";

class ErrorBoundary extends Component {
  state = { hasError: false };
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  componentDidCatch(error, info) {
    console.error("ErrorBoundary caught an error", error, info);
  }
  render() {
    if (this.state.hasError) {
      return (
        <h2>
          There was an error with this listing. <Link to="/">Click here</Link>{" "}
          to back to the home page or wait five seconds.
        </h2>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

- Now anything that is a child of this component will have errors caught here. Think of this like a catch block from try/catch.
- A static method is one that can be called on the constructor. You'd call this method like this: `ErrorBoundary.getDerivedStateFromError(error)`. This method must be static.
- If you want to call an error logging service, `componentDidCatch` would be an amazing place to do that. I can recommend [Azure Monitor][azure], [Sentry][sentry], and [TrackJS][trackjs].

Let's go make Details use it. Go to Details.js

```javascript
// add import
import ErrorBoundary from "./ErrorBoundary";

// replace export
const DetailsWithRouter = withRouter(Details);

export default function DetailsErrorBoundary(props) {
  return (
    <ErrorBoundary>
      <DetailsWithRouter {...props} />
    </ErrorBoundary>
  );
}
```

- Now this is totally self contained. No one rendering Details has to know that it has its own error boundary. I'll let you decide if you like this pattern or if you would have preferred doing this in App.js at the Router level. Differing opinions exist.
- We totally could have made ErrorBoundary a bit more flexible and made it able to accept a component to display in cases of errors. In general I recommend the "WET" code rule (as opposed to [DRY][dry], lol): Write Everything Twice (or I even prefer Write Everything Thrice). In this case, we have one use case for this component, so I won't spend the extra time to make it flexible. If I used it again, I'd make it work for both of those use cases, but not _every_ use case. On the third or fourth time, I'd then go back and invest the time to make it flexible.

Let's make it redirect automatically after five seconds. We could do a set timeout in the `componentDidCatch` but let's do it with `componentDidUpdate` to show you how that works.

```javascript
// top
import { Link, Redirect } from "react-router-dom";

// add redirect
state = { hasError: false, redirect: false };

// under componentDidCatch
componentDidUpdate() {
  if (this.state.hasError) {
    setTimeout(() => this.setState({ redirect: true }), 5000);
  }
}

// first thing inside render
if (this.state.redirect) {
  return <Redirect to="/" />;
} } else if (this.state.hasError) {
  …
}
```

- `componentDidUpdate` is how you react to state and prop changes with class components. In this case we're reacting to the state changing. You're also passed the previous state and props in the paremeters (which we didn't need) in case you want to detect what changed.
- Rendering Redirect components is how you do redirects with React Router. You can also do it programatically but I find this approach elegant.

> 🏁 [Click here to see the state of the project up until now: 10-error-boundaries][step]

[step]: https://github.com/btholt/citr-v6-project/tree/master/10-error-boundaries
[azure]: https://azure.microsoft.com/en-us/services/monitor/?WT.mc_id=reactintro-github-brholt
[sentry]: https://sentry.io/
[trackjs]: https://trackjs.com/
[dry]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
