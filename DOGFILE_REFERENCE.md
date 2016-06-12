# Dogfile Format Reference

This is a work in progress, almost none of this is implemented in `dog` yet. This document is a draft of the Dogfile format.

[dog](https://github.com/xsb/dog) is a command line application that executes tasks. It's the first tool that uses Dogfiles and is developed at the same time as the Dogfile format itself.

## File Format

Dogfiles are [YAML](http://yaml.org/) files that describe the execution of automated tasks. The root object of a Dogfile is an array of map objects. These maps are called Tasks, here you can see an example of a Dogfile with two simple Tasks:

```yml
- task: hello
  description: Say Hello
  run: echo hello

- task: bye
  description: Say Good Bye
  run: echo bye
```

Multiple Dogfiles in the same directory are processed together as a single entity. Although the name `Dogfile.yml` is recommended, any file with a name that starts with `Dogfile` and includes valid (following this standard) YAML syntax is a Dogfile.

## Task definition

The task map accepts the following keys.

### task

Name of the task. A string made of lowercase characters (a-z), may contain hyphens (-).

```yml
- task: deploy
```

### description

Description of the task.

```yml
  description: Create Docker container
```
### run

The code that will be executed.

```yml
  run: npm install
```

Multiline scripts are supported.

```yml
  run: |
    echo "This is the Dogfile in your current directory:"

    for dogfile in `ls -1 Dogfile*`; do
      cat $dogfile
    done
```

### type

The default execution type is `sh` on UNIX-like operating systems and `cmd` on Windows, but other execution types will be supported.

```yml
  type: ruby
  run: |
    hello = "Hello Dog!"
    puts hello
```

### pre

Pre-hooks execute other tasks before starting the current one.

```yml
  pre: test
```

Multiple consecutive tasks can be executed as pre-hooks. The tasks defined in the following array will be executed in order, one by one.

```yml
  pre:
    - get-dependencies
    - compile
    - package
```

### post

Post-hooks are analog to pre-hooks but they are executed after current task finishes its execution.

```yml
  post: clean
```

They also accept arrays.

```yml
  post:
    - compress
    - upload
```

### workdir

Sets the working directory for the task.

```yml
  workdir: app
```

### tag

Tags are used to group similar tasks.

```yml
  tags: dev
```

Multiple tags are allowed.

```yml
  tags:
    - build
    - dev
```

Some special tags are provided. Hidden tasks are useful when we have tasks that are only executed as pre or post hooks but we don't want to show them in our task list.

```yml
  tags: hidden # Hide this task from the list
```

We can also tag our most important tasks to be highlighted at the top of the list in a separated group.

```yml
  tags: top # Show this task at the top of the list
```

### parameters

Additional parameters can be provided to the task that will be executed. Only works with shell-based executor types.

```yml
  parameters:
    # Variable with default value
    - name: planet
      default: Earth
    # Required variable without default value
    - name: city
      required: true
    # Variable with an array of allowed values
    - name: animal
      values:
        - dog
        - cat
        - human
  run: echo "I am a $animal that lives in $city, Planet $planet"
```

### register

Registers store the output of executed commands so chained tasks (using pre or post hooks) can process the output later.

```yml
  task: remote-whoami
  description: Check User in remote system
  run: ssh remote-host-example.com whoami
  register: remoteUser

  task: print-remote-user
  description: Print remote Username
  pre: remote-whoami
  run: echo "I am $remoteUser when I ssh into remote-host-example.com"
```

### Non standard keys

Optional keys that are not part of the Dogfile Format. Tools using Dogfiles and having special requirements can use their own keys that will be ignored by the tools that only follow the standard.

Any parameter starting by `x_` will simply be ignored.

```yml
- task: clear-cache
  description: Clear the cache
  x_path: /task/clear-cache
  x_tls_required: true
  run: ./scripts/cache-clear.sh
```