SSH into machine without entering password

While I'm developing the Microsoft.PowerShell.IoT module, I found myself constantly copying the build results into my RaspberryPi, so that I could test the changes.
This is specially painful because I can't test the module on my laptop, since this module interacts with specific hardware such has GPIO.

The typical scenario is:

1. Make changes on code
1. run `dotnet build` - Build the dotnet solution
1. `scp -r .\src\Microsoft.PowerShell.IoT\bin\Debug\netcoreapp2.0\linux-arm\publish\ pi@192.168.1.90:/home/pi/PowerShell.IoT` - Copy the output of the build to the Raspberry

Since the copy is done via `scp`, I'll be prompted for the password.
Let's see a way to avoid entering the password everytime.

# Disclosure - I'm by no means an expert on the SSH topic, so do some research on your own to make sure you understand what I mean here and if it makes sense

SSH is a protocol that allows you to access another machine to either access the command line, execute some code, copy files etc.
For that, there's (at least) one client and one server.
The server can be seen as the machine to which you will access (the pi in this scenario), and the client is the machine which will ask to access (the laptop).

SSH uses a set of keys: a public key and a private key. 
In a (extremely) simplified explanation, when you try to access a machine, you are also giving your public key. Then the target machine says "hum ok, that means nothing to me, you still have to prompt for the password for the user you are trying to access" (we will see in a moment why is this public key important).


