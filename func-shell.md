# Functional Shell (draft)

Design goals:

- Keep a minimum compatibility layer, so that users used to traditional shell can still use their basic commands as usual
- Have a functional scripting language more robust and easier to program than the shell's.


## Compatibility with traditional shells

    # works as usual
    echo "test" $PORT | cat >file
    apache -f /file &
    
Any line not starting with a reserver command keyword is a traditional shell command (with some limitations).

## Special constructs

Additional construct and commands provide access to functional programming.

### Typed variable

Variables are dynamically but strongly typed, with type coercion between primary types.

    var foo = "This is a string variable" # a string
    var bar = 9999.88  # a number
    
    echo $foo $bar # outputs "This is a string variable 9999.88"
    
### Asyncronous command calls

Background jobs are managed via promises, with a literal form [command].

    # equivalent constructs
    mycommand -v myoptions &
    [mycommand -v myoptions]

However, the promise form provides additional possibilities

    [mycommand -v myoptions].pipe(cat >myfile)
    [mycommand -v myoptions].output.split("\n").filter((x) -> x != "").pipe(cat);
    [mycommand -v myoptions].output.split("\n").filter((x) -> x != "") | cat
    
Methods of a Promise object are either streamed or blocking.

    [mycommand].errorCode # blocking until the errorCode is available    
    [mycommand].output    # blocking until the full command output is available
    [mycommand].stream(cat) # streams the output of background job to the command cat

In the following example, the shell starts two commands as background jobs (mycommand
and othercommand), and pipes the ouput of mycommand to the input stream of othercommand).
    [mycommand].stream([othercommand])
    
### Functions and lambdas

Functions are a type, and can be defined as lambdas. Note that they can be used as a replacement for traditional shell aliases.

    var func = (x) -> echo x
    var func = (x:String) -> {
        echo multiline function
        echo are not a problem
        echo $x
        echo x.capitalize()
    }
    func("a string")


    # Functional Aliases
    var func2 = [command arg %1 --tes %2]
    $func(un test, Env.get(PORT))  # execute func avec les arguments passés dans %1 et %2
    
    # currying ?
    var func3 = func2.args("test") # becomes [command arg "test" --tes %1] 
    var func3 = func2.args(, "test") # becomes [command arg %1 --tes "test"] 
    
    
### Global objects

Most of the predefined commands are in their own Object, acting as private namespace. Global objects always start with a capital letter.

- Global: namespace for global commands
- Env: for functions relative to the environement variables
- Object: for objects defined as key-value sets (same as javascript)
- String: for string objects
- Process: functions relatives to processes
- Array: a type for arrays
- Table: a type for tables of typed data
- Function: a type for functions
- Class: a type for classes
- URL: a type for URL objects
- Promise: for asynchronous execution of commands or functions

#### Examples

    var httpReq = URL.get("http://www.example.com/test"); # renvoie une promesse
    var res = httpReq.result  # bloque et renvoie le résultat
    var res = http.on(URL.SUCCESS, (data) -> echo data)
    
Note the strong typing:

    # Error: URL.on() expects an enum {URL.SUCCESS ou URL_FAIL) as first parameter
    URL.on("une string", (data) -> echo data) 

Some functions return promises

    var httpReq = URL.get("http://www.example.com/test"); # httpReq is a promise
    var res = httpReq.result  # blocks and returns the result
    var res = http.on(URL.SUCCESS, (data) -> echo data) # also with events


