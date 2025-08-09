# CHALLENGE 8: CMDi 6
This challenge aggressively filters dangerous characters like ;, &, |, $, and even backticks and parentheses. It seems like there’s no easy way to inject a command. After trying all the usual suspects and getting nowhere, I remembered that newlines (\n) can break a command into multiple lines — which the shell will happily interpret as separate commands.

To test it out, I injected:

/challenge\ncat /flag.txt

This made the shell see two lines:

ls -l /challenge cat /flag.txt

So the first line lists the directory like normal, and the second line prints the flag.

This works because the server doesn't filter newline characters, and shell=True means the shell will happily execute multiple lines. %0a (newline) acts like an invisible ENTER, injecting a second command even without any of the usual injection symbols.
