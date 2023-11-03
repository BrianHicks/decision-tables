# Decision Tables

This repo has two purposes:

1. To provide me with a tool I've been wanting.
2. To demonstrate some ideas about structuring Elm apps.

The subject: decision tables!
I want an app that make working with decision tables super easy.
Here comes one now, for helping me decide whether or not to open the windows:

| Raining | Temperature (°F) | Relative Humidity | Open Windows? |
| ------- | ---------------- | ----------------- | ------------- |
| Yes     | below 60°        | below 80%         | No            |
| Yes     | below 60°        | 80% or above      | No            |
| Yes     | 60–80°           | below 80%         | No            |
| Yes     | 60–80°           | 80% or above      | No            |
| Yes     | above 80°        | below 80%         | No            |
| Yes     | above 80°        | 80% or above      | No            |
| No      | below 60°        | below 80%         | No            |
| No      | below 60°        | 80% or above      | No            |
| No      | 60–80°           | below 80%         | **Yes**       |
| No      | 60–80°           | 80% or above      | No            |
| No      | above 80°        | below 80%         | No            |
| No      | above 80°        | 80% or above      | No            |

We've got a bunch of input variables and an output variable.
You can figure out an answer ("Open Windows?") by supplying values for all the inputs.

This decision table is explicit but verbose.
We can improve readability by substituting `-` for rows in which the distinction between values doesn't matter, given the values to the left:

| Raining | Temperature (°F) | Relative Humidity | Open Windows? |
| ------- | ---------------- | ----------------- | ------------- |
| Yes     | -                | -                 | No            |
| No      | below 60°        | -                 | No            |
| No      | 60–80°           | below 80%         | **Yes**       |
| No      | 60–80°           | 80% or above      | No            |
| No      | above 80°        | -                 | No            |

Much easier to read, and the result is the same!

The windows example is nice but maybe too simple: you could get away with three `if` statements since there's only one condition where you actually want to open the windows.
Here's a more complex example, for deciding whether or not to enable dark mode on a website:

| Page Purpose | User Preference | `prefers-color-scheme` Media Query | Color Scheme |
|--------------|-----------------|------------------------------------|--------------|
| Marketing    | Unknown         | Light                              | Dark         |
| Marketing    | Unknown         | Dark                               | Dark         |
| Marketing    | Light           | -                                  | Light        |
| Marketing    | Dark            | -                                  | Dark         |
| Marketing    | Unset           | Light                              | Dark         |
| Marketing    | Unset           | Dark                               | Dark         |
| Product      | Unknown         | Light                              | Light        |
| Product      | Unknown         | Dark                               | Dark         |
| Product      | Light           | -                                  | Light        |
| Product      | Dark            | -                                  | Dark         |
| Product      | Unset           | Light                              | Light        |
| Product      | Unset           | Dark                               | Dark         |

These are fantastic for both calculating and communicating about complex logic in UIs.
You can do a bunch of other interesting things with them too: finding properties for property tests, determining that you thought you needed some input but you don't, finding that you specified two conflicting things, etc.
Here's further reading: [an introduction](https://www.hillelwayne.com/post/decision-tables/), [more advanced patterns](https://www.hillelwayne.com/post/decision-table-patterns/), [modeling missing requirements](https://www.hillelwayne.com/requirements/).
