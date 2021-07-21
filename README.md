# (5/5) Finally - home!

We are given a link 'https://s3.us-west-2.amazonaws.com/cyber-ctf.be/chl5/9f2d4057-618a-4016-a854-e6ed23d30b21.html'
The link opens a page with an input box:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/landing.png)

After trying to enter some random input I receive this sad face:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/sad-face.png)

So now, let's do what we should do with every web challenge! open the dev tools and try to enter another input.<br/>
Only now, as I've entered the input, I could see the debugger is being called, maybe to get my attention, maybe to ruin brute force attacks?

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/dev-tools.png)

Let's use the debugger and keep going!<br/>
Looking at the next break point I could see more of the logic

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/dev-tools-more.png)

So the var 'answer' is our input.<br/>
And the var 'a' is our input encoded.<br/>
And we can see the code saves the result of calling to console.log(...a) and then checks, if the result is 0 we win.<br/>

But something is weird, ```console.log``` simply logs data, it shouldn't return a value, right?<br/>
So I've decided to call the ```console.log``` function to see whats going on and indeed it behaved weirdly

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/checking-the-console.png)

So I've clicked on the ```console.log``` ref to see what is being called when we call ```console.log```, something must have replaced it

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/console-impostor.png)

I've found this WASM code there, disgusted with WASM I've immediately turned to google looking for some kind of WASM decompiler and I've found this:<br/>
https://github.com/WebAssembly/wabt/blob/main/docs/decompiler.md<br/>

After decompiling the WASM to a more readable format and converting it into some C code manually we can start to reverse this code.

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/code-example.png)

(I've also removed some of the globals as I've seen they are useless, I've forgot to screenshot this part I'm sorry :( )<br/>

So the function receives 31 args so that means our input string is 31 chars long.<br/>

So the first part of the function is simply declaring the local variables and declaring the counter 'em'.

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/declaring-vars.png)

the second part is a repetitive part that happens for each of the 31 args so I've determined that I can understand what happened with the first args<br/>
and apply this logic for the reset.<br/>

There was this repetitive function:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/abs.png)

That I realized simply takes the abs value of the passed value and after that added it to the counter 'em'.<br/>
So that means that if we want to get a value of zero at the end we must make sure each section for each arg is 0, otherwise the result will be bigger than zero<br/>
and we fail the condition.<br/>

Here is the code for a single arg after converting:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/arg-code.png)

1. So firstly it takes the supplied argument, lets call it 'a', and preforms xor with -1
2. then preforms & (and operation) with some number (in the example it's 167) lets call that number 'b'
3. then preforms & with the our variable 'b' after it xored 'b' with -1 (for example 167 ^ -1 = -168)
4. then preforms | (or operation) with the result of the 2. line and 3. line.
5. then it subtracts a third number, lets call it 'c'
6. lastly it preforms abs operation on the result of 5 and adds to the counter 'em'

we can represent this for all variables as this function:
```python
counter += abs(((a ^ -1) & b) | ((b ^ -1) & a) - c)
```
so we need 
```python
((a ^ -1) & b) | ((b ^ -1) & a) - c
```
to equal zero.
We can see that mathematically 
```python
((a ^ -1) & b) | ((b ^ -1) & a)
```
is the same as
```python
a ^ b
```
so that means:
```python
counter += abs((a ^ b) - c)
```
because we need ```a ^ b``` to equal 'c', we know that 'a' must be equal to ```b ^ c``` (because xor is a linear operation)

And now extracting all the b,c values we get this:

![alt text](https://raw.githubusercontent.com/GabiCtrlZ/ch5-cyberark/main/pictures/values.png)

And after running this code

```python
ans = [193 ^ 167, 32 ^ 16, 130 ^ 240, 192 ^ 159, 150 ^ 240, 52 ^ 65, 27 ^ 105, 69 ^ 49, 31 ^ 119, 63 ^ 12, 160 ^ 210, 51 ^ 108, 137 ^ 184, 133 ^ 235, 16 ^ 118, 4 ^ 52, 138 ^ 213, 102 ^ 5, 95 ^ 107, 91 ^ 106, 148 ^ 165, 83 ^ 12, 134 ^ 182, 239 ^ 220, 191 ^ 140, 162 ^ 149, 32 ^ 18, 144 ^ 168, 219 ^ 234, 149 ^ 173, 18 ^ 39]
ans = [chr(a) for a in ans]
print(''.join(ans))
```

We get the flag! f0r_furth3r_1nf0_c411_033728185<br/>

Or not? trying to enter this as the flag gives us an error? what?<br/>
So I've decided to call the number 033728185 and received this message:<br/>
'count down for landing 10cba9876543210 successful landing great job cadet! we understand you've found a message from extraterrestrial life,<br/>
please send that important flag to us'<br/>

I've remembered that in challenge number 2 we got a rust code and the flag was a extraterrestrial_msg! the flag there was 'What a lovely day'<br/>
so I've tried to send that as the flag and failed again.<br/>

So after banging my head against the wall, I've understood that the '10cba9876543210' in the recording from the phone must be a clue, so I took the 'What a lovely day' flag<br/>
and tried to encode in in base13 (because '10cba9876543210' is counting in base13) resulting in '69 80 76 8c 26 76 26 84 87 91 7a 84 94 26 79 76 94'<br/>

After entering this I've finally finished the challenge!