# CHALLENGE 3: CMDi 1
This challenge shows how passing user input directly to a shell command without sanitization can lead to command injection (CMDi). The server runs ls -l on whatever directory you provide via the target parameter.

Since the server runs the command using 'shell=True', I can inject extra shell commands by using special characters like ';'. For example, I sent:

target=/challenge; cat /flag.txt

which runs: ls -l /challenge; cat /flag.txt so it lists /challenge and then outputs the flag.

This works because the server passes the user input directly to a shell command with shell=True, it executes everything, including injected commands separated by ;. There’s no sanitization or escaping, so CMDi happens.
