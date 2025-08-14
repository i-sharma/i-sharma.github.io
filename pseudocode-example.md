Here is how you can highlight your pseudocode. You can use `plaintext` for a simple, un-highlighted code block:

```plaintext
function orderPizza():
    // Start the pizza-making job in the background
    futurePizza = startBackgroundJob(makePizza)
    return futurePizza


function main():
    // Step 1: Place the order
    pizzaBox = orderPizza()   // returns a Future object

    // Step 2: Chat with your friend/laptop while pizza is cooking
    chatForMinutes(10)

    // Step 3: Check the box
    if pizzaBox.isReady():
        eat(pizzaBox.get())
    else:
        print("Not ready yet... back to chatting")
        chatForMinutes(5)

    // Step 4: Decide to 'wait' for it now
    pizza = pizzaBox.waitUntilReady()  // blocking until pizza is done
    eat(pizza)


function makePizza():
    // simulate cooking time
    waitMinutes(12)
    return "Delicious Pizza"


// ---- Run the scenario ----
main()
```

If you want syntax highlighting that resembles a programming language, you can try using a language with a similar C-style syntax, like `javascript`:

```javascript
function orderPizza():
    // Start the pizza-making job in the background
    futurePizza = startBackgroundJob(makePizza)
    return futurePizza


function main():
    // Step 1: Place the order
    pizzaBox = orderPizza()   // returns a Future object

    // Step 2: Chat with your friend/laptop while pizza is cooking
    chatForMinutes(10)

    // Step 3: Check the box
    if pizzaBox.isReady():
        eat(pizzaBox.get())
    else:
        print("Not ready yet... back to chatting")
        chatForMinutes(5)

    // Step 4: Decide to 'wait' for it now
    pizza = pizzaBox.waitUntilReady()  // blocking until pizza is done
    eat(pizza)


function makePizza():
    // simulate cooking time
    waitMinutes(12)
    return "Delicious Pizza"


// ---- Run the scenario ----
main()
```

Choose the option that best fits the style of your blog.
