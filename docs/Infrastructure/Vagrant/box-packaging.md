# Box Packaging

This guide covers how to package your configured UTM-based Ubuntu Server 24.04 virtual machine into a `.box` file that is compatible with [Vagrant](https://www.vagrantup.com) using the [`vagrant-utm`](https://github.com/naveenrajm7/vagrant_utm) provider.

---

## 1. Prepare the Directory Structure and Export / Share the vm in UTM into your current Directory

You can create a box.utm directory inside the vm and move Data, screenshot.png and config.plist to box.utm

## 2. Create `metadata.json`

Inside the UTM vm directory (for example, Ubuntu_server_24.04.utm), create a file called `metadata.json` with the following content:

```json
{
  "provider": "utm",
}
```

now this is the structure:

```
.
├── box.utm
│   ├── Data
│   │   ├── 7FB247A3-DC9F-4A61-A123-0AEE1BEEC636.qcow2
│   │   └── efi_vars.fd
│   ├── config.plist
│   └── screenshot.png
└── metadata.json
```

> This file is required by Vagrant to recognize and use the box correctly.

---

## 3. Create the `.box` Archive

From **outside** the `Ubuntu_server_24.04.utm` folder, run:

```bash
tar -czvf ubuntu-server-24.04.box -C Ubuntu_server_24.04.utm .
```

This will create a `ubuntu-server-24.04.box` file that includes both `box.utm` and `metadata.json`.

### ✅ Verify the contents:

```bash
tar -tf ubuntu-server-24.04.box
```

Expected output:

```
box.utm/
metadata.json
```

---

## 4. Add the Box to Vagrant

Run the following command to add your `.box` file to Vagrant:

```bash
vagrant box add ubuntu-server-24.04 ./ubuntu-server-24.04.box --provider=utm
```

Check that it’s added:

```bash
vagrant box list
```

You should see:

```
ubuntu-server-24.04 (utm, 0)
```

---

## 5. Reference in Your Vagrantfile

Here's a basic `Vagrantfile` snippet that uses your new box:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu-server-24.04"
  config.vm.hostname = "Ubuntu-server-test"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.provider "utm" do |my_vm|
    my_vm.name = "my_vm"
    my_vm.memory = 1024
    my_vm.cpus = 1
  end
  config.vm.provision :shell, path: "bootstrap.sh"
end
```

---

## 6. Start the Machine

Spin up the VM:

```bash
vagrant up
```

Connect via SSH:

```bash
vagrant ssh
```

---

## ✅ Done

Your Ubuntu Server 24.04 VM is now fully portable as a Vagrant `.box` and ready for use in any project using the `vagrant-utm` provider.