---
name: commit_message_staged
description: Creates a commit message for only staged files.
---
Every time this skill runs, check on disk using bash whether `tmp_commit_message.txt` exists and whether it has content; do not assume it exists or that it does not exist without checking.

Let's review all the files that are staged for commit, then create a commit message. Do not do the commit, I only want the commit message in a tmp_commit_message.txt file in the root of the project, if the file exists and has content, tell me there's an error and stop the execution. If the file doesn't exist or is empty, continue.

The commit message I want needs to be simple, human readable, it should explain what was done like explaining it to a boss. If the person needs more information they should go see the code.

The commit message should have this structure:

```
Tag line

- task 1
- task 2
...
- task n
````

Where Tag line is a line that can be checked in the log to identify the main idea for the commit. Below the tag line there should be a list of tasks **but** the list is not necessary if the commit fits only in the Tag line, only the Tag line is preferred over over explicit tasks.

Ask me if I approve the commit message in tmp_commit_message.txt, if I do, ask me if I want to do the commit and do it for me. After you're done, remove the file. Do not any mention about an agent helping doing this commit.
