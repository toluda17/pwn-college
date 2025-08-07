
I'm now starting the Web Security module on Pwn College.

In this section, I’ll be learning about common web vulnerabilities, how they work, and how to exploit them in controlled environments. I’m aiming to strengthen my understanding of how web applications can be attacked — and ultimately, how they can be better defended.

Unfortunately, for academic integrity reasons, I can't actually put down my code here but I'm going to write walkthroughs of the steps I took in solving each challenge.

Let’s begin.

# CHALLENGE 1: PATH TRAVERSAL 1
This challenge is about path traversal — basically tricking the server into reading files it’s not supposed to by messing with file paths. The webserver is supposed to only serve files from /challenge/files/, but the way it’s coded, it just slaps the user input straight onto that path and tries to open it.

The key part is this:

requested_path = app.root_path + "/files/" + path
return open(requested_path).read()

The path variable is whatever I put after /repository/ in the URL. Since the server doesn’t check if path tries to go up directories (like with ../), I can easily escape from /files/ and grab other files on the system.

I realized I could use ../ to move up the folder tree. So I requested:

GET /repository/../../flag.txt

which the server reads as /challenge/files/../../flag.txt → /flag.txt. That let me access the flag outside the intended folder.

# CHALLENGE 2: PATH TRAVERSAL 2
This challenge tries to stop path traversal by stripping leading dots and slashes from the user input path. However, the method used is pretty naive and doesn’t cover all cases.

I noticed that .strip("/.") only removes dots and slashes at the start and end of the string. So if I prepend a word before the ../, the strip doesn’t remove it anymore. For example, sending a path like:

GET /content/bypass../..//flag.txt

lets the ../.. stay intact in the middle, effectively moving up X+1 directories (one more than the number of ../ parts) because of how the path joins.

# CHALLENGE 3: CMDi 1
This challenge shows how passing user input directly to a shell command without sanitization can lead to command injection (CMDi). The server runs ls -l on whatever directory you provide via the target parameter.

Since the server runs the command using 'shell=True', I can inject extra shell commands by using special characters like ';'.
For example, I sent:

target=/challenge; cat /flag.txt

which runs: ls -l /challenge; cat /flag.txt so it lists /challenge and then outputs the flag.

This works because the server passes the user input directly to a shell command with shell=True, it executes everything, including injected commands separated by ;. There’s no sanitization or escaping, so CMDi happens.

# CHALLENGE 4: CMDi 2
This challenge tries to block command injection by removing ';' from the user input, but it still runs the command with shell=True. So I had to find another way to inject commands without using ';'.

I remembered that the pipe character '|' can chain commands too. It wasn’t filtered out, so I used it to pipe the output of ls into another command. For example:

ls -l /challenge | cat /flag.txt would list /challenge and then print the flag. Since the server just appends my input directly, I injected:

/challenge | cat /flag.txt

as the subdirectory parameter.

This works because the server runs the command in a shell and only removes ;, but not pipes (|). Pipes let me chain commands, so my injected command runs after the ls. This bypasses the naive filtering.

# CHALLENGE 5: CMDi 3
This challenge tries to block injection by wrapping my input in single quotes like this:

ls -l 'user_input'

The single quotes prevent special characters from working inside, because the shell treats everything inside quotes literally — until it finds a matching '. I realized I could break out of the single quotes by injecting a ' to close the current quote early, then add my malicious command, and finally fix the quotes so the whole command stays valid. For example, I sent:

' ; cat /flag.txt #
The resulting command looked like:

ls -l '' ; cat /flag.txt #'  

The first '' is just an empty argument (from the broken quotes). Then the ; cat /flag.txt runs my command. 
The # comments out the trailing ' to avoid syntax errors.

This works because the shell needs matching quotes, so I close the initial quote early with a ', inject my command, then comment out the leftover trailing quote with #. This tricks the shell into running my injected command safely.

# CHALLENGE 6: CMDi 4
This challenge runs a command like:

TZ=user_input date

where the user input sets the timezone environment variable before running the date command.
I figured out I could inject a command by ending the TZ= value early and then chaining my own command. Since it’s setting an environment variable, I could do something like:

TZ=foo; cat /flag.txt # This way, the shell runs:

TZ=foo; cat /flag.txt # date. The ';' ends the TZ= assignment, then my cat /flag.txt runs, and # comments out the rest (date) so it doesn’t cause errors.
This works because the shell runs the whole line, and TZ=foo; ends the environment variable assignment, letting me run arbitrary commands after it. The # comments out the rest so no syntax errors happen.

# CHALLENGE 8: CMDi 6
This challenge aggressively filters dangerous characters like ;, &, |, $, and even backticks and parentheses. It seems like there’s no easy way to inject a command.
After trying all the usual suspects and getting nowhere, I remembered that newlines (\n) can break a command into multiple lines — which the shell will happily interpret as separate commands.

To test it out, I injected:

/challenge\ncat /flag.txt

This made the shell see two lines:

ls -l /challenge
cat /flag.txt

So the first line lists the directory like normal, and the second line prints the flag.

This works because the server doesn't filter newline characters, and shell=True means the shell will happily execute multiple lines. %0a (newline) acts like an invisible ENTER, injecting a second command even without any of the usual injection symbols.
