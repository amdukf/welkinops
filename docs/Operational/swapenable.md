# Enable Swap

Check if swap is already enabled
```
swapon --show
```

You can also check with:
```
free -h
```

Create a swap file
```
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
```

Set the correct permissions
```
sudo chmod 600 /swapfile
```

Set up the swap area
```
sudo mkswap /swapfile
```

Enable the swap
```
sudo swapon /swapfile
```

Verify it’s active
```
swapon --show
free -h
```

Make it permanent
```
sudo nano /etc/fstab
/swapfile none swap sw 0 0
```

The swappiness parameter tells Linux how aggressively it should use swap instead of RAM.
It’s a value between 0 and 100:
- 0 → avoid swap as much as possible (use RAM first)
- 100 → swap very aggressively, even if RAM isn’t full

You can check your system’s current swappiness with this command:
```
cat /proc/sys/vm/swappiness
```

And for chaning the swappiness
```
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```