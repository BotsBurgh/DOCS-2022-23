# Robot

The Arch API's robot class is an inheritable class that controls the registration of components and plugins. They are used for configuration and managing custom runmodes with different components.

A robot class inherits the [Robot](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/arch/Robot.kt) class. The current year's robot class is called [TriRobot](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/api/TriRobot.kt). A simple, blank robot class looks like the following:

```kotlin
class EmptyRobot(teleop: LinearOpMode, cfg: Config): Robot(teleop, cfg) {
    override fun configure(builder: RuntimeBuilder) {
        // Register any components here!
        // builder.registerComponent(MyComponent())
    }
}
```

This robot class can now be used by any teleop.

```kotlin
@TeleOp(name = "TeleOp Main")
class TeleOpMain: LinearOpMode() {
    override fun runOpMode() {
        // This will run the entire Arch runtime!
        EmptyRobot(this, Config(RunMode.TeleOp)).run()
    }
}

```

> **Note:** This is the reason why our teleop files never change. All meaningful code is either in the robot class or in a modular component / plugin.

## Notable `RuntimeBuilder` functions

The `configure` function in a robot class is what's used to register all components and plugins. This function is passed a [`RuntimeBuilder`](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/arch/runtime/RuntimeBuilder.kt) object. This object has many functions, but the important ones are:

- ```kotlin
  fun registerComponent(component: Component): RuntimeBuilder
  ```

  This registers any object that inherits [`Component`](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/arch/Component.kt). Note that it requires an object, not a class. Make sure to `registerComponent(Foo())` instead of `registerComponent(Foo)`.

  ### Example

  ```kotlin
  builder.registerComponent(Wheels())
  ```

- ```kotlin
  fun registerPre(func: CtxFunc, order: Byte = 0): RuntimeBuilder
  ```

  This registers just a function to be run once in the pre phase. `CtxFunc` is a [type alias](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/9e1a68deb9e34f040c19148f46c035d34a3d51da/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/arch/Context.kt#L7) for `(ctx: Context) -> Unit`. The `order` parameter specifies when the given `func` will be run. Smaller numbers will be run first, with 0 being the default. The order can be between -127 and 127.

  ### Example

  ```kotlin
  // Print "Initialized" once done initializing.
  builder.registerPre(fun(ctx: Context) {
    ctx.teleop.telemetry.addData("Status", "Initialized")
    ctx.teleop.telemetry.update()
  }, order = 127)
  ```

  > **Note:** This function also contains the `runMode` property. I have not mentioned it, as there may be an API change in the future that removes it. It was originally created to configure whether a component should be run in the teleop or autonomous section, but due to runtime differences in the two sections this argument may be removed.

- ```kotlin
  fun registerCycle(func: CtxFunc, order: Byte = 0): RuntimeBuilder
  ```

  This is the same as `registerPre`, but the given `func` is run multiple times in the runloop instead of once in the initialization one. For more information, see the [runtime](runtime.md) page.

---

All functions in the builder are chainable, meaning you can do:

```kotlin
builder
    .registerComponent(Foo())
    .registerComponent(Bar())
```

...instead of:

```kotlin
// This is still possible, but more frusturating.
builder.registerComponent(Foo())
builder.registerComponent(Bar())
```
