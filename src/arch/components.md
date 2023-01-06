# Components

Components are how developers can insert functions into the [runtime](runtime.md). Components can currently either insert initialization code (pre) or runloop code (cycle). 

## Creating a component

A component is usually created in a class. Here is an excerpt from the [component abstract class](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/arch/Component.kt):

```kotlin
abstract class Component {
    open val pre: CtxFunc? = null
    open val cycle: CtxFunc? = null

    open val order: Byte = 0
}
```

A component has three optional properties, though most components will only override 2. `pre` and `cycle` are anonymous functions (or lambdas) that will be run in a certain phase in the runtime. If a function is not specified in the component, it will not be added to the runtime. (Null is ignored in `pre` and `cycle` when registering the component.)

Here is an example telemetry component:

```kotlin
/*
 * This will print "Initialized" and "Running" in their respective phases.
 * It won't work correctly yet, so please continue reading.
 */
class Logger: Component() {
    override val pre = fun(ctx: Context) {
        ctx.teleop.telemetry.addData("Status", "Initialized")
        ctx.teleop.telemetry.update()
    }

    override val cycle = fun(ctx: Context) {
        ctx.teleop.telemetry.addData("Status", "Running")
        ctx.teleop.telemetry.update()
    }

    ...
}
```

> `CtxFunc` is a [type alias](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/9e1a68deb9e34f040c19148f46c035d34a3d51da/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/arch/Context.kt#L7) for `(ctx: Context) -> Unit`.

`order`, on the other hand, is a `Byte` that represents when a component should be run. Lower numbers are executed first. `order` can be between -127 and 127, but it defaults to 0.

```kotlin
// This will run before Bar, since -1 < 0.
class Foo: Component() {
    override order = -1;
}

// This will run after Foo.
class Bar: Component() {
    override order = 0;
}
```

Looking back at our `Logger` component, we can now see the issue. Its order is not defined, meaning we cannot confirm whether it will run after all other components. `Logger` uses `telemetry.update()`, which is usually run at the end of the runloop. (Having multiple `update()`s in a single runloop cycle breaks things.) To fix this, let's define the order.

```kotlin
class Logger: Component() {
    override order = 127; // Or Byte.MAX_VALUE
}
```

That's our first working component! For the full code, you can look at [the Github repo](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/e2cb96f4cc37e7717553451c69f9a0134370717d/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/api/components/LoggerTeleOp.kt).

## Registering a component

We've programmed a simple logger component, but how do we use it? This is where the [robot class](robot.md) comes into play. (Please look at that page first for setting up a robot class before you continue here.)

In the robot class, you need to register your component with the `registerComponent` function. Let's register out `Logger` component.

```kotlin
// Make sure to also run this in a new teleop! There's more information in the robot class chapter.
class MyRobot(teleop: LinearOpMode, cfg: Config): Robot(teleop, cfg) {
    override fun configure(builder: RuntimeBuilder) {
        builder.registerComponent(Logger())
    }
}
```

There we go! If everything worked, then you should be able to run your new teleop and it should print `Status: Running` on the driver station.
