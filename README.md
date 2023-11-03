# Decision Tables

This repo has two purposes:

1. To provide me with a tool I've been wanting.
2. To demonstrate some ideas about structuring Elm apps.

## Decision Tables

Our subject: decision tables!
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
| Marketing    | System          | Light                              | Dark         |
| Marketing    | System          | Dark                               | Dark         |
| Product      | Unknown         | Light                              | Light        |
| Product      | Unknown         | Dark                               | Dark         |
| Product      | Light           | -                                  | Light        |
| Product      | Dark            | -                                  | Dark         |
| Product      | System          | Light                              | Light        |
| Product      | System          | Dark                               | Dark         |

These are fantastic for both calculating and communicating about complex logic in UIs.
You can do a bunch of other interesting things with them too: finding properties for property tests, determining that you thought you needed some input but you don't, finding that you specified two conflicting things, etc.
Here's further reading: [an introduction](https://www.hillelwayne.com/post/decision-tables/), [more advanced patterns](https://www.hillelwayne.com/post/decision-table-patterns/), [modeling missing requirements](https://www.hillelwayne.com/requirements/).
(I have some materials by other authors if you ask me, but these are really good.
Just read 'em!)

## Domain Modeling

Let's break down the kinds of inputs and outputs we've seen:

- **Boolean:** yes or no, true or false.
  - "Raining" or "Open Windows?" above.
  - Could also model our color scheme above: "Is it dark mode?"
- **Enumeration:** A set of possible values.
  - "User Preference" above, which can be:
    - Unknown (not logged in)
    - Light / Dark / System (concrete preference)
- **Segmented Range:** sectioning up a continuous value.
  Importantly, this should always have full coverage of the possible range of values!
  It's not much use if the value is 70% and the two rules are "below 70%" and "above 70%", so the app needs to validate exhaustiveness here.

Once you have these possibilities, you can build them into tables.
All together, that might look like this:

```
type Possibilities
    = Boolean { trueLabel: String, falseLabel: String }
    | Enum (List String)
    | SegmentedRange { min: Int, max: Int, segments: List (Int, Int) }


type Value
    = StringValue String
    | IntValue Int
    | BoolValue Bool


type alias Column =
    { name : String
    , possibilities : Possibilities
    }


type alias Rule =
    { inputs : List Value
    , outputs: List Value
    }


type alias Table =
    { inputs: List Column
    , outputs: List Column
    , rules: List Rules
    , wantToCollapse: List (List Value)
    }
```

There's some work to do to keep this all in sync, so it needs to live behind a solid module boundary.
