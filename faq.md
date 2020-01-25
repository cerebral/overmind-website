# FAQ

## The devtool is not responding?

First… try to refresh your app to reconnect. If this does not work make sure that the entry point in your application is actually creating an instance of Overmind. The code **createOvermind\(config\)** has to run to instantiate the devtools.

## The devtools does not open in VS Code?

Restart VS Code

## My operator actions are not running?

Operators are identified with a Symbol. If you happen to use Overmind across packages you might be running two versions of Overmind. The same goes for core package and view package installed out of version sync. Make sure you are only running on package of Overmind by looking into your **node\_modules** folder.



