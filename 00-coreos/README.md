# Create Simple CoreOS Instance on GCE

This article simply demonstrates the CLI usage for creating a simple [CoreOS](https://coreos.com) instance. No console is used.

## Goal

We want to create a VM using all the standard settings, but run it programmatically. (*What better way to learn the environment?*)

## Prerequisites

1. [GCE](https://cloud.google.com) account
1. [Setup SSH Access Keys](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys) - For our purposes, we recommend you maintain your keys manually. What you gain in complexity, you also gain in knowledge and competency.
1. [Verify Creation of Single Linux VM](https://cloud.google.com/compute/docs/quickstart-linux)
1. [Initialize `gcloud`](https://cloud.google.com/sdk/docs/initializing)
1. [Set your default Region and Zone](https://cloud.google.com/compute/docs/gcloud-compute/#change_your_default_zone_and_region_in_the_metadata_server) - We do this to make our `gcloud` commands below simpler; the project, region, and zone will all use the defaults.

All done? Then you are ready to go. Wave bye-bye to the Web-based console...


## Steps

1. **Determine boot disk image / projecte**. This example uses CoreOS so we format as follows:

    ```
    the_boot_disk_info=$(gcloud compute images list --filter=family=coreos-stable --format="value(name,selfLink)" | head -n 1)
    the_boot_disk_image=$(echo "$the_boot_disk_info" | awk '{print $1}')
    the_boot_disk_project=$(echo "$the_boot_disk_info" | awk '{print $2}')
    ```
    The above queries for only `coreos-stable` boot images and retrieves both the `name` and the `selfLink` (which, by default, returns the image's related project). When creating a GCE VM, it's best practice to specify both the boot image and the project from which the boot image was created.

    It's instructive to run the command without any options to see what you get.
1. **VM Sizing and Defaults**.
    * VM Size: We use `g1-small`, which is very small but still usable for our purposes. (Use `gcloud compute machine-types list` for a full list.)

        ```
        the_machine_type='g1-small'
        ```
    * Network: This will be unspecified, so the `default` network is used. (This network is created for you automatically.)
    * Service Account: Unspecified, so the default Compute Engine service account is used (this is created for you automatically when you used the console to create a VM.
    * Firewall: We use `http-server` which is a default network access rule created by Google Compute Engine.

        ```
        the_tags='http-server'
        ```
    * Boot Disk: We specify 10GB for our boot disk, and the least expensive type of `pd-standard` to create a standard disk.

        ```
        the_boot_disk_type='pd-standard'
        the_boot_disk_size='10GB'
        ```
1. **VM Name and Description.** These will always be passed in.

    ```
    read the_vm_name
    read the_vm_description
    ```
1. **The Raw Command**. Given the above, here's what we use:

    ```
    gcloud compute instances create \
      --boot-disk-size=$the_boot_disk_size --boot-disk-type=$the_boot_disk_type \
      --description="$the_vm_description" \
      --image=$the_boot_disk_image --image-project=$the_boot_disk_project \
      --machine-type $the_machine_type --tags="$the_tags" \
      $the_vm_name
    ```
1. **View the Created Instance.** To check on your results, simply use:

    ```
    gcloud compute instances list
    ```
1. **Access the Instance.** We will use SSH to do this, here is a sample command to list network addresses on the running VM:

    ```
    read the_ssh_key_file
    gcloud compute ssh \
      --strict-host-key-checking=no --ssh-key-file="$the_ssh_key_file" $the_vm_name \
      -- ip a
    ```
1. **Destroy the Instance.**

    ```
    gcloud compute instances delete $the_vm_name
    ```

