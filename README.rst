==================
edpm-image-builder
==================

A repository to build OS images for deploying EDPM bare metal nodes using
diskimage-builder.

Image build elements
--------------------

Images are built with elements from ``diskimage-builder``,
``ironic-python-agent-builder``, and the ``dib`` directory of this repository.
These can be installed in a venv, for example from the ``edpm-image-builder``
directory::

  python3 -m venv ./venv
  source ./venv/bin/activate
  pip install -r requirements.txt
  export ELEMENTS_PATH=$(pwd)/dib:$(pwd)/venv/share/ironic-python-agent-builder/dib

edpm-hardened-uefi
------------------

The CentOS-9-Stream version of ``edpm-hardened-uefi.qcow2`` can be built with
master branch ``current-podified`` by running::

    diskimage-builder ./images/edpm-hardened-uefi-centos-9-stream.yaml

To create a FIPS enabled image, add ``edpm-hardened-uefi-fips.yaml`` to
include the ``fips`` element::

    diskimage-builder ./images/edpm-hardened-uefi-centos-9-stream.yaml ./images/edpm-hardened-uefi-fips.yaml

See dib/repo-setup/README.md for environment variables to control which RDO
repositories to configure.

``edpm-hardened-uefi.qcow2`` can be packaged inside a container image for
distribution by running::

    buildah bud -f ./Containerfile.image -t edpm-hardened-uefi:latest

It can then be copied out of the container image, for example into
``/path/to/images``::

    podman run --volume /path/to/images:/target:Z --rm edpm-hardened-uefi:latest

ironic-python-agent
-------------------

The CentOS-9-Stream version of ``ironic-python-agent.initramfs`` and
``ironic-python-agent.kernel`` can be built with master branch
``current-podified`` by running::

    diskimage-builder ./images/ironic-python-agent-centos-9-stream.yaml

Similarly, the rhel-9 version can be built with master branch
``current-podified`` by running::

    diskimage-builder ./images/ironic-python-agent-rhel-9.yaml

``ironic-python-agent.qcow2`` can be packaged inside a container image for
distribution by running::

    buildah bud -f ./Containerfile.ramdisk -t ironic-python-agent:latest

It can then be copied out of the container image, for example into
``/path/to/images``::

    podman run --volume /path/to/images:/target:Z --rm ironic-python-agent:latest

block-device-yaml
-----------------

The file ``dib/edpm-partition-uefi/block-device-default.yaml`` is generated by
the script ``block-device-yaml`` by running::

    ./block-device-yaml --disk-size 5GiB --output dib/edpm-partition-uefi/block-device-default.yaml

This script can also generate a block device description with custom LVM volume
layouts. For example, to generate a block device layout which replicates the
TripleO overcloud-hardened-uefi-full.qcow2 which has an extra /srv mount point,
run the following::

    ./block-device-yaml --disk-size 6GiB \
        --volumes lv_root=:lv_tmp=240:lv_var=952:lv_log=240:lv_audit=192:lv_home=240:lv_srv=48 \
        --filesystems lv_root=fs_root:lv_tmp=fs_tmp:lv_var=fs_var:lv_log=fs_log:lv_audit=fs_audit:lv_home=fs_home:lv_srv=fs_srv \
        --mounts lv_root=/:lv_tmp=/tmp:lv_var=/var:lv_log=/var/log:lv_audit=/var/log/audit:lv_home=/home:lv_srv=/srv \
        --mount-options lv_tmp=rw,nosuid,nodev,noexec,relatime:lv_home=rw,nodev,relatime:lv_srv=rw,nodev,relatime \
        --output ../tripleo-image-elements/elements/overcloud-partition-uefi/block-device-default.yaml

To see a description of all arguments and their defaults run::

    ./block-device-yaml --help
