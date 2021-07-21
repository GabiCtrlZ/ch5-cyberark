# (5/5) Finally - home!

We are given a link 'https://s3.us-west-2.amazonaws.com/cyber-ctf.be/chl5/9f2d4057-618a-4016-a854-e6ed23d30b21.html'
The link opens a page with an input box:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/landing.png)

After trying to enter some random input I receive this sad face:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/sad-face.png)

So now, let's do what we should do with every web challenge! open the dev tools and try and enter another input.
Only now as I've entered the input I could see the debugger is being called, maybe to get my attention, maybe to ruin brute force attacks.

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/dev-tools.png)

Let's use the debugger and keep going!
Looking at the next break point I could see more of the logic

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/dev-tools.png)

So the var 'answer' is our input.
And the var 'a' is our input encoded.
And we can see the code saves the result of calling to console.log(...a) and then checks, if the result is 0 we win.

But something is weird, console.log simply logs data, it should return a value, right?
So I've decided to call the console.log function to see whats going on and indeed it behaved weirdly

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/checking-the-console.png)

So I've clicked on the console.log ref to see what is being called when we call console.log, something must have replaced it

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/console-impostor.png)

I've found this WASM code there, disgusted with WASM I've immediately turned to google looking for some kind of WASM decompiler and I've found this:
https://github.com/WebAssembly/wabt/blob/main/docs/decompiler.md

So after decompiling the WASM to a more readable format and converting it into some C code manually we can start to reverse this code.

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/code-example.png)

(I've also removed some of the globals as I've seen they are useless, I've forgot to screenshot this part I'm sorry :( )

So the function receives 31 args so that means our input string is 31 chars long.

So the first part of the function is simply declaring the local variables and declaring the counter 'em'.

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/declaring-vars.png)

the second part is a repetitive part that happens for each of the 31 args so I've determined that I can understand what happened with the first args
and apply this logic for the reset.

There was this repetitive function:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/abs.png)

That I realized simply takes the abs value of the passed value and after that added it to the counter 'em'.
So that means that if we want to get a value of zero at the end we must make sure each section for each arg is 0, otherwise the result will be bigger than zero
and we fail the condition.

Here is the code for a single arg after converting:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/arg-code.png)

1. So firstly it takes the supplied argument, lets call it 'a', and preforms xor with -1
2. then preforms & (and operation) with some number (in the example it's 167) lets call that number 'b'
3. then preforms & with the our variable 'b' after it xored 'b' with -1 (for example 167 ^ -1 = -168)
4. then preforms | (or operation) with the result of the 2. line and 3. line.
5. then it subtracts a third number, lets call it 'c'
6. lastly it preforms abs operation on the result of 5 and adds to the counter 'em'

we can represent this for all variables as this function
counter += abs(((a ^ -1) & b) | ((b ^ -1) & a) - c)
so we need ((a ^ -1) & b) | ((b ^ -1) & a) - c to equal zero.
We can see that ((a ^ -1) & b) | ((b ^ -1) & a) is the same as a ^ b
so 
counter += abs((a ^ b) - c)

we need a ^ b to equal c so we know that a must be equal to b ^ c

and now extracting all the b,c values for each arg and preforming the xor operation on all of them we are left with the flag




