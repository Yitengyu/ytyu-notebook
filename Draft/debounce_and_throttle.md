# Debounce And Throttle

### Debounce

Respond to an event only if it would not be triggered in a certain time.

In other word, merge events triggered in less than a certain frequency to the very last one.

Advantage: Only respond to a relatively stable state, decrease unnecessary change.

```javascript
// example
const debounce = (func, delay) => {
  let inDebounce
  return function() {
    const context = this
    const args = arguments
    clearTimeout(inDebounce)
    inDebounce = setTimeout(() => func.apply(context, args), delay)
  }
}

const scrollHandler = () => {
    console.log("you are scrolling.")
}
window.addEventListener("scroll", debounce(scrollHandle, 50))
```



eg.

A: "I wanna an apple."

A: "No, banana." (Immediately)

A: "Sorry, apple again." (Immediately)

A: "OK, orange, thanks". (Immediately)

After 5 seconds.

B: "Here is your orange."



## Throttle

Only trigger event once in a certain period.

A certain period could be a limited time, or a request-respond process.

```javascript
//example
const throttle = (func, limit) => {
  let inThrottle
  return function() {
    const args = arguments
    const context = this
    if (!inThrottle) {
      func.apply(context, args)
      inThrottle = true
      setTimeout(() => inThrottle = false, limit)
    }
  }
}
```

Eg. (not same with the above code)

0:00 A: "I wanna an apple."

0:00 B goes into shop

0:05 B comes back with an apple

0:06 A: "I wanna another two apples."

0:06 B goes into shop

0:07 A : "another three, thanks."

0:08 A: "plus another banana"

0:11 B comes with two apples, and goes into shop

0:16 B comes with three apples and one banana.


PS: Suppose there is something to record A's requests.



