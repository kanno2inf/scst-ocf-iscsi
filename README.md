# scst-ocf-iscsi
iSCSI Resource Agent for Pacemaker

This is a simple iSCSI Resouce agent for pacemaker.

This is what I'm using in my (Very Old) Production SAN:

- SCST Version 3.1.0-pre1
- Pacemaker 1.1.12

A simple CIB Looks like:

```node isan01 \
    attributes standby="off"
node isan02 \
    attributes standby="off"
primitive DRBD_VG1 ocf:linbit:drbd \
    params drbd_resource="ISCSIVG1" \
    op monitor interval="29" role="Master" \
    op monitor interval="31" role="Slave"
primitive ISCSI_IP1 ocf:heartbeat:IPaddr2 \
    params ip="192.168.100.20" \
    op monitor interval="10s"
primitive ISCSI_LUN_LUN10 ocf:scst:SCSTLun \
    params target_iqn="iqn.2012-02.com.

isan:vdisk.lun10" lun="0" path="/dev/drbd/by-res/DRBD_VG1" handler="vdisk_fileio" device_name="VDISK-LUN10" additional_parameters="nv_cache=1" \
    op monitor interval="10s"
primitive ISCSI_TGT_LUN10 ocf:scst:SCSTTarget \
    params iqn="iqn.2012-02.com.isan:vdisk.lun10" portals="192.168.100.20" \
    op monitor interval="10s" timeout="60s"
group GR_ISCSIVG1 ISCSI_TGT_LUN10 ISCSI_LUN_LUN10 ISCSI_IP1 
ms MS_DRBD_VG1 DRBD_VG1 \
    meta master-max="1" master-node-max="1" clone-max="2" clone-node-max="1" notify="true"
colocation CO_ISCSI_ON_DRBD_VG1 inf: GR_ISCSIVG1 MS_DRBD_VG1:Master 
order OR_TARGET_BEFORE_VG1 inf: CL_ISCSI_TGT_LUN1:start GR_ISCSIVG1:start 
order OR_DRBD_BEFORE_VG1 inf: MS_DRBD_VG1:promote GR_ISCSIVG1:start
property $id="cib-bootstrap-options" \
    dc-version="1.0.9-da7075976b5ff0bee71074385f8fd02f296ec8a3" \
    cluster-infrastructure="openais" \
    expected-quorum-votes="2" \
    stonith-enabled="false" \
    no-quorum-policy="ignore" \
    default-action-timeout="240"
rsc_defaults $id="rsc-options" \
    resource-stickiness="200"
```
