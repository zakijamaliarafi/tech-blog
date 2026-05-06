---
title: 'Bash Scripting Basics for Automation'
description: 'Automate repetitive tasks with simple Bash scripts.'
pubDate: 'May 10 2026'
---

If you find yourself running the same sequence of Linux commands repeatedly, it is time to write a Bash script. Bash (Bourne Again Shell) is the default command-line interpreter for most Linux systems, and scripting allows you to automate complex tasks.

## Creating Your First Script

A Bash script is simply a plain text file containing a list of commands.

1.  **Create a file:** Create a new file, typically with a `.sh` extension (though the extension isn't strictly necessary).
    ```bash
    touch myscript.sh
    ```
2.  **Add the Shebang:** Open the file in an editor and add `#!/bin/bash` as the very first line. This tells the system to use the Bash interpreter to execute the file.
3.  **Write your commands:** Add the commands you want to run.
    ```bash
    #!/bin/bash
    echo "Starting backup process..."
    tar -czf backup.tar.gz /home/user/documents
    echo "Backup complete!"
    ```

## Making It Executable

Before you can run your script, you need to grant it execute permissions using the `chmod` command:
```bash
chmod +x myscript.sh
```

Now you can run it by specifying its path:
```bash
./myscript.sh
```

## Variables and Control Structures

Bash scripting becomes powerful when you introduce variables and logic.

**Variables:**
```bash
#!/bin/bash
BACKUP_DIR="/var/backups"
echo "Saving to $BACKUP_DIR"
```

**If/Else Statements:**
```bash
#!/bin/bash
if [ -d "/var/www/html" ]; then
  echo "Web directory exists."
else
  echo "Web directory not found!"
fi
```

**For Loops:**
```bash
#!/bin/bash
for file in *.txt; do
  echo "Processing $file"
done
```

Starting with simple scripts and gradually adding logic is the best way to master Linux automation.
