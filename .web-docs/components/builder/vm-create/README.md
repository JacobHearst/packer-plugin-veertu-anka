Type: `veertu-anka-vm-create`

**Packer 3.x will no longer support Anka 2.x. You can still however use the Packer 2.x release for support.**

The `veertu-anka-vm-create` Packer builder is able to create new Anka VM Templates for use with the
[Anka Virtualization](https://veertu.com/technology/) package and the [Anka Build Cloud](https://veertu.com/anka-build/). The builder takes the path to macOS installer .app 
and installs that macOS version inside of an Anka VM Template.

The builder does _not_ manage templates. Once a template is created, it is up
to you to use it or delete it.

## Configuration Reference

There are many configuration options available for the builder. They are
segmented below into two categories: required and optional parameters.

### Required Configuration

* `installer` (String) The path to a macOS installer. This process takes about 20 minutes.
  - Starting in 3.1.2: This can also be set to 'latest' or a specific macOS version in order to have Anka attempt downloading the installer for you (`vm_name` will be set to `anka-packer-base-${installer}`).

* `type` (String) Must be `veertu-anka-vm-create`.

### Optional Configuration

* `vm_name` (String) The name for the VM that is created. One is generated with installer data if not provided (`anka-packer-base-{{ installer.OSVersion }}-{{ installer.BundlerVersion }}`).

* `vcpu_count` (String) The number of vCPU cores, defaults to `2`.

  > This change gears us up for Anka 3.0 release when cpu_count will be vcpu_count. For now this is still CPU and not vCPU.

* `ram_size` (String) The size in "[0-9]+G" format, defaults to `4G`.

* `disk_size` (String) The size in "[0-9]+G" format, defaults to `40G`.

  > We will automatically resize the internal disk for you by executing `diskutil apfs resizeContainer disk0s2 0` inside of the VM

* `stop_vm` (Boolean) Whether or not to stop the vm after it has been created, defaults to false.

* `use_anka_cp` (Boolean) Use built in anka cp command. You shouldn't need this option. Defaults to false.

* `anka_password` (String) Sets the password for the vm. Can also be set with `ANKA_DEFAULT_PASSWD` env var. Defaults to `admin`.

* `anka_user` (String) Sets the username for the vm. Can also be set with `ANKA_DEFAULT_USER` env var. Defaults to `anka`.

* `boot_delay` (String) The time to wait before running packer provisioner commands, defaults to `7s`.

* `log_level` (String) The log level for Anka. This currently only supports `debug` and is only useful for VM creation failures.

* `hw_uuid` (String) (Anka 2 only) The Hardware UUID you wish to set (usually generated with `uuidgen`).

* `port_forwarding_rules` (Struct) 

  > If port forwarding rules are already set and you want to not have them fail the packer build, use `packer build --force`.
  
  * `port_forwarding_guest_port` (Int)
  * `port_forwarding_host_port` (Int)
  * `port_forwarding_rule_name` (String)

* `display_controller` (string) The display controller to set (run `anka modify VMNAME set display --help` to see available options).

## Example

Here is an example:

```hcl

variable "vm_name" {
  type = string
  default = "anka-packer-base-macos"
}

variable "installer" {
  type = string
  default = "/Applications/Install macOS Big Sur.app/"
}

variable "vcpu_count" {
  type = string
  default = ""
}

source "veertu-anka-vm-create" "base" {
  installer = "${var.installer}"
  vm_name = "${var.vm_name}"
  vcpu_count = "${var.vcpu_count}"
}

build {
  sources = [
    "source.veertu-anka-vm-create.base"
  ]

  provisioner "shell" {
    inline = [
      "echo hello world",
      "echo llamas rock"
    ]
  }
}

```
