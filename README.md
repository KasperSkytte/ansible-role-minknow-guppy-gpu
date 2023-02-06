minknow_guppy_gpu
=========

This Ansible role sets up MinKNOW from Oxford Nanopore Technologies with live GPU basecalling support from Guppy. Alternatively only installs Guppy with GPU support without installing MinKNOW. Currently only supports Debian/Ubuntu because that's what I needed, but feel free to contribute. Installation is performed in much the same way as ONT recommends according to [this guide](https://community.nanoporetech.com/docs/prepare/library_prep_protocols/Guppy-protocol/v/gpb_2003_v1_revae_14dec2018/installing-gpu-version-of-guppy-with-minknow-for-minion) (requires login, corporate BS).

Requirements
------------

None other than ansible.

Installation
----------------
Install from Ansible Galaxy with
```
ansible-galaxy install kasperskytte.minknow_guppy_gpu
```
Role Variables
--------------

Defaults can be found in `vars/main.yml`

- `ont_install_minKNOW`: Whether to also install MinKNOW or not
- `ont_guppy_gpu_dir`: Location where the GPU version of Guppy will be installed

If `ont_install_minKNOW` is `true` the following options are passed on directly to the `guppy_basecall_server` command in the system service unit:
- `guppy_gpu_runners_per_device`
- `guppy_chunks_per_runner`
- `guppy_cfg`: Which Guppy basecall configuration file to use
- `guppy_cuda_devices`: Which CUDA GPU's to use, fx `auto` or `cuda:0,1`

Other internal variables that don't require any adjustments by the end user is available in `vars/main.yml`.

### **Setting optimal Guppy parameters**
The optimal settings of the last two options depends on your hardware. ONT has written the following about it ([source](https://community.nanoporetech.com/docs/prepare/library_prep_protocols/Guppy-protocol/v/gpb_2003_v1_revae_14dec2018/linux-guppy)):


The following calculation provides a rough ceiling to the amount of GPU memory that Guppy will use:

**memory used [in bytes] = gpu_runners_per_device * chunks_per_runner * chunk_size * model_factor**

Where model_factor depends on the basecall model used:

- `Fast`: 1200
- `HAC`: 3300
- `SUP`: 12600

Note that `gpu_runners_per_device` is a limit setting. Guppy will create at least one runner per device and will dynamically increase this number as needed up to `gpu_runners_per_device`. Performance is usually much better with multiple runners, so it is important to choose parameters such that `chunks_per_runner` * `chunk_size` * `model_factor` is less than half the total GPU memory available. If this value is more than the available GPU memory, Guppy will exit with an error.

For example, when basecalling using a SUP model and a chunk size of 2000 on a GPU with 8 GB of memory, we have to make sure that the GPU can fit at least one runner:


**chunks_per_runner * chunk_size * model_factor** should be lower than GPU memory

**chunks_per_runner * 2000 * 12600** should be lower than 8 GB

**chunks_per_runner** lower than ~340

This represents the limit beyond which Guppy will not run at all. For best performance we recommend using an integer fraction of this number, rounded down to an even number, e.g. a third (~112) or a quarter (~84). Especially for fast models, it can be best to have a dozen runners or more. The ideal value varies depending on GPU architecture and available memory.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```
- hosts: servers
  become: true
  roles:
    - kasperskytte.minknow_guppy_gpu
  vars:
    ont_install_minKNOW: true
    guppy_cfg: dna_r9.4.1_450bps_hac.cfg
    guppy_gpu_runners_per_device: 8
    guppy_chunks_per_runner: 80
    guppy_cuda_devices: auto
```
License
-------

GPL, 2.0 or later
