# Intro

The idea of Functional Reactive Programming is quite well described by Conal Elliot at [Stack Overflow](http://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming/1030631#1030631).

Bacon.js is a library for functional reactive programming. Or let's say it's a library for
working with [events](#event) and dynamic values (which are called [Properties](#property) in Bacon.js).

Anyways, you can wrap an event source,
say "mouse clicks on an element" into an [`EventStream`](#eventstream) by saying

```js
var cliks = $("h1").asEventStream("click")
```

Each EventStream represents a stream of events. It is an Observable object, meaning
that you can listen to events in the stream using, for instance, the [`onValue`](#stream-onvalue) method
with a callback. Like this:

```js
cliks.onValue(function() { alert("you clicked the h1 element") })
```

But you can do neater stuff too. The Bacon of bacon.js is in that you can transform,
filter and combine these streams in a multitude of ways (see API below). The methods [`map`](#observable-map),
[`filter`](#observable-filter), for example, are similar to same functions in functional list programming
(like [Underscore](http://documentcloud.github.com/underscore/)). So, if you say

```js
var plus = $("#plus").asEventStream("click").map(1)
var minus = $("#minus").asEventStream("click").map(-1)
var both = plus.merge(minus)
```

.. you'll have a stream that will output the number 1 when the "plus" button is clicked
and another stream outputting -1 when the "minus" button is clicked. The `both` stream will
be a merged stream containing events from both the plus and minus streams. This allows
you to subscribe to both streams with one handler:

```js
both.onValue(function(val) { /* val will be 1 or -1 */ })
```

In addition to EventStreams, bacon.js has a thing called [`Property`](#property), that is almost like an
EventStream, but has a "current value". So things that change and have a current state are
Properties, while things that consist of discrete events are EventStreams. You could think
mouse clicks as an EventStream and mouse position as a Property. You can create Properties from
an EventStream with [`scan`](#observable-scan) or [`toProperty`](#stream-toproperty) methods. So, let's say

```js
function add(x, y) { return x + y }
var counter = both.scan(0, add)
counter.onValue(function(sum) { $("#sum").text(sum) })
```

The `counter` property will contain the sum of the values in the `both` stream, so it's practically
a counter that can be increased and decreased using the plus and minus buttons. The [`scan`](#observable-scan) method
was used here to calculate the "current sum" of events in the `both` stream, by giving a "seed value"
`0` and an "accumulator function" `add`. The scan method creates a property that starts with the given
seed value and on each event in the source stream applies the accumulator function to the current
property value and the new value from the stream.

Properties can be very conveniently used for assigning values and attributes to DOM elements with JQuery.
Here we assign the value of a property as the text of a span element whenever it changes:

```js
property.assign($("span"), "text")
```

Hiding and showing the same span depending on the content of the property value is equally straightforward

```js
function hiddenForEmptyValue(value) { return value == "" ? "hidden" : "visible" }
property.map(hiddenForEmptyValue).assign($("span"), "css", "visibility")
```

In the example above a property value of "hello" would be mapped to "visible", which in turn would result in Bacon calling

```js
$("span").css("visibility", "visible")
```
