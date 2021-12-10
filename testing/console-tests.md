You can test your [custom commands](/features/commands) running in console with `craft` test helper.

```python
def test_my_command(self):
    self.craft("my_command", "arg1", "arg2").assertSuccess()
```

This will programmatically run the command if it has been registered in your project and assert that no errors has been reported.

## Available Assertions

The following assertions are available when testing command with `craft`.

- [assertSuccess](#assertsuccess)
- [assertHasErrors](#asserthaserrors)
- [assertOutputContains](#assertoutputcontains)
- [assertExactOutput](#assertexactoutput)
- [assertOutputMissing](#assertoutputmissing)
- [assertExactErrors](#assertexacterrors)

### assertSuccess

Assert that command exited with code 0 meaning that it ran successfully.

```python
self.craft("my_command").assertSuccess()
```

### assertHasErrors

Assert command output has errors.

```python
self.craft("my_command").assertHasErrors()
```

### assertOutputContains

Assert command output contains the given string.

```python
self.craft("my_command").assertOutputContains(output)
```

### assertExactOutput

Assert command output to be exactly the same as the given reference output.
Be careful to add eventual `\n` line endings characters when using this assertion method.

```python
self.craft("my_command").assertExactOutput(output)
```

### assertOutputMissing

Assert command output does not contain the given reference output.

```python
self.craft("my_command").assertOutputMissing(output)
```

### assertExactErrors

Assert command output has exactly the given errors.

```python
self.craft("my_command").assertExactErrors(errors)
```
