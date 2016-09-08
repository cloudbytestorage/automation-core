## What is automation-core ?

This is a DSL for scripting. 

## Why ?

I saw a dire need to automate use-cases fast minus errors. I saw my colleagues resorting to programming
and learning the essence of programming, designing their automation scripts, and so on. I felt there was 
a gap to execute the test cases & come out with the results at a pace required by the organization. I could feel
this as another dev project. Obviously, they had limited their choice of tools to a single programming language.

## Details

To understand the details behind this project, 
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

### It has been done in Ansible, Fabric, etc libs (in python world). Why re-invent ?

> I agree this, may not be production grade. However, It has been my belief that pure
> DSL can not accomplish advanced business needs that may require parallelsim, state 
> management, flow control etc. If we resort to scripting i.e. code using various 
> languages, we lose our valuable contributors in form of admins, support, qa, business
> analysts, etc. We understand we cannot be dependent on dev folks for all the times.
> Hence, this is a hybrid solution that tries to accomplish this, via Groovy 
> (a highly potential but less celebrated language).

<br />

### Another try to help you align with my thoughts !!!

> If you believe accomplishing a task by implementing simple functions than writing 
> elaborate packages & / or classes, then you are with me. Feeling at home with AWS lambda ?
> I also agree that achieving this solution can be possible via pure functional languages. 
> You are also right if you believe in building suitable LISP macros or utilizing unix pipes, 
> awk & others as an alternative to this Groovy solution. Finally, it has to be a combination 
> of various tools, functional languages & **literate programming practices** which can enable all
> the non-dev folks to be a part of automation !!!

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
    
    ssh
    
    ssh {
        to(elastistorMacOpts, 'Remove lock file @ elastistor')
        
        run('rm /tenants/f7ec4eb1486c3aa6a4bafaa12d93e084/PoolRaidz1/Account1TSM1/amit.has')
        
    }

    ssh('task-002') {
        to(elastistorMacOpts, 'Restart lockd service @ elastistor')
        
        run('restart lockd')
        
        storeAs('task-002.output', [])
    }

    ssh {
        to(cythonMacOpts, 'Trigger cython test cases')
        
        run('cd /cthon04 && pwd')
        
        storeAs('task-003.output', container)

        then('is it cthon05', { contains("cthon05") ? "'$it' contains cthon05." : "'$it' does not have cthon05." })

        then('verify if numeric', { isNumber() ? "'$it' is a number." : "'$it' is a string." })

        then('verify if empty', { isEmpty() ? "'$it' is an empty thing." : "'$it' is a filled stuff." })
        
        storeAs('task-003.output2', container)
        
    }

    ssh([repeat: 4, interval: 10000]) {
        to(cythonMacOpts, 'Verify cython test cases by sampling at periodic intervals')
        
        run('cd /cthon04 && cat abc.out')
        
        then { container.get('task-003.output').contains("cthon04") ? "Yes. It is cthon04." : "No. It is not cthon04." }
    }

    ssh('task-005') {
        to(elastistorMacOpts, 'Enable lock file @ elastistor')
        
        run('touch /tenants/f7ec4eb1486c3aa6a4bafaa12d93e084/PoolRaidz1/Account1TSM1/amit.has')
        
        storeAs('task-005.output', container)
    }

    ssh('task-006', !container.get('task-005.output').contains('No such file')) {
        
        to(elastistorMacOpts, 'Restart lockd service @ elastistor')
        
        run('restart lockd')
    }
}
```

<br/ >

### Corresponding output

```shell

$ groovy GrCythonP5TestCase.groovy
{
    "uuid": "2016-06-06 16:24:05",
    "status": "Failed",
    "warnings": [
        {
            "msg": "Invalid 'ssh' property used.",
            "suggest": "Did you mean to use 'ssh' as a function ?"
        }
    ],
    "sshRuns": [
        {
            "threadName": "main",
            "output": "rm: /tenants/f7ec4eb1486c3aa6a4bafaa12d93e084/PoolRaidz1/Account1TSM1/amit.has: No such file or d
irectory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "rm /tenants/f7ec4eb1486c3aa6a4bafaa12d93e084/PoolRaidz1/Account1TSM1/amit.has",
            "uuid": "2016-06-06 16:24:05",
            "startTime": "x:y:z",
            "name": "Remove lock file @ elastistor"
        },
        {
            "uuid": "task-002",
            "status": "Failed",
            "warnings": [
                {
                    "msg": "Invalid 'storeAs' function used.",
                    "suggest": "Either 'storeAs' is an invalid function or its arguments are invalid !"
                }
            ]
        },
        {
            "threadName": "main",
            "output": "/cthon04\r\n",
            "exitStatus": 0,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && pwd",
            "uuid": "2016-06-06 16:24:07",
            "startTime": "x:y:z",
            "name": "Trigger cython test cases",
            "task-003.output": "/cthon04\r\n",
            "is it cthon05": "'/cthon04\r\n' does not have cthon05.",
            "verify if numeric": "''/cthon04\r\n' does not have cthon05.' is a string.",
            "verify if empty": "'''/cthon04\r\n' does not have cthon05.' is a string.' is a filled stuff.",
            "task-003.output2": "'''/cthon04\r\n' does not have cthon05.' is a string.' is a filled stuff."
        },
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-06 16:24:09",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "operation 16:24:09": "Yes. It is cthon04."
        },
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-06 16:24:20",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "operation 16:24:20": "Yes. It is cthon04."
        },
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-06 16:24:31",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "operation 16:24:31": "Yes. It is cthon04."
        },
        {
            "threadName": "main",
            "output": "cat: abc.out: No such file or directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "cd /cthon04 && cat abc.out",
            "uuid": "2016-06-06 16:24:43",
            "startTime": "x:y:z",
            "name": "Verify cython test cases by sampling at periodic intervals",
            "operation 16:24:43": "Yes. It is cthon04."
        },
        {
            "threadName": "main",
            "output": "touch: /tenants/f7ec4eb1486c3aa6a4bafaa12d93e084/PoolRaidz1/Account1TSM1/amit.has: No such file o
r directory\r\n",
            "exitStatus": 1,
            "exception": null,
            "endTime": "x1:y1:z1",
            "timeTaken": "x secs",
            "command": "touch /tenants/f7ec4eb1486c3aa6a4bafaa12d93e084/PoolRaidz1/Account1TSM1/amit.has",
            "uuid": "task-005",
            "startTime": "x:y:z",
            "name": "Enable lock file @ elastistor",
            "task-005.output": "touch: /tenants/f7ec4eb1486c3aa6a4bafaa12d93e084/PoolRaidz1/Account1TSM1/amit.has: No su
ch file or directory\r\n"
        },
        {
            "uuid": "task-006",
            "status": "Failed",
            "warnings": [
                {
                    "msg": "Can not run as condition is not satisfied."
                }
            ]
        }
    ],
    "sshCount": 9
}
```
