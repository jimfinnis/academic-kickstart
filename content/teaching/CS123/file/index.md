+++
title = "The FileLogger class"
date=2020-12-14  # Add today's date.
type="unindexed" # stops it being indexed as "teaching", could be anything
+++
This is the code for a FileLogger class, a subclass
of the Logger abstract class introduced [here]({{< ref "../examples/2/" >}}) which
logs to files. 

To use it, you pass a filename for a log file into its constructor. When you've finished
with it, you must call its *shutdown()* method.

## Imports
We're going to be using the **FileWriter** and **IOException** classes from the **java.io**
package, so we need to import them.

```java
import java.io.FileWriter;
import java.io.IOException;
```

## Class declaration

Our class is called **FileLogger** and it's a subclass of **Logger**, so we can declare
it:

```java
public class FileLogger extends Logger {
```

## File handling

Because we're logging to a file, we're going to need to open the file once, write to it whenever
we want to log, and then close it when we're finished.
That means we are going to need to store information about the file inside each FileLogger object.
We'll use an instance variable here: a reference to a FileWriter object. We can then use that variable
to write to the file.

(Closing the file is rather more of a problem, and I'll come to that later.)

```java
    // an object representing an open file we are writing log
    // messages to

    FileWriter fw;
```

## Constructor

Now we can write the constructor for the class. First, this will call the superclass' constructor.
Looking at the source code for Logger, we can see that the constructor doesn't do anything at the 
moment - but that might change. Then we will create a new FileWriter and store it in the
instance variable we created earlier.

Creating the FileWriter might throw an exception - if we don't have permission to write to that filename,
for example. We can't handle this in the constructor, so we'll just say the constructor might throw
the exception and whatever code does "new FileLogger()" will have to deal with it. We'll look into
how to do that later.


```java
    // create a new file logger, opening a file and storing
    // a reference to it. This file will remain open for the
    // entire run of the program - remember to close it at
    // the end by calling close() on this logger.

    public FileLogger(String filename) throws IOException {
        super(); // call the superclass (Logger) constructor
        // try to create the file writer
        fw = new FileWriter(filename);
    }
```

## Closing the logger
I mentioned earlier that closing the file is a problem. Ideally, we would like the FileLogger to close
its file when it is no longer is use, and that means we have to add an extra method to tell the logger
when we have finished with it.

The code will set the FileWriter instance variable to null once it has closed the FileWriter. This
will let us check that variable to see if we have closed it already, to stop us doing it twice or
trying to log afterwards.

We'll also have to handle a possible IOException again, but here I'll just print a message. It's
probably not a complete disaster if the close fails.

```java
    // call this at the end of the program to close the output file
    // and set it to null (so we can check it has been closed)

    public void close(){
        try {
            if (fw!=null) { // if file writer is opened
                fw.close(); // close it 
                fw = null;  // and set to null
            }
        } catch (IOException ioe) {
            // oops, the close went wrong!
            System.err.println("IOException: " + ioe.getMessage());
        }
    }
```    

## The performLog() method
Now we can write the important *performLog()* method. Remember, this is the method that makes different
kinds of Logger behave differently - and it's the method that does the actual logging.
It will:
* check that the FileWriter instance variable isn't null (which it will be if we there was a problem opening
the FileWriter or we have called *close()* on our FileLogger), 
* if it isn't null, it will write the message to the file
* and then "flush" the file (see below).
If anything goes wrong in this process an exception will happen. We'll catch this exception, and
print the message we were trying to log to the console (it's better than nothing) and also the actual
exception.

```
    // actually perform the logging, writing the message to the open
    // file and "flushing" the writer, to guarantee the message is
    // stored on the disk. If we don't do this, and we forget to call
    // shutdown(), the final messages might never make it into the
    // actual disk file.

    @Override
    protected void performLog(String message) {
        try
        {
            if(fw!=null){           // if FileWriter is open
                fw.write(message);  // write the message to the file
                fw.flush();         // flush the file
            }
        }
        catch(IOException ioe)
        {
            // something went wrong, print the message
            System.err.println("Attempted to log: "+message);
            // and also whatever it was that went wrong!
            System.err.println("IOException: " + ioe.getMessage());
        }
    }
}
```
### Why flush the file?
Flushing a file makes sure the entire file is written to the disk. Sometimes the operating system
keeps a block of data to be written - a "buffer" - and only writes the buffer to the disk when
it is full (typically a few kilobytes) or when you close the file.
This stops the disk constantly doing tiny write operations
of a few bytes only, which is very slow. All those tiny write operations will be combined into one
big write operation.

Unfortunately this means that if you don't close the file properly, you can lose the last few things
you wrote because the buffer never gets written to disk. Normally, the operating system will detect this
and will flush for you, but it's always a good idea to be sure.

Here, we're flushing after every log message. That has the advantage of making sure every message is actually
written to the disk, but might slow things down because we're forcing the system to do a lot of tiny writes!

## Usage

Here is an example of using both the FileLogger and the ConsoleLogger. Perhaps
*obj1* is a really important task where we want all the log data to be stored in 
a file, while *obj2* and *obj3* are less important:

```java
public class Main {
    public static void main(String args[]){
        // create loggers
        FileLogger fileLogger = new FileLogger();
        Logger consLogger = new ConsoleLogger();
        
        // create some objects using the loggers - obj1 uses
        // the file logger, the others use the console logger.
        
        SomeClass obj1 = new SomeClass(fileLogger);
        SomeClass obj2 = new SomeClass(consoleLogger);
        SomeClass obj3 = new SomeClass(consoleLogger);
        
        // add some code here which actually does things with those
        // objects!
        
        // when we're finished, shut down the file logger

        fileLogger.close();
    }
}
```
### Question

Look at these lines:
```java
        FileLogger fileLogger = new FileLogger();
        Logger consLogger = new ConsoleLogger();
```
Why is one variable declared as FileLogger, while the other one is a generic Logger?
Read all the code and try to work this out.


{{% spoiler id="0" text="click to show the answer" %}}

We need to be able to close the logger. If the *fileLogger* variable were just a plain Logger
we would not be able to call the *close()* method, because Logger does not have that method. Even
though the variable is actually pointing to a FileLogger, the program doesn't know that.

As far as the program is concerned, *fileLogger* is a Logger. It might be a ConsoleLogger, it might
be a FileLogger, but it doesn't know, so we can only call the methods we know Logger has.

The only way we can fix this is to tell the program what kind of Logger the variables is:
make *fileLogger* actually a FileLogger, so we can call *close()*
on it.

{{% /spoiler %}}