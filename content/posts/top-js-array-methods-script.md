---
title: Top JS array methods Script
date: 2020-06-29T23:43:59.755Z
tags:
  - javascript
  - es6
thumbnail: /images/uploads/9.png
---
When working with data in javascript there is a high chance that you will be handling arrays, either the data is coming from an external API in JSON format or you are working with that internally in your app. We must manipulate that data efficiently and clearly. In modern javascript we don't need to use traditional for loops, we can use a set of array methods that the language provides. Here are my favorite ones.

So let's say we have an array that has information about our video games collection, each item is an object that has the title of the game, the year of launch, the platforms that you own, the genre and a rating.

## Map

What if we want to have an array that only has the titles of our game collection, we can make use of the map method for that,

The map method creates a new array with the return value of a callback function that gets invoked for each item of our original array. What this means is that we can manipulate every item however we want, to compute new values, replace or add new properties.

Our callback takes every game object and we just return the game.title.
```js
const gamesTitles = gamesCollection.map((game) => game.title)
```

Suppose that I want to keep all of my previous game properties but I will like to compute a new one, the age property will simply be a number that represents how many years my game has been out in the market. We take the game object in our callback and we will first retrieve the current year and then we can subtract that from the game yearReleased property, finally, we will return a new object that spreads our previous properties and adds the new age.

```js
const games = gamesCollection.map((game) => {
    const age = new Date().getFullYear() - game.yearReleased
    return { ...game, age }
})
```

## Filter

Imagine now we want to filter our games by genre, so we only have our horror games. We can use the filter array method for that.
```js
const horrorGames = gamesCollection.filter((game) => {
  return game.genre === 'horror'
})
```

As before our callback function takes every game item object as the first parameter and then we need to return a boolean condition, so the method knows what to filter. if the condition is true the item will be part of our new array.

A side note, if no items satisfy the condition we will have an empty array as a result

## Find

In case we want to retrieve a specific game and we know the title we can use the find array method. This method is very similar to the filter one, we return a boolean condition in our callback function, but the difference here is that the method does not return an array, it retrieves a single item. If the method does not find anything it will return undefined and if there is more than once occurrences of our condition it will return the first one.

```js
const theLastOfUs = gamesCollection.find((game) => {
  return game.title === 'The last of us'
})
```

## Every / Some

Let's say we want to know if we own any ps4 games in our collection, the some method, will return true if at least one return value of our callback is true.
```js
const hasPs4Games = gamesCollection.some((game) => game.platform === 'ps4')
```

We would like to know if our collection is good, the every array method will also return a boolean but in this case if ALL of the items of the array satisfy the condition of our callback
```js
const isMyCollectionGood = gamesCollection.every((game) => game.score >= 85)
```

## Reduce

And finally, let's explore Reduce which is probably the most powerful and confusing array method of this list. hopefully, we can break it down so it is more clear

as before let's propose a practical use case.

We want to know the average score of our entire game collection, for that, we can attach our reduce method to our array.

Reduce is kind of different out of the previous methods, the first argument is still our callback function, but the first parameter of this callback is an accumulator, this is just a value that will be increasing or mutating in each item interaction of our array, and the second argument of the callback is the current item value. The second argument of the reduce method is an optional starting value of the accumulator because we want to compute the sum of all of our ratings the initial value will be 0, then for each iteration of our array, we will adding the accumulator value with the score of each game. 
```js
const scoresSum = games.reduce((acc, curr) => acc + curr.score, 0)
const avgScore =  scoresSum / gamesCollection.length
```

One useful tip that I can give you is that you name the callback parameters in a way that makes sense for you, so in my case, the accumulator will be the sum and the current value is our game

Once we have that we can simply divide the accumulator by the length of our collection and then we have the average score.

What makes reduce powerfully is that our accumulator does not needs to be a number it can be anything, let's take a look at a more complex example.

Let's say we want to have an object that has the name of the game as a key and the score rating as a value, in this case, the initial value will be an empty object, and for each iteration of the array we will create a new property that has the title and the value will be the score.
```js
const gamesScores = games.reduce((obj, data) => {
    acc[curr.title] = curr.score
    return acc
}, {})
```

Imagine we want to group our games by genre, in this case, the key value of the object will the genre and then we can check if the current genre exists in the accumulator we spread the value along with our current game, otherwise, we just assign the current game.

```js
const groupedByGenre = games.reduce((acc, curr) => {
    acc[curr.genre] = acc[curr.genre] ? [...acc[curr.genre], curr] : [curr]
    return acc
}, {})
```