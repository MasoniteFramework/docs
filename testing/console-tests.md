# Console Tests

You can test what has been output to standard console during Masonite unit tests thanks to useful
console assertions.

Output here is the standard output often named `stdout`.
Error here is the standard error often named `stderr`.

External packages, prints in your code can output content in console (as output or error).

{% hint style="warning" %}
If you want to assert content output by a Masonite command you should use [Commands Tests](/testing/commands-tests.md#available-assertions) assertions instead.
{% endhint %}


### Available Assertions

The following assertions are available:

* [assertConsoleEmpty](console-tests.md#assertconsoleempty)
* [assertConsoleNotEmpty](console-tests.md#assertconsolenotempty)
* [assertConsoleExactOutput](console-tests.md#assertconsoleexactoutput)
* [assertConsoleOutputContains](console-tests.md#assertconsoleoutputcontains)
* [assertConsoleOutputMissing](console-tests.md#assertconsoleoutputmissing)
* [assertConsoleHasErrors](console-tests.md#assertconsolehaserrors)
* [assertConsoleExactError](console-tests.md#assertconsoleexacterror)
* [assertConsoleErrorContains](console-tests.md#assertconsoleerrorcontains)

#### assertConsoleEmpty

Assert that nothing has been printed to the console.

#### assertConsoleNotEmpty

Assert that something has been printed to the console (output or error).

#### assertConsoleExactOutput

Assert that console standard output is equal to given output.

```python
print("Success !")
self.assertConsoleExactOutput("Success !\n")
```

#### assertConsoleOutputContains

Assert that console standard output contains given output.

```python
print("Success !")
self.assertConsoleOutputContains("Success")
```

#### assertConsoleOutputMissing

Assert that console standard output does not contain the given output.

```python
print("Success !")
self.assertConsoleOutputMissing("hello")
```

#### assertConsoleHasErrors

Assert that something has been output to console standard error.

#### assertConsoleExactError

Assert that console standard error is equal to given error.

```python
print("An error occured !", file=sys.stderr)
self.assertConsoleExactError("An error occured !\n")
```

#### assertConsoleErrorContains

Assert that console standard error contains given error.

```python
print("An error occured !", file=sys.stderr)
self.assertConsoleErrorContains("error")
```
