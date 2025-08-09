# CHALLENGE 4: CMDi 2
This challenge tries to block command injection by removing ';' from the user input, but it still runs the command with shell=True. So I had to find another way to inject commands without using ';'.

I remembered that the pipe character '|' can chain commands too. It wasn’t filtered out, so I used it to pipe the output of ls into another command. For example:

ls -l /challenge | cat /flag.txt would list /challenge and then print the flag. Since the server just appends my input directly, I injected:

/challenge | cat /flag.txt

as the subdirectory parameter.

This works because the server runs the command in a shell and only removes ;, but not pipes (|). Pipes let me chain commands, so my injected command runs after the ls. This bypasses the naive filtering.
