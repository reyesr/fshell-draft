# Functional Shell (draft)

Anti-design goals:

- Avoid painful shell filtering. ie in order to extract a pid or get the size of a volume

    ps -ef | grep XYZ | awk '{ print $2 }'
    
    space=`df -h | awk '{print $5}' | grep % | grep -v Use | sort -n | tail -1 | cut -d "%" -f1 -`

- Avoid medieval control flow constructs

    if [ "$num" -gt "150" ]
    

Design goals: mainly be what a modern shell should be

- Keep a minimum compatibility layer, so that users used to a traditional shell can still use their basic commands as usual
- Have a functional and familiar scripting language more robust and easier than the shell's.
- Provides tooling for easy system operations (start commands, manage processes, manage services)
- Advanced scripting with support for clean syntax and runtime errors, temporary files and directory, temporary and permanent storage (for basic key-value structures), PID and single-instance running of commands and functions, integrated internet protocols support natively available.
- Advanced command completion, and filesystem browsing
- Native support for complex data structure (xml, json, csv), with look up, walking, filters, and creation functions.

## Compatibility with traditional shells

    # works as usual
    echo "test" $PORT | cat >file
    apache -f /file &
    
Any line not starting with a reserved command keyword or literal is a traditional shell command (with some limitations).

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

    [mycommand -v myoptions].pipe(grep XYZ)
    [mycommand -v myoptions].output.split("\n").filter((x) -> x != "").pipe(cat);
    [mycommand -v myoptions].output.split("\n").filter((x) -> x != "") | cat
    
Methods of a Promise object are either streamed or blocking.

    [mycommand].errorCode # blocking until the errorCode is available    
    [mycommand].output    # blocking until the full command output is available
    [mycommand].stream(grep XYZ) # streams the output of background job to the command grep

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
    
### Flow control

The normal if/then/else, while, for, foreach functions are all available

    var s = "my string"
    if (s == "my string") { echo $s is ok } else { echo abort!! }
    var obj = Object.fromJson([cat <file])
    for (key: obj) {
        echo $key = $obj[key]
    }
    
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
- Class?: a type for classes
- URL: a type for URL objects
- Promise: for asynchronous execution of commands or functions

#### Examples

Note the strong typing:

    # Error: URL.on() expects an enum {URL.SUCCESS ou URL_FAIL) as first parameter
    URL.on("foobar", (data) -> echo data) 

Some functions return promises

    var httpReq = URL.get("http://www.example.com/test"); # httpReq is a promise
    var res = httpReq.result  # blocks and returns the result
    var res = http.on(URL.SUCCESS, (data) -> echo data) # also with events

