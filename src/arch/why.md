# What it solves

The Arch API was designed to fix some issues that we ran into [last year](https://github.com/BotsBurgh/BOTSBURGH-FTC-2021-22), specifically placing all robot logic within a [single class](https://github.com/BotsBurgh/BOTSBURGH-FTC-2021-22/blob/main/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/api/Robot.java). It was bulky, difficult to read, and difficult to edit.

The Arch API, while more complicated, provides some very useful features when programming the robot.

- Toggling on and off certain features, specifically for testing individual components of the robot
- Having a dedicated robot lifecycle, so all code could be organized into initialization and runloop stages
- Splitting up code so individual files are easier to read and manage
