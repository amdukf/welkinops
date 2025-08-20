# Bulk File Move


### Option 1: Using `rsync`
```bash
rsync -av --remove-source-files --progress /source/ /dest/
```

### Option 2: Using `find`
```bash
find /path/source -type f | xargs -I{} mv {} /path/dest/
```

### For create test files:
```bash
mkdir -p /tmp/testfiles
seq 1000 | xargs -I{} touch /tmp/testfiles/file_{}.txt
```