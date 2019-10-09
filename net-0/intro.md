# Commands

## Run all virtual machines in same time
```bash
    lstart -o "--con0=-xterm" -d net-0
```

## Close all virtual machines
```bash
    lhalt
```

## Restart all running virtual machines
```bash
    lcrash  # Close and delete all disk (or with options)
    lstart
```

## More commands:
```bash
    man netkit
```

## Sources: [here](http://wiki.netkit.org/man/man7/netkit.7.html)