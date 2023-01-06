# Advanced Patterns

## Maintaining State

If your component needs to maintain its state (a variable) between multiple executions of the `cycle` function, you can just add the property to the class.

```kotlin
class Counter: Component() {
    // Make sure its a "var" so it can be changed later!
    private var count = 0
 
    override val cycle = fun(ctx: Context) {
        this.count += 1

        ctx.teleop.telemetry.addData("Count", this.count)

        // Remove this depending on whether you have a logger component registered
        ctx.teleop.telemetry.update()
    }
}
```

## Helper Functions

If your component needs to do the same thing multiple times in a cycle, or maybe needs to increase readability, it may be worth splitting up code into functions. Not everything has to be stored in `pre` and `cycle`.

```kotlin
class MySuperReadableComponent: Component() {
    override val pre = fun(ctx: Context) {
        sayHello()
        configureStuff(ctx)
        sayGoodbye()
    }

    /*
     * These functions should be private, as this is not a plugin and no other
     * component should access this.
     */
    private fun sayHello() {
        println("Hello!")
    }

    /*
     * Most functions will still need access to the context, so you can pass it
     * in with a function parameter.
     */
    private fun configureStuff(ctx: Context) {
        ctx.my_plugin.init()
    }

    private fun sayGoodbye() {
        println("Goodbye!")

        // Kotlin will fail to compile if we uncomment the following code:
        // (There's not context parameter!)
        //
        // ctx.teleop.telemetry.addData("MySuperReadableComponent", "Goodbye!")
    }

    
}
```

> **Note:** The `println` function will not actually work in a component. (I just needed an example that didn't use the context variable.) If you need to print text, please use the telemetry.
