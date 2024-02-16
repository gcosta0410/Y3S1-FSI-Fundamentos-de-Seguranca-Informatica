## Format String Attack

### Task 1
In order to crash the program, we need to be able to stop or halt it's execution somehow. We know that there's one printf() function with a format string vulnerability, which we can explot in order to achieve this goal. <br>
As a first test, we decided to run the build_string.py script to generate the bad file with the string we want use and send that string to the server. <br>
This worked, but we were a bit confused as to why it did.

<img src="./images/6/1.png">

Afterall, on the surface, it only looks like we were sending a random string with a few "%.8x" and a "%n" at the end. <br>
We didn't know what the "%n" did, but we knew the "%.8x" part would just print a bunch of random strings with 8 hexadecimal characters each, so we assumed the "%n" would give us something along those lines. <br>

<img src="./images/6/2.png">

Upon further investigation, we found out that "%n" is a string formater that takes a pointer as an argument and writes on that memory positon the amount of characters written or read by the functions printf and scanf, respectively, so we decided to send only the "%n" character to the program and it still worked. <br>

<img src="./images/6/3.png">

We don't know the exact inner workings, but our best informed guess is that this "%n" is either overwriting something needed for program execution, like the return address of the function that calls the myprintf() function, causing it to crash or attempting to write on the stack without permission to do so. (After task 3, we're more inclined to the first one).

### Task 2

#### Task 2a

In order to print the first 4 bytes of our input, we first need to define which input we're going to search for. In our case, we chose the hex string "DEADBEEF". <br> 
Afterwards, we need to calculate how many "%x" format specifiers are needed in order to get to our string. <br>
We did this via trial and error: first, we chose a big arbitrary number, like 100, and ran the program, finding our string on the output. Then we chose another number we thought would be close to the real number, like 50, and repeated the process. Continuing this process we eventually got 64 as an answer.

<img src="./images/6/4.png">

#### Task 2b

In order to read the secret message, all we have to do is insert it's address (already given to us by the server) at the beginning of our payload and repeat the process of part a) of this task with a minor change: Instead of printing out 64 hexadecimal values, we'll print 63 of them and since the 64th is our secret message's address, if we pass that address to a "%s" format specifier, it will read the contents from there.

<img src="./images/6/5.png">

### Task 3

#### Task 3a and 3b

Task 3b includes task 3a, so we'll explain both under the same section. <br>

In order to rewrite a stack value we can use the "%n" format specifier, that takes a pointer and writes the number of printed characters on that address.<br>

In order to rewrite the target variable's value, we would have to give it's address (given to us by the server) to this "%n". We can achieve this using the same method we used for task 2. <br>

After that, we just have to make sure all the printed chars add up to 0x5000 (20480 in decimal). The best way to do so is to divide 20480 by 63, which gives us 325.079, meaning we have to print x62 325 char long hex strings and x1 326 char long hex string (instead of 330, because we need to remove the 4 bytes of our addr string). Since the "%x" format specifier allows us to modify the amount of hex chars we want to print we can just add "%.325x" * 62 + "%.326x" to our string. 

<img src="./images/6/6.png">

## Week 6 CTF Challenge
### Challenge 1

First we started by running checksec on the given program, which yielded the follwing results:

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041738126101581835/image.png">

- RELRO: Partial RELRO -> Means that the Global Offset Table is moved in memory. This does not prevent format string attacks.
- Stack: Canary Found -> Means that there is a canary in the stack, which will make it harder to perform buffer overflow attacks.
- NX: NX Enabled -> Means that DEP is enabled.
- PIE: No PIE -> Means that ASLR is disabled.

Upon running checksec, we analyzed the given source code and answered the proposed questions:

- Qual é a linha do código onde a vulnerabilidade se encontra? 
    - Line 27 ( printf(buffer); ) is vulnerable to format string attacks.
- O que é que a vulnerabilidade permite fazer?
    - Write or read from memory.
- Qual é a funcionalidade que te permite obter a flag?
    - Sending a payload with format specifiers in it allows us to read or even write on memory. 

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041779399223169144/image.png">

Afterwards, we gave execute permissions to the exploit_example.py script and attached a gdb instance to the program's pid.

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041749863429316640/image.png">

Since we want to read the value of a global variable and ASLR is disabled, we can use gdb to grab it's address (0x804c060) so we can later on build the payload with it. 

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041750034154258564/image.png">

Now that we have the address of the variable, we can start calculating the offset of the start of the buffer that's storing our malicious input. Turns out the buffer is right after the vulnerable printf() statement.

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041747769104269392/image.png">

Since the buffer is straight after the printf() statement, placing the address of the variable we want to read at the start of the buffer followed by a single "%s" format specifier will allow us to read the contents of this variable.

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041748217735430235/image.png">

Now we only have to run the exploit_example.py script outside local mode in order to get the flag. (flag{638c3092c156c1c73052fa3289a81106})

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041750756279201934/image.png">

### Challenge 2

Similar to the last challenge, we ran checksec on the given program. Since the results were the same, we'll refrain from explaining them again.

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041784507180781609/image.png">

Analyzing the source code we got the answers to the following questions:
- Qual é a linha do código onde a vulnerabilidade se encontra? E o que é que a vulnerabilidade permite fazer?
    - Line 14 ( printf(buffer); ) is vulnerable to format string attacks, which allows us to write or read from memory.
- A flag é carregada para memória? Ou existe alguma funcionalidade que podemos utilizar para ter acesso à mesma?
    - The flag is never loaded into memory. The way to access it is via a shell that's opened in the code via a system() call.
- Para desbloqueares essa funcionalidade o que é que tens de fazer?
    - By altering the value of the key variable to "0xbeef" we can enter the if statement that opens the shell, allowing us to read the flag.txt file

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041784706976464927/image.png">

Now that we know what we need to do, we can follow the same procedure we used in the last challenge to obtain the address of the key variable (0x804c034).

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041786851809312819/image.png">

Upon finding the address we need for the payload, we, once again, tried to find the offset of the start of the buffer so that we can prepare our payload. Once again, it's right next to the printf() statement.

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041789223096815626/image.png">

Since this time we need to change the value of a variable, we need to use the %n format specifier. <br>
The intented value for the variable is 0xbeef, the same as 48879 in decimal form, which means we neeed to print 48879 chars before writing to the memory block that holds the key variable's value. <br>
Because a single format specifier will start reading into the buffer, we can't put the address of the variable right at the start, or else we won't have the "space" to write the amount of characters needed for the exploit. <br>
This means that we'll need to write something random first, then the address of the variable and only then we can start processing our very own input. <br>
In this case we chose to write a 4 byte string, "0000" (30303030 in hexadecimal), before our 4 byte address, which means that we'll be printing 8 chars, leaving us with 48871 chars left to be written, which we can do so with a "%.48871x" format specifier. <br>
Having ensured exactly "0xbeef" chars have been written to the stdout, we can now overwrite the value of the key variable using a "%n" format specifier. <br>
Executing the script confirms our procedure was correct, since we got the "Backdoor" message and a shell open. <br>
Now all we have to do is print the contents of the flag.txt file, giving us the desired flag(flag{500c226130f2e19c8407dcee589a0419}).

<img src="https://cdn.discordapp.com/attachments/1021902913079103488/1041788573373972520/image.png">

