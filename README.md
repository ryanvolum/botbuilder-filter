# Filtering Middleware

## Getting Started

Install from npm:
```
npm install botbuilder-filter-middleware --save
```
Import the package:
```
import { filterMiddleware } from 'botbuilder-filter-middleware';
```
See sample for full implementation. 

## Background

When we .use middleware, every message we receive gets passed through it. We use middleware to gather things like intent or sentiment, or to log telemetry, or to transform messages (e.g. translation), or to intercept a message. However, we don't always want our middleware to run. For instance, sentiment analysis on text is only useful if that text is fairly long. "No" registers as having incredibly low sentiment, though it doesn't necessarily indicate a disgruntled user. In this case, we would only want to call our sentiment analysis service if our incoming message is over a specific length. 

Enter middleware filtering. filterMiddleware is just a function that only runs middleware if a predicate is true.

If we want our bot to pass route messages through a piece of middleware, we .use use that middleware:

```ts
const bot = new Bot(connector)
    .use(
        new TestMiddleware()
    )
```

If we only want that middleware to run based on custom logic we pass in our filterMiddleware function, which takes a predicate and middleware:

```ts
const bot = new Bot(connector)
    .use(filterMiddleware(
        lengthPredicate,
        new TestMiddleware()
    ))
```

A predicate is just a boolean function. In the above case, our lengthPredicate is checks whether the length of the message in context.request.text is over 5 characters:

```ts
const lengthPredicate = (context: BotContext) => {
    return (context.request.type === "message" && context.request.text && context.request.text.length > 5);
}
```

For the sake of demonstration, our TestMiddleware replies to the user if it runs:

```ts
    public receiveActivity(context: BotContext, next: () => Promise<void>): Promise<void> {
        context.reply('called receiveActivity');
        return next();
    }
```

So, the sample will only reply from middleware if the incoming message is at least 5 characters long! Of course, you can define any predicate. 

Note, we can pass an arbitrary number of middlewares into the filter:

```ts
const bot = new Bot(connector)
    .use(filterMiddleware(
        lengthPredicate,
        new LuisRecognizer(env.LUIS_APP_ID, env.LUIS_PASSWORD)
        new TestMiddleware()
    ))
```

Neither of the above pieces of middleware will run if the length predicate is not met.
