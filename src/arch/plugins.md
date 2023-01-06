# Plugins

By now, hopefully you've learned how [components](components.md) work. Maybe you've also learned how to do advanced things like [helper functions](component-patterns.md#helper-functions). If you have, then you probably have seen the limitations of such functions. Helper functions should only be used in the same component they were defined. If you need to use the function in multiple components, then you use a _plugin_.

Plugins are essentially empty class files that can be accessed through the context by any component. Usually they have functions used for controlling various aspects of a robot. All components need to inherit the [`Plugin`](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/arch/Plugin.kt) abstract class so they can be registered.

Here is an empty plugin example:

```kotlin
private var foo_store: Foo? = null

val Context.foo
    get() = foo_store!!

class Foo: Plugin() {
    init {
        foo_store = this
    }

    /*
     * You can put any public function or variable here, and components will be
     * able to access it from the "ctx.foo" property.
     *
     * Functions here will also be able to access the context by using "this.ctx".
     */
}
```

That's some complicated code, which can be explained in the design section. For now, just consider it a rule of thumb that that code is **required** for the plugin to work. Next, we need to register the plugin in the [robot class](robot.md).

```kotlin
class MyRobot(teleop: LinearOpMode, cfg: Config): Robot(teleop, cfg) {
    override fun configure(builder: RuntimeBuilder) {
        builder.registerPlugin(Foo())
    }
}
```

## Example - The Distance Sensor

> **TODO:** The method for initializing plugins is unintuitive and literally hurts to explain. I will write this in later, once I create a better system for doing this. For now, feel free to look at a few example plugins:
>
> - [Distance Sensors](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/api/plugins/DistanceSensors.kt)
> - [Linear Slides](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/api/plugins/LinearSlides.kt)
> - [Wheels](https://github.com/BotsBurgh/BOTSBURGH-FTC-2022-23/blob/develop/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/api/plugins/Wheels.kt)

<!--



HELLO, FELLOW SOURCE CODE EXPLORER! This was my first draft for this section.
As said above, I'm going to rewrite it when the plugin system gets a little easier to use.
I'm keeping it here so I can copy + paste whatever still applies when I'm done changing things.



Distance sensors are one of the simplest pieces of hardware to program. You really only have to initialize them, and then you're free to call `getDistance` whenever you need their output. Because of this, we're going to create an example distance sensor plugin that other components will use. To begin, let's copy our empty plugin from above and change some of the names:

```kotlin
private var distance_sensors_store: DistanceSensors? = null

val Context.distance_sensors
    get() = distance_sensors_store!!

// Note the plurality of this name.
class DistanceSensors: Plugin() {
    init {
        distance_sensors_store = this
    }
    
    // TODO: Everything else :)
}
```

Now that that's out of the way, let's look at the `DistanceSensor` class. We know a few things:

1. A distance sensor can be aquired through the `hardwareMap` on the teleop.
2. It provides a function called `getDistance`, which takes an enum representing the unit of measurement to use.
    - In our case, we will only be using inches since the FTC arena uses the imperial system of measurement. (Each tile is a 2 foot by 2 foot square.)

With this in mind, let's figure out what we want out plugin to do:

- Initialize and store a distance sensor object for the front and back of a robot.
- Provide a function for getting the distance in inches from both the front and back sensors.

That sounds pretty simple! Let's get started by defining the variables that the distance sensors will be stored in:

```kotlin
class DistanceSensors: Plugin() {
    ...

    private var front: DistanceSensor? = null
    private var back: DistanceSensor? = null
}
```

The `front` and `back` variables store our distance sensor objects. We default them to null because when the plugin is created, we don't have access to the runtime context, and thus the hardware map to retrieve the objects. Speaking of hardware map, let's write an init functions for retrieve our objects!

```kotlin
class DistanceSensors: Plugin() {
    ...

    private var front: DistanceSensor? = null
    private var back: DistanceSensor? = null

    fun init() {
        // Wait a minute, where did "this.ctx" come from???
        this.front = this.ctx.teleop.hardwareMap.get(DistanceSensor::class.java, "distanceFront")
        this.back = this.ctx.teleop.hardwareMap.get(DistanceSensor::class.java, "distanceBack")
    }
}
```

Ok, so that's our initialization function. You may be wondering where the `this.ctx` property came from. To be completely honest, explaining it in depth would require explaining how the entire Arch API system works in the background. As nobody has that kind of time, let's just summarize:

- `init` is not special. It needs to be called from a component in the `pre` phase, or else the plugin will not work.
- A plugin is constructed into an object before being registered
    - When a class is constructed, all of its properties are initialized. This means that `front` and `back` need to be set to a value.
    - We don't have access to the hardware map yet to give it the right value, so we need to put in the placeholder `null`.
- After a plugin is registered, a property inherited from the `Plugin` class called `ctx` is created.
- Now that the plugin has access to the context, it is able to use the hardware map. Unfortuneately, it cannot do this on its own. It needs a component in the `pre` phase to help it.
    
-->
