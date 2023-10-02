# godot-optional
Introduces to Godot Option and Result types inspired from Rust

## Option
A generic `Option<T>`

Options are types that explicitly annotate that a value can be `null`, and forces the user to handle the exception

Basic usage:
```gdscript
# By returning an Option, it's clear that this function can return null, which must be handled
func get_player_stats(id: String) -> Option:
    return Option.None() # Represents a null
    return Option.Some( data ) # Sucess!

var res: Option = get_player_stats("player_3")
if res.is_none():
    print("Player doesn't exist!")
    return

var data = res.expect("Already checked if None or Some above") # Safest
var data = res.unwrap() # Crashes if res is None. Least safe, but quick for prototyping
var data = res.unwrap_or( 42 ) # Get from default value
var data = res.unwrap_or_else( some_complex_function ) # Get default value from function
var data = res.unwrap_unchecked() # It's okay to use it here because we've already checked above
```

Option also comes with a safe way to index arrays and dictionaries
```gdscript
var my_arr = [2, 4, 6]
print( Option.arr_get(1))  # Prints "4"
print( Option.arr_get(4))  # Prints "None" because index 4 is out of bounds
```


## Result
A generic `Result<T, E>`

Results are types that explicitly annotate that an operation (most often a function call) can fail, and forces the user to handle the exception

In case of a success, the `Ok` variant is returned containing the value returned by said operation
In case of a failure, the `Err` variant is returned containing information about the error.

Basic usage:
```gdscript
# By returning a Result, it's clear that this function can fail
func my_function() -> Result:
    return Result.from_gderr(ERR_PRINTER_ON_FIRE)
    # Also supports custom error types!
    return Result.Err( Error.new(Error.MyCustomError).info("expected", "some_value") )
    return Result.Err("my error message")
    return Result.Ok(data) # Success!

var res: Result = my_function()
# Ways to handle results:
if res.is_err():
    res.stringify_error() # @GlobalScope.Error to String
    # Custom errors can bear extra details. See the "Custom error types" section below
    res.err_cause(...) .err_info(...) .err_msg(...)
    return

var data = res.expect("Already checked if Err or Ok above") # Safest
var data = res.unwrap() # Crashes if res is Err. Least safe, but quick for prototyping
var data = res.unwrap_or( 42 ) # Defaults to 42
var data = res.unwrap_or_else( some_complex_function )
var data = res.unwrap_unchecked() # It's okay to use it here because we've already checked above
```

Result also comes with a safe way to open files and parse JSON

```gdscript
# "Error" refers to custom error types. Not to be confused with @GlobalScope.Error
 var res: Result = Result.open_file("res://file.txt", FileAccess.READ) # Result<FileAccess, Error>
 var json_res: Result = Result.parse_json_file("res://data.json") # Result<data, Error>
```

## Custom error types
Godot-optional introduces a custom `Error` class for custom error types. 

The aim is to allow for errors to carry with them details about the exception, leading to better error handling. 
It also acts as a place to have a centralized list of errors specific to your application, as Godot's global Error enum doesn't cover most cases. 

Usage:
```gdscript
# Can be made from a Godot error, and with optional additional details
var myerr = Error.new(ERR_PRINTER_ON_FIRE) .cause('Not enough ink!')
    # Or with an additional message too
    .msg("The printer gods demand input..")

# Prints: "Printer on fire { "cause": "Not enough ink!", "msg": "The printer gods demand input.." }"
print(myerr)

# You can even nest them!
Error.from_gderr(ERR_TIMEOUT) .cause( Error.new(Error.Other).msg("Oh no!") )

# Used alongside a Result:
Result.Err( Error.new(Error.MyCustomError) )
Result.open_file( ... ) .err_msg("Failed to open the specified file.")
```

You can also define custom error types in the Error script
```gdscript
# res://addons/optional/Error.gd
enum {
    Other,
    # Define custom errors here ...
    MyCustomError,
}
```
