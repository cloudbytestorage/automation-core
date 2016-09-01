## What is automation-core ?

This is a DSL for scripting. To understand the details behind this project, 
please read the docs @ [design thoughts](https://github.com/CloudByteStorages/automation-core/tree/master/touchstone/DesignThoughts)

### Features

- No debug, no logging. The automation/core will tell the issues proactively.
- Execute commands via ssh
- Run the output over a chain of operators
- Store specific values as the chain of operators gets executed
- Use specific values as the chain of commands get processed
- Conditional execution of remote commands
 - i.e. execute based on outputs of previous commands

### DSL grammar:

- ssh (multiple variants)
- sampling for a fixed amount of times
- conditional sampling
- to
- run
- storeAs
- then (multiple variants)
- can use groovy operators in above grammar

### A sample input file (i.e. a dsl script with .groovy extension)

```groovy
def cythonMacOpts = {
    defaultHost = '20.10.112.21'
    defaultUser = 'root'
    defaultPassword = 'test123'
}

def elastistorMacOpts = {
    defaultHost = '20.10.48.140'
    defaultUser = 'root'
    defaultPassword = 'test'
}

def Map container = [:]

def automaton = {
    
   ...

    ssh([repeat: 3, interval: 4000]) {
        
        to(cythonMacOpts, 'Verify cython test cases by sampling at periodic intervals')
        
        run('cd /cthon04 && cat abc.out')
        
        storeAs('cython-sampler.output', container)
        
        repeatIf({ contains("No such file") })
        
    }

   ...
}
```

### Corresponding output

```shell
$ groovy GrCythonP5TestCase.groovy

{
    "sshRuns": [
        {
            ...
        },
        {
            ...
        },
        {
            ...
        },
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-10 16:02:55",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "cython-sampler.output": "cat: abc.out: No such file or directory\r\n",
            "allowRepeat": "true"
        },
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-10 16:03:01",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "cython-sampler.output": "cat: abc.out: No such file or directory\r\n",
            "allowRepeat": "true"
        },        
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-10 16:03:11",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "cython-sampler.output": "cat: abc.out: No such file or directory\r\n",
            "allowRepeat": "true"
        },
        {
            ...
        },
        {
            ...
        }
    ],
    "sshCount": 8
}
```

### sample input file - II

```groovy

def cythonMacOpts = {
    defaultHost = '20.10.112.21'
    defaultUser = 'root'
    defaultPassword = 'test123'
}

def elastistorMacOpts = {
    defaultHost = '20.10.48.140'
    defaultUser = 'root'
    defaultPassword = 'test'
}

def Map container = [:]

def automaton = {
    
   ...

    ssh([repeat: 3, interval: 4000]) {
        
        to(cythonMacOpts, 'Verify cython test cases by sampling at periodic intervals')
        
        run('cd /cthon04 && cat abc.out')
        
        storeAs('cython-sampler.output', container)
        
        repeatIf({ contains("Nooop such file") })
        
    }

   ...
}
```

### Corresponding output - II

```shell

$ groovy GrCythonP5TestCase.groovy
{
    "sshRuns": [
        {
            ...
        },
        {
            ...
        },
        {
            ...
        },
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-10 16:10:22",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "cython-sampler.output": "cat: abc.out: No such file or directory\r\n",
            "allowRepeat": "false"
        },
        {
            ...
        },
        {
            ...
        }
    ],
    "sshCount": 6
}
```
