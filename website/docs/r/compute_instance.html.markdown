---
layout: "google"
page_title: "Google: google_compute_instance"
sidebar_current: "docs-google-compute-instance-x"
description: |-
  Manages a VM instance resource within GCE.
---

# google\_compute\_instance

Manages a VM instance resource within GCE. For more information see
[the official documentation](https://cloud.google.com/compute/docs/instances)
and
[API](https://cloud.google.com/compute/docs/reference/latest/instances).


## Example Usage

```hcl
resource "google_compute_instance" "default" {
  name         = "test"
  machine_type = "n1-standard-1"
  zone         = "us-central1-a"

  tags = ["foo", "bar"]

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-8"
    }
  }

  // Local SSD disk
  scratch_disk {
  }

  network_interface {
    network = "default"

    access_config {
      // Ephemeral IP
    }
  }

  metadata {
    foo = "bar"
  }

  metadata_startup_script = "echo hi > /test.txt"

  service_account {
    scopes = ["userinfo-email", "compute-ro", "storage-ro"]
  }
}
```

## Argument Reference

The following arguments are supported:

* `boot_disk` - (Required) The boot disk for the instance.
    Structure is documented below.

* `machine_type` - (Required) The machine type to create.

    **Note:** If you want to update this value (resize the VM) after initial creation, you must set [`allow_stopping_for_update`](#allow_stopping_for_update) to `true`.

    To create a machine with a [custom type][custom-vm-types] (such as extended memory), format the value like `custom-VCPUS-MEM_IN_MB` like `custom-6-20480` for 6 vCPU and 20GB of RAM.

* `name` - (Required) A unique name for the resource, required by GCE.
    Changing this forces a new resource to be created.

* `zone` - (Required) The zone that the machine should be created in.

* `network_interface` - (Required) Networks to attach to the instance. This can
    be specified multiple times. Structure is documented below.

- - -

* `allow_stopping_for_update` - (Optional) If true, allows Terraform to stop the instance to update its properties.
  If you try to update a property that requires stopping the instance without setting this field, the update will fail.

* `attached_disk` - (Optional) List of disks to attach to the instance. Structure is documented below.

* `can_ip_forward` - (Optional) Whether to allow sending and receiving of
    packets with non-matching source or destination IPs.
    This defaults to false.

* `create_timeout` - (Optional) Configurable timeout in minutes for creating instances. Default is 4 minutes.
    Changing this forces a new resource to be created.

* `description` - (Optional) A brief description of this resource.

* `deletion_protection` - (Optional) Enable deletion protection on this instance. Defaults to false.
    **Note:** you must disable deletion protection before removing the resource (e.g., via `terraform destroy`), or the instance cannot be deleted and the Terraform run will not complete successfully.

* `guest_accelerator` - (Optional) List of the type and count of accelerator cards attached to the instance. Structure documented below.
    **Note:** GPU accelerators can only be used with [`on_host_maintenance`](#on_host_maintenance) option set to TERMINATE.

* `labels` - (Optional) A set of key/value label pairs to assign to the instance.

* `metadata` - (Optional) Metadata key/value pairs to make available from
    within the instance.

* `metadata_startup_script` - (Optional) An alternative to using the
    startup-script metadata key, except this one forces the instance to be
    recreated (thus re-running the script) if it is changed. This replaces the
    startup-script metadata key on the created instance and thus the two
    mechanisms are not allowed to be used simultaneously.

* `min_cpu_platform` - (Optional) Specifies a minimum CPU platform for the VM instance. Applicable values are the friendly names of CPU platforms, such as
`Intel Haswell` or `Intel Skylake`. See the complete list [here](https://cloud.google.com/compute/docs/instances/specify-min-cpu-platform).
    **Note**: [`allow_stopping_for_update`](#allow_stopping_for_update) must be set to true in order to update this field.

* `project` - (Optional) The ID of the project in which the resource belongs. If it
    is not provided, the provider project is used.

* `scheduling` - (Optional) The scheduling strategy to use. More details about
    this configuration option are detailed below.

* `scratch_disk` - (Optional) Scratch disks to attach to the instance. This can be
    specified multiple times for multiple scratch disks. Structure is documented below.

* `service_account` - (Optional) Service account to attach to the instance.
    Structure is documented below.
    **Note**: [`allow_stopping_for_update`](#allow_stopping_for_update) must be set to true in order to update this field.

* `tags` - (Optional) A list of tags to attach to the instance.

---

The `boot_disk` block supports:

* `auto_delete` - (Optional) Whether the disk will be auto-deleted when the instance
    is deleted. Defaults to true.

* `device_name` - (Optional) Name with which attached disk will be accessible.
    On the instance, this device will be `/dev/disk/by-id/google-{{device_name}}`.

* `disk_encryption_key_raw` - (Optional) A 256-bit [customer-supplied encryption key]
    (https://cloud.google.com/compute/docs/disks/customer-supplied-encryption),
    encoded in [RFC 4648 base64](https://tools.ietf.org/html/rfc4648#section-4)
    to encrypt this disk.

* `initialize_params` - (Optional) Parameters for a new disk that will be created
    alongside the new instance. Either `initialize_params` or `source` must be set.
    Structure is documented below.

* `source` - (Optional) The name or self_link of the existing disk (such as those managed by
    `google_compute_disk`) to attach.

The `initialize_params` block supports:

* `size` - (Optional) The size of the image in gigabytes. If not specified, it
    will inherit the size of its base image.

* `type` - (Optional) The GCE disk type. May be set to pd-standard or pd-ssd.

* `image` - (Optional) The image from which to initialize this disk. This can be
    one of: the image's `self_link`, `projects/{project}/global/images/{image}`,
    `projects/{project}/global/images/family/{family}`, `global/images/{image}`,
    `global/images/family/{family}`, `family/{family}`, `{project}/{family}`,
    `{project}/{image}`, `{family}`, or `{image}`. If referred by family, the
    images names must include the family name. If they don't, use the
    [google_compute_image data source](/docs/providers/google/d/datasource_compute_image.html).
    For instance, the image `centos-6-v20180104` includes its family name `centos-6`.
    These images can be referred by family name here.

The `scratch_disk` block supports:

* `interface` - (Optional) The disk interface to use for attaching this disk; either SCSI or NVME.
    Defaults to SCSI.

The `attached_disk` block supports:

* `source` - (Required) The name or self_link of the disk to attach to this instance.

* `device_name` - (Optional) Name with which the attached disk will be accessible
    under `/dev/disk/by-id/`

* `mode` - (Optional) Either "READ_ONLY" or "READ_WRITE", defaults to "READ_WRITE"
    If you have a persistent disk with data that you want to share
    between multiple instances, detach it from any read-write instances and
    attach it to one or more instances in read-only mode.

* `disk_encryption_key_raw` - (Optional) A 256-bit [customer-supplied encryption key]
    (https://cloud.google.com/compute/docs/disks/customer-supplied-encryption),
    encoded in [RFC 4648 base64](https://tools.ietf.org/html/rfc4648#section-4)
    to encrypt this disk.

The `network_interface` block supports:

* `network` - (Optional) The name or self_link of the network to attach this interface to.
    Either `network` or `subnetwork` must be provided.

*  `subnetwork` - (Optional) The name or self_link of the subnetwork to attach this
    interface to. The subnetwork must exist in the same region this instance will be
    created in. Either `network` or `subnetwork` must be provided.

*  `subnetwork_project` - (Optional) The project in which the subnetwork belongs.
   If the `subnetwork` is a self_link, this field is ignored in favor of the project
   defined in the subnetwork self_link. If the `subnetwork` is a name and this
   field is not provided, the provider project is used.

* `address` - (Optional) The private IP address to assign to the instance. If
    empty, the address will be automatically assigned.

* `access_config` - (Optional) Access configurations, i.e. IPs via which this
    instance can be accessed via the Internet. Omit to ensure that the instance
    is not accessible from the Internet (this means that ssh provisioners will
    not work unless you are running Terraform can send traffic to the instance's
    network (e.g. via tunnel or because it is running on another cloud instance
    on that network). This block can be repeated multiple times. Structure
    documented below.

* `alias_ip_range` - (Optional) An
    array of alias IP ranges for this network interface. Can only be specified for network
    interfaces on subnet-mode networks. Structure documented below.

The `access_config` block supports:

* `nat_ip` - (Optional) The IP address that will be 1:1 mapped to the instance's
    network ip. If not given, one will be generated.

* `public_ptr_domain_name` - (Optional) The DNS domain name for the public PTR record.
    To set this field on an instance, you must be verified as the owner of the domain.
    See [the docs](https://cloud.google.com/compute/docs/instances/create-ptr-record) for how
    to become verified as a domain owner.

* `network_tier` - (Optional) The [networking tier][network-tier] used for configuring this instance.
    This field can take the following values: PREMIUM or STANDARD. If this field is
    not specified, it is assumed to be PREMIUM.

The `alias_ip_range` block supports:

* `ip_cidr_range` - The IP CIDR range represented by this alias IP range. This IP CIDR range
    must belong to the specified subnetwork and cannot contain IP addresses reserved by
    system or used by other network interfaces. This range may be a single IP address
    (e.g. 10.2.3.4), a netmask (e.g. /24) or a CIDR format string (e.g. 10.1.2.0/24).

* `subnetwork_range_name` - (Optional) The subnetwork secondary range name specifying
    the secondary range from which to allocate the IP CIDR range for this alias IP
    range. If left unspecified, the primary range of the subnetwork will be used.

The `service_account` block supports:

* `email` - (Optional) The service account e-mail address. If not given, the
    default Google Compute Engine service account is used.
    **Note**: [`allow_stopping_for_update`](#allow_stopping_for_update) must be set to true in order to update this field.

* `scopes` - (Required) A list of service scopes. Both OAuth2 URLs and gcloud
    short names are supported. To allow full access to all Cloud APIs, use the
    `cloud-platform` scope. See a complete list of scopes [here](https://cloud.google.com/sdk/gcloud/reference/alpha/compute/instances/set-scopes#--scopes).
    **Note**: [`allow_stopping_for_update`](#allow_stopping_for_update) must be set to true in order to update this field.

The `scheduling` block supports:

* `preemptible` - (Optional) Is the instance preemptible.

* `on_host_maintenance` - (Optional) Describes maintenance behavior for the
    instance. Can be MIGRATE or TERMINATE, for more info, read
    [here](https://cloud.google.com/compute/docs/instances/setting-instance-scheduling-options)

* `automatic_restart` - (Optional) Specifies if the instance should be
    restarted if it was terminated by Compute Engine (not a user).

The `guest_accelerator` block supports:

* `type` (Required) - The accelerator type resource to expose to this instance. E.g. `nvidia-tesla-k80`.

* `count` (Required) - The number of the guest accelerator cards exposed to this instance.

## Attributes Reference

In addition to the arguments listed above, the following computed attributes are
exported:

* `instance_id` - The server-assigned unique identifier of this instance.

* `metadata_fingerprint` - The unique fingerprint of the metadata.

* `self_link` - The URI of the created resource.

* `tags_fingerprint` - The unique fingerprint of the tags.

* `label_fingerprint` - The unique fingerprint of the labels.

* `cpu_platform` - The CPU platform used by this instance.

* `network_interface.0.address` - The internal ip address of the instance, either manually or dynamically assigned.

* `network_interface.0.access_config.0.assigned_nat_ip` - If the instance has an access config, either the given external ip (in the `nat_ip` field) or the ephemeral (generated) ip (if you didn't provide one).

* `attached_disk.0.disk_encryption_key_sha256` - The [RFC 4648 base64](https://tools.ietf.org/html/rfc4648#section-4)
    encoded SHA-256 hash of the [customer-supplied encryption key]
    (https://cloud.google.com/compute/docs/disks/customer-supplied-encryption) that protects this resource.

* `boot_disk.disk_encryption_key_sha256` - The [RFC 4648 base64](https://tools.ietf.org/html/rfc4648#section-4)
    encoded SHA-256 hash of the [customer-supplied encryption key]
    (https://cloud.google.com/compute/docs/disks/customer-supplied-encryption) that protects this resource.

* `disk.0.disk_encryption_key_sha256` - The [RFC 4648 base64](https://tools.ietf.org/html/rfc4648#section-4)
    encoded SHA-256 hash of the [customer-supplied encryption key]
    (https://cloud.google.com/compute/docs/disks/customer-supplied-encryption) that protects this resource.

## Import

~> **Note:** The fields `boot_disk.0.disk_entryption_raw` and `attached_disk.*.disk_encryption_key_raw` cannot be imported automatically. The API doesn't return this information. If you are setting one of these fields in your config, you will need to update your state manually after importing the resource.

Instances can be imported using the `project`, `zone` and `name`, e.g.

```
$ terraform import google_compute_instance.default gcp-project/us-central1-a/test
```

[custom-vm-types]: https://cloud.google.com/dataproc/docs/concepts/compute/custom-machine-types
[network-tier]: https://cloud.google.com/network-tiers/docs/overview
