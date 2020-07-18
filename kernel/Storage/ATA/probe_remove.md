+ probe
+ remove

# probe


# remove

``` C
ata_pci_remove_one - +
                     | - ata_host_detach - +
                                           - ata_port_detach - +   //cycle for each port and set ATA_PFLAG_UNLOADING
                                                               | - ata_port_schedule_eh - +
                                                                                          | - ata_std_sched_eh - +
                                                                                                                 | - scsi_error_handler - +
                                                                                                                                          | - ata_scsi_error - +
                                                                                                                                                               | - too long, break
                                                               | - ata_tlink_delete - + //cycle for each pmp_link to delete device
                                                               | - scsi_remove_host
                                                               | - ata_tport_delete
```

``` C
ata_scsi_error - +
                 | - ata_scsi_cmd_error_handler
                 | - ata_scsi_port_error_handler - +
                                                   | - ata_eh_unload  //ata device disabled
```
