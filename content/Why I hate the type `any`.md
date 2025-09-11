### Why I hate the `any` type in Typescript
`any` can eat my ass.  Although it may be necessary at some points, its usage greatly expands the possible error surface of a codebase.  In 90% of cases, I’d say if you’re choosing `any` , the right choice would be to choose `unknown` instead. 

`unknown` is the exact opposite of `any`.  While `any` assumes you can do _anything_ with the type, `unknown` assumes you can do _nothing_.  With `unknown`, you _must_ safely _define_ the shape of the type before using it.

“But `unknown` throws my code into a sea of red” some of you might say.  And it’s true, the moment you change your types from `any` to `unknown`, Unknown immediately takes to the streets and paints the town red.  But that’s as it should be.  All code that is typed with `any` *should* *be* *red* because it *is* ***unsafe***.  Changing `any` to `unknown` only serves to **reflect the true nature of your code**—when you use `any`, you don’t know what the type is or can be.

Let’s take a look at an example of where `any` is commonly used: to connect your application to the outside world.

We have an application that needs some data from an endpoint.  Simple right?  We just fetch the data and call it a day.  So that’s what we do:
```ts
async function fetchDecks(): Promise<FilledDeckData[]> {
  // send a get request to the endpoint
  const response = await client.get('/api/v1/deck/user');
  // parse the JSON body
  const data = await response.json();

  // check if the body has a decks field
  if (!data.decks) {
    throw new Error('An error has occurred while retrieving decks');
  }

  // check if each member of the array fits the `FilledDeckData` interface
  data.deck.forEach((deck: any) => {
    if (!deck.userId || !deck.title || !deck.description || !deck.id) {
      throw new Error('An error has occurred while retrieving decks');
    }
  });
  const result: FilledDeckData[] = data.deck;
  return result;
}
```

Looks pretty simple right?  We simply send a `GET` request to the endpoint, parse the response into an object.  Then we check if the shape of the data matches our `FilledDeckData` interface.

If it does, we narrow the type and return the result.
If it doesn’t, we throw appropriate errors which will be handled upstream.

But the problem is… this code isn’t right.  

Yet Typescript doesn’t say a word.  
Why is that?

Well let’s track the types throughout the function
```ts
async function fetchDecks(): Promise<FilledDeckData[]> {
  const response = await client.get('/api/v1/deck/user');                // response is type `Response` (standard javascript Response interface)
  const data = await response.json();                                        // data is type `any` because `json()` returns a Promise with `any` value

  // check if the body has a decks field
  if (!('decks' in data))                                                    // data is type `any` because `any` already includes the 'decks' field
    throw new Error('An error has occurred while retrieving decks');
  }

  // check if each member of the array fits the `FilledDeckData` interface
  data.deck.forEach((deck: any) => {
    if (!('userId' in deck && 'title' in deck && 'description' in deck && 'id' in deck)) {
      throw new Error('An error has occurred while retrieving decks');
    }
  });                                                                        // data is still type `any` because `any` encompasses every type
  const result: FilledDeckData[] = data.deck;                                // because `data` is still type `any`, we have to manually narrow the type.
  return result;
}
```

From the point where data was typed `any` on, Typescript essentially became useless.   Although our typechecks enforce our constraints at runtime, our code in this function is effectively no longer typechecked.

Isn’t that fine though?  Runtime is more important than compile time -- as long as our constraints are enforced correctly at runtime, this doesn’t matter that much.  Right?

I would've agreed if not for the fact that the above function isn’t correct.  And that the bugs it contains would've never been pushed if `response.json()` simply returned an `unknown` type rather than `any`.

#### The Bugs
Even though it *looks like* we’ve validated the data in the above function, **we haven’t**.  That’s what makes the behavior of using `any` so dangerous.

For example, in the above code, what happens if `data` is `null/undefined` ?  A type error will be thrown at `if (!('decks' in data))` and our program would enter an invalid state.

What about if `data.deck` isn’t an array?  Well you can’t use `forEach()` on any other primitive, so another type error occurs.

Also, what if `data.deck` doesn’t exist? Actually that shouldn’t be a problem because we already checked it exists with the line `if (!('decks' in data))`.

Oh wait… We checked if `data.decks` exists, not `data.deck`.  Typescript also stays quiet about this spelling mistake.

These are just some of the many issues that are allowed to slip through in the dead-types space created after `response.json()` returned an `any`.    The point is, using `any` **greatly increases the possible bug surface of your code** and this is made even more dangerous because it lets us think we did our due diligence when we didn’t.
#### How does using unknown fix anything?
Let's try taking the above code and changing `response.json()`'s return type to `unknown`:
![[using-unknown 1.png]]
As expected, `unknown` douses the code in a sea of red.  But for every error it reveals, there exists a very real bug that we have failed to account for.  (You can match the errors `unknown` highlights to the list of bugs above).

Put simply, if `response.json()` returned `unknown` instead of `any`, **none of the bugs mentioned above would've been possible.** 

Using `any` increases the bug surface of your code until it is rescued back to type-safe land through an assertion.  Here, our code is unsafe from the moment we call `response.json()` until we assert the `any`  is `FilledDeckData[]` (even then, the assertion itself is a bug waiting to happen).  And what makes `any`'s crimes even more grave is that it allows for introductions of bugs in the places where it matters most: where our applications interface with the outside world, with unknown territory.  

By defining these unknown types as `any`, we kill off the usefulness of typescript and say it's okay to use these unknowns with anything -- a dangerous a-okay to tell to Typescript.

Using `any` leads to crashes, wrong type information, and hard-to-track behavior that, if you're lucky, rears its ugly head immediately while you're working on the code or, if you're not, bids its time until it's deep into legacy-code territory.

In conclusion, `any` can eat my ass.  The next time you work with data you don't know, consider typing it `unknown` instead of `any` .  Work through the errors Typescript highlights for you so that both you and your teammates can use the code with confidence.