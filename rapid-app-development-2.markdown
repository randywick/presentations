# Create a dumb example

Let's create a very basic web UI by shamelessly stealing an old codepen I wrote while trying to learn some animations a few years ago.  We'll pretend this has some more complex logic, server stuff, API maybe.

script.js
```javascript
class CharacterWrapper {

    constructor (char) {
        this.char = char;
    }

    get wrapper () {
        const span = document.createElement('span');
        span.textContent = this.char;
        span.classList.add("character")

        return span;
    }

}

class SmoothCounter {

    constructor (element) {
        this.element = element;
        this.value = 0;
        this.updateInterval = 20;
        this.changeInterval = 400;
        this.play();
    }

    get value () {
        return this._value;
    }

    set value (value) {
        this._value = Math.max(value, 0);
    }

    play () {
        this.tickInterval = setInterval(() => this.update(), this.updateInterval);
    }

    pause () {
        clearInterval(this.tickInterval);
    }

    update () {
        const numberString = $(this.element)
            .find("> span")
            .toArray()
            .filter((span) => !span.classList.contains("absolute"))
            .map((span) => span.textContent)
            .join("")
            .replace(/,/g, '');

        const currentValue = Number.parseInt(numberString, 10) || 0;

        if (currentValue === this.value) {
            return;
        }

        let change;

        if (this.value > currentValue) {
            change = Math.min(1, this.value - currentValue);
        } else {
            change = Math.min(1, currentValue - this.value) * -1;
        }

        const resultElements = (currentValue + change)
            .toLocaleString()
            .split("")
            .map((char) => new CharacterWrapper(char).wrapper)
            .reverse();

        const currentElements = $(this.element)
            .find("> span")
            .toArray()
            .reverse();

        const elementLeft = this.element.getBoundingClientRect().x;

        currentElements.forEach((span, index) => {
            if (!resultElements[index]) {
                return;
            }

            if (span.textContent !== resultElements[index].textContent) {
                const left = span.getBoundingClientRect().x - elementLeft;
                $(span)
                    .addClass("absolute changing")
                    .css({ left });
            } else {
                $(span).addClass("absolute hidden");
            }
        });

        setTimeout(() => currentElements.forEach((span) => $(span).remove()), this.changeInterval);

        resultElements
            .reverse()
            .forEach((span) => $(span).appendTo($(this.element)));

        // this.element.textContent = (currentValue + change).toLocaleString();

    }

}

const element = document.querySelector(".value");
const counter = new SmoothCounter(element);

document.querySelector(".add").addEventListener("click", () => (counter.value += 100));
document.querySelector(".subtract").addEventListener("click", () => (counter.value -= 100));
```

app.css
```css
html, body {
	 padding: 0;
	 margin: 0;
}
 body {
	 display: flex;
	 flex-flow: row nowrap;
	 align-items: center;
	 justify-content: space-around;
	 height: 100vh;
}
 .value {
	 font-family: sans-serif;
	 font-size: 96px;
	 position: relative;
}
 .value > span {
	 transition: transform 400ms, opacity 400ms;
	 background-color: white;
}
 .value > span.absolute {
	 position: absolute;
}
 .value > span.changing {
	 display: inline-block;
	 transform: translate(8px, -100px) rotate(5deg);
	 opacity: 0;
	 z-index: 1;
}
 .value > span.hidden {
	 visibility: hidden;
}
 button {
	 padding: 15px;
	 text-transform: uppercase;
	 background: white;
	 border: 2px solid #ccc;
	 font-weight: bold;
	 font-size: 64px;
}
 button.add {
	 color: green;
}
 button.subtract {
	 color: red;
}
 .controls > button {
	 cursor: pointer;
}
```

index.html
```html
<html>
  <body>
    <head>
      <title>Some People Count</title>
      <link rel="stylesheet" href="app.css">
      <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.js"></script>
    </head>
    <div class="main">
      <div class="output">
          <span class="value">
              <span class="character">0</span>
          </span>
      </div>
      <div class="controls">
          <button class="subtract">- 100</button>
          <button class="add">+ 100</button>
      </div>
    </div>
    <script src="script.js"></script>
  </body>
</html>
```

# Non-deployment

We can simply open this html file in a browser window, but because of the way
browsers restrict resource access across origins, this becomes unserviceable
the instant we begin to add complexity into our new application.  For example,
since our `origin` is `null`, we are not allowed to make any AJAX calls for
dynamic data.

CORS.  Am I right?

We can eliminate some of these issues by serving the app locally.  Let's see
what that looks like.  Any webserver will work, but since we're especially lazy
let's use the go-to `http-server`

```
$ http-server
```

We can see that this is served from our static webserver, but for a variety of
reasons, it's still less than ideal.  For example, although we've defeated the
browser's cross origin restrictions by upgrading our origin from `null` to
`localhost`, we are still unlikely to be granted cross-origin access to third
party resources since project may be served at this origin and so it lacks even
rough identity resolution.

Further, we cannot access our site via https, which can cause additional
resource access issues.  We just need to get this thing onto a dedicated host.

# Deployment, first example

https://surge.sh/

For the most basic of projects, we can rapidly deploy this as a static website.
There are plenty of patterns we can follow, but let's go for the simplest.

Let's `surge`.

```
$ surge
```

If this is the first time we're deploying this project using `surge`, we may
be required to authenticate and provide some basic project details, but within
a few seconds, we can have our project hosted and available within a few
seconds.

We can further configure this to use a custom domain or subdomain by either
specifying the appropriate runtime arguments or by creating a file named `CNAME`
containing our custom domain.  Cool, let's do that.

```
imaginarysignal.com
```

## Alternatives

- Netlify
- Render*
- GitHub Pages*
- Firebase
- Vercel*

And plenty more.  Instead of using a CLI to deploy, some services only work via
VCS webhooks.  Because we're focused on rapid prototyping, at this stage I
prefer a CLI.

VCS integration, on the other hand, introduces its own pros and cons.  For the
(obvious but minor) cost in data privacy, Many services will simplify the CI/CD
process around common frameworks so we can build and deploy, for example,
a Gatsby SSR package, or the latest flavor of JAMstack.  Which sounds kinda
gross.  Beyond framework integration, this tier of services offers things like
server-side logic, databases, and, increasingly, functions as a service.  Even
better, we can preview changes in lower environments or feature branches --
sometimes with no configuration necessary.

Let's explore.

# Vercel

https://vercel.com/dashboard

I'm lazy, but I'm also tasked with provisioning multiple environments for our
new project.

_This always happens to me._

Let's simplify.  We can use a service like Vercel, which used to be called,
simply, `now` (or `zeit` if you want to be all German about it).  We can
attach Vercel to a GitHub project and automatically build and deploy upon
merge triggers.

Vercel even provides us with ephemeral preview builds upon push trigger to non-
production or domain-linked branch.

[Push some trivial change to a new branch]

While this is a static website -- we don't have the ability to deploy a server
here -- we do have access to FaaS in the form of Vercel Serverless Functions.
Serverless functions can be written in a variety of languages and are dead
simple to implement.

At this point I should emphasize that we've moved beyond simple prototyping and
into a plausible production environment for low to moderately complex
applications.

Quick note: there is nothing special about GitHub in this equation, and Vercel
seems to work with any comparable Git-flavored VCS provider.

## Alternatives

- Glitch
- Heroku
- Aerobatic

# Intermission / Bonus Service

https://glitch.com/

Glitch is one of the first services I ever encountered that offered free
realtime source collaboration, free static website hosting, free node.js server
hosting, and more.

In 2021, we have access to things like Live Share through VSCode, but Glitch
offered its collab editor long before this was available.  It's still worth a
look.

# Orchestration At Scale

https://aws.amazon.com/amplify/

The last time we talked about rapid application development, we looked at the
Serverless Framework, which combines tooling from a variety of providers in
order to simplify the orchestration, management, and lifecycle of complex
service meshes through a unified definition layer combining aspects of IaaS and
application manifest.

Let's look at a similar effort, but one more opinionated.

AWS Amplify is two things:

1. A framework for full-stack web and mobile applications with support for things such as authentication, data syncing, push notification, GraphQL, and more;
1. A console for lifecycle management of framework applications, familiar features such as merge-trigger deployments, preview deployments, and other service integrations.

We can use Amplify to host our simple static website, and then we can
incrementally improve it using the framework.

[IF TIME, SET UP STATIC HOSTING FROM REPO]

The Amplify CLI provides some particularly juicy functionality:

### Set up authentication

```
$ amplify add auth
```

We can configure identity providers, user pools, federated login, and so forth,
and then within our application we simply use the Amplify SDK to initiate a
login session or drop the pre-built React components into our app.  We never
have to worry about refreshing or invalidating tokens, configuring environment-
or stage-based user pools, or anything.

### Provision and Access Data

We can use the CLI to create and configure DynamoDB tables...

```
$ amplify add storage
```

...And then access those
data via REST or GraphQL.  We can further use AppSync to reflect realtime
changes to data on the client.

```
$ amplify add api
```

This generator not only creates the CloudFormation templates necessary to create
and manage the necessary infrastructure, it also configures IAM and the Amplify
SDK to restrict or allow access to storage to unauthenticated users.

### Functions, Oh My

For tasks requiring custom server logic, we can configure and manage Lambdas,
also using the CLI:

```
$ amplify add function
```

Typically we would think of a Lambda in conjunction with an application like
this as an API logic provider.  However, since we have API access built in via
the storage and API modules, we don't actually need any special handling for
this, and under the hood the Amplify SDK's API class uses AppSync anyway.

However, there are circumstances where we need to handle logic away from the
client.  For example, logic consuming secrets such as third party tokens.

# Conclusion

There are a ton of tools and services available to facilitate rapid application
development at every conceivable stage.

In this presentation, we looked at services of particular value to our sample
application at a variety of stages of maturity.  In practice, we wouldn't likely
"graduate" from one tier to the next in the way this might suggest.  Vendor
lock-in, opinionated SDKS, and data access patterns established early on in the
application's life influence appropriate and available architectural frameworks
later on.  Moreover, a heavy framework such as Amplify applied to a tiny static
website such as our example would be monumental overkill.

The lesson I hope we can take from this presentation is that there are a variety
of ways to accelerate our process and simplify our infrastructure all while
building our skills with, and, let's face it, getting to play with, some of the
latest and hottest technologies.  It's the same lesson I had in mind when we
discussed the Serverless framework back in January, and I'm thrilled to see the
way our architecture choices have evolved over time to be less product-of-what-
came-before and more targeted to the issue at hand.

Thank you.
